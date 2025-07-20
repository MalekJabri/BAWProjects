# GitHub – JFrog – IBM BAW Integration Lab

This repository demonstrates how to integrate GitHub, JFrog Artifactory, and IBM Business Automation Workflow (BAW) into a unified CI/CD pipeline.

## 🔍 Purpose

The goal of this lab is to showcase how modern DevOps tools can be combined to:

- Store and version BAW-related artifacts (e.g., .twx files, service libraries, scripts)
- Automate build and deployment pipelines using GitHub Actions
- Leverage JFrog Artifactory for artifact management and promotion
- Deploy process applications or services to IBM BAW environments via REST APIs or CLI

## 📚 Official Documentation

This lab is based on the official IBM guidance for continuous integration pipelines:

- [Integrating GitHub with IBM BAW](https://www.ibm.com/docs/en/baw/25.0.0?topic=cpci-integrating-github)
- [Integrating JFrog Artifactory with IBM BAW](https://www.ibm.com/docs/en/baw/25.0.0?topic=cpci-integrating-jfrog-artifactory)

These pages explain how IBM BAW interacts with source control and artifact repositories to support automated deployment flows.

## 🧱 Architecture Overview

Developer Push →
GitHub Action →
Build + Create BAW artifact (.twx or .jar) →
Publish to JFrog Artifactory →
Trigger deployment to IBM BAW (Dev/Test/Prod)


## 🧪 What You’ll Learn

- How to configure GitHub Actions with JFrog CLI
- How to publish and retrieve artifacts from JFrog
- How to deploy to IBM BAW using automated scripts or APIs
- Best practices for CI/CD in automation workflows

## 🧰 Tech Stack

- GitHub Actions
- JFrog Artifactory
- IBM Business Automation Workflow (BAW)
- Bash / Shell scripts
- (Optional) Docker or Kubernetes if BAW is containerized


## ⚠️ Prerequisites

- Access to a JFrog Artifactory instance
- IBM BAW installed or provisioned (local, cloud, or containerized)
- GitHub repository with access to GitHub Actions
- JFrog CLI installed in CI agents
- API token or credentials with permission to upload artifacts

## 🧭 Getting Started

1. Fork or clone this repo
2. Configure secrets (`JFROG_URL`, `JFROG_USER`, `JFROG_PASSWORD`)
3. Push a change or manually trigger a GitHub Action
4. Monitor artifact upload and deployment steps

## 📌 License

This repository is provided for educational and demonstration purposes only.

---

Feel free to adapt or improve this README as your lab evolves!
