# Platypus Programming Platform: Generative AI on AWS (Case Study)

**RMIT University Master of IT capstone · Jul–Oct 2026 · Richard Paul John Rathnaraj**

> This repository is a case study only. The source code belongs to RMIT University and is not included. I'm happy to walk through the architecture and design decisions on a call.

## Overview

Platypus is a web platform where instructors build programming courses and students complete interactive coding exercises. Our five-person postgraduate team extended an existing codebase with AI learning support, progress monitoring and in-browser Python execution, then deployed the platform to AWS.

**What I owned:**

- The AI learning features on **Amazon Bedrock** (Socratic coding hints and a course-aware chatbot)
- The **AWS deployment** and cloud security
- **In-browser Python and Turtle graphics** execution

## Architecture

![AWS deployment architecture](images/aws-architecture.png)

Users reach the platform through an AWS load balancer. A single EC2 instance runs the stack with Docker Compose: Nginx serving the React app and proxying `/api` to an Express API, a BullMQ worker, PostgreSQL with Flyway migrations, Redis, and Judge0 for remote code execution. Images are pulled from private Amazon ECR repositories, and the API calls Amazon Bedrock using temporary credentials from an EC2 IAM role.

## Key decisions

**Choosing the model.** The client asked me to choose the AI service. We compared Amazon Bedrock, an external OpenAI-compatible provider and a mock provider. I chose Bedrock with **Amazon Nova Micro**: our use cases (hints, programming guidance and course chat) were text-only, so a larger multimodal model was unnecessary. Nova Micro suited low-latency, low-cost text interactions and kept model access and IAM permissions inside AWS.

**A provider-agnostic AI layer.** Features call a shared interface with common request, response and error types, so the model or provider can change without rewriting features. A mock provider keeps development and automated tests cheap.

**No static AWS keys.** The EC2 instance role grants only Bedrock model invocation and ECR image pull, within the organisation's IAM permissions boundary. Runtime secrets are kept out of container images.

**Running Python in the browser.** Supported exercises run client-side with Pyodide inside a Web Worker. This reduces load on the Judge0 service, keeps the page responsive, and lets students stop runaway loops without freezing the app.

## Delivered and verified

- 4 versioned private ECR images (frontend, API, worker, PostgreSQL)
- 10-service Docker Compose stack on EC2, behind Nginx and an AWS load balancer
- Validation returned HTTP 200 end-to-end (instance, load balancer, API through Nginx); direct Bedrock invocation succeeded
- PostgreSQL moved from trust authentication to SCRAM-SHA-256
- Contributed to the team's AI and chatbot test suite (25 test files, 135 test cases)
- End-to-end demonstration of instructor and student workflows

## Problems I solved

- **ECR publishing failures:** immutable image tags clashed with OCI provenance manifests generated during the build.
- **IAM permissions boundary denials:** the instance role had to fit inside the organisation's required boundary and naming convention.
- **A frontend container restart loop:** caused by a configuration-replacement script that didn't safely handle files it had already processed.

## What I would do differently

- **Tighter hint guardrails.** Later hints in a sequence could become too direct. I would add an output check that blocks hints containing the solution before they reach the student.
- **HTTPS and infrastructure as code.** Add TLS on the load balancer with AWS Certificate Manager, and define the environment in Terraform instead of configuring it by hand.
- **Move off a single EC2 instance.** Run the services on ECS Fargate for resilience and simpler scaling.
- **Measure cost and latency.** We chose Nova Micro for low cost and latency but did not benchmark either. I would capture CloudWatch metrics from day one.

## Tech stack

- **AWS:** EC2, ECR, Application Load Balancer, IAM roles and instance profiles, Amazon Bedrock (Nova Micro)
- **Application:** React, TypeScript, Express, PostgreSQL, Redis, BullMQ, Flyway, Judge0
- **Tooling:** Docker, Docker Compose, Nginx, Pyodide, Web Workers, HTML Canvas, OpenAI Codex

## Contact

[LinkedIn](https://www.linkedin.com/in/richard-paul-john-rathnaraj-936215222) · [GitHub](https://github.com/Rich-py)
