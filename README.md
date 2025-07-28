# GitHub Actions – IBM BAW Integration Lab

This repository demonstrates how to use GitHub Actions to automate CI/CD workflows for IBM Business Automation Workflow (BAW) projects.

## 🔍 Purpose

The goal of this lab is to show how GitHub Actions can be used to:

- Build and package BAW artifacts (e.g., .twx files)
- Automate deployment steps to IBM BAW environments
- Enable continuous integration for BAW development

## 📚 Official Documentation

This lab is based on the official IBM guidance for continuous integration pipelines:

- [Integrating GitHub with IBM BAW](https://www.ibm.com/docs/en/baw/25.0.0?topic=cpci-integrating-github)
- [Integrating JFrog Artifactory with IBM BAW](https://www.ibm.com/docs/en/baw/25.0.0?topic=cpci-integrating-jfrog-artifactory)

These pages explain how IBM BAW interacts with source control and artifact repositories to support automated deployment flows.

## 🧪 Example Workflow

The main workflow for this lab is defined in `.github/workflows/test-workflow.yml`:

```yaml
name: CI/CD for BAW Project 

on:
  push:
    branches: [ main ]
    paths:
      - 'workflow/CO/**.json'
  workflow_dispatch:

jobs:
  install_package:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        id: checkout_code
        uses: actions/checkout@v4

      - name: Welcome message
        id: welcome_message
        run: |
          echo "::group::My Group"
          echo "🎉 Showcase of CI/CD pipeline in the BAW project "
          echo "=== BAW Package Installation ==="
          echo "Date: $(date)"
          echo "::endgroup::"
      - name: Extract values from json file
        id: extract_json
        run: ...
      - name: Get CSRF token
        id: csrf_token
        run: ...
      - name: Download the package
        id: download_package  
        run: ....     
      - name: Install the package
        id: install_package
        run: ...

```

## 🧰 Tech Stack

- GitHub Actions
- IBM Business Automation Workflow (BAW)
- Bash / Shell scripts
- Act toolkit

## ⚠️ Prerequisites

- IBM BAW installed or provisioned (local, cloud, or containerized)
- GitHub repository with access to GitHub Actions

## 🧭 Getting Started

1. Fork or clone this repo
2. Review and adapt `.github/workflows/test-workflow.yml` as needed
3. Push a change or manually trigger the workflow
4. Monitor build and deployment steps in the Actions tab

## 📌 License

This repository is provided for educational and demonstration purposes only.

---

Feel free to adapt or improve this README as your lab evolves!