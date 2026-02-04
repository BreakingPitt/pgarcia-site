---
layout: post
title: "Linting Docker Images with Dockle: Security Checks for Your Container Artifacts"
date: 2026-02-04
---
Dockerfiles are easy to review, but once they turn into images it is much harder to see what really ended up inside. Dockle bridges that gap by scanning built images for insecure defaults, missing best practices and subtle configuration issues before they reach production.

> **What is Dockle used for?**
Dockle is a container image linter that scans built Docker images for security misconfigurations and bad practices, such as running as root, missing health checks or using the latest tag. It helps you catch these issues before pushing images to production.

## Table of contents

- [What is a linter?](#what-is-a-linter)
- [Types of linters](#types-of-linters)
- [What is Dockle?](#what-is-dockle)
  - [Key features of Dockle](#key-features-of-dockle)
  - [Dockle check levels](#dockle-check-levels)
- [Installing Dockle](#installing-dockle)
  - [macOS (via Homebrew)](#macos-via-homebrew)
  - [macOS binary](#macos-binary)
  - [Docker](#docker)
- [How to use Dockle to scan a Docker image](#how-to-use-dockle-to-scan-a-docker-image)
- [How to export Dockle scan results](#how-to-export-dockle-scan-results)
- [Configuring Dockle](#configuring-dockle)
- [CI/CD integration: Using Dockle with GitHub Actions](#cicd-integration-using-dockle-with-github-actions)
- [Frequently asked questions (FAQs)](#frequently-asked-questions-faqs)
- [Conclusion](#conclusion)
- [Resources](#resources)

## What is a linter?

A linter is a static analysis tool that reviews source code to detect issues such as syntax errors, potential bugs, security flaws or violations of coding conventions, without running the code.Linters are essential tools for maintaining clean, secure and consistent codebases in modern software development.

By integrating linters into development workflows and CI/CD pipelines, teams can reduce technical debt, improve code readability and prevent errors from reaching production environments. Although linters are often associated with programming languages, the same idea also applies to other artefacts such as configuration files, Dockerfiles or container images.

### Benefits of using a linter

- **Static analysis without execution** – Linters examine code without running it, helping developers catch issues early in the development lifecycle.
- **Comprehensive issue detection** – Detect unused variables, formatting issues, performance bottlenecks and potential security risks.
- **Improved code quality** – Enforce consistent coding standards and style guides, resulting in more maintainable and predictable code.
- **Customizable rule sets** – Most linters support configurable rules to meet specific language, project or compliance requirements.
- **Seamless automation** – Easily integrate with IDEs and CI/CD tools like GitHub Actions or GitLab CI to enable automated checks on each commit or pull request.

## Types of linters

Linters come in several types, each one designed to address different development needs, whether by programming language, project complexity or specific concerns like security or code style. Choosing the right type of linter can significantly improve your development workflow and overall code quality.

Here are the most common categories of linters:

- **Language-specific linters** – These linters are designed for a single programming language and provide in‑depth, syntax‑aware analysis. Tools in this category, such as `ESLint` for JavaScript, `Pylint` for Python and `RuboCop` for Ruby, offer precise rule sets tailored to each language’s syntax and best practices.
- **General-purpose linters** – These linters support multiple programming languages and provide broad code quality checks across diverse codebases. They are ideal for teams working in polyglot environments and aiming for unified quality standards. Tools like `SonarQube` and `Codacy` provide cross‑language support and integrate with CI/CD pipelines to enforce coding standards, detect bugs and highlight security risks in real time.
- **Specialized linters** – Focused on specific concerns such as **security**, **code formatting** or **style enforcement**. Tools like `Bandit` scan Python code for security issues, while `Prettier` enforces consistent formatting.

Understanding the different linter types allows development teams to select the best tool for their specific goals, whether it is reducing bugs, improving security or maintaining clean and consistent code across large projects.

## What is Dockle?

Dockle is a container image security linter that helps ensure your Docker images follow best practices and are free from high‑risk configurations. It analyzes container images after they have been built, reviewing the image’s command history and metadata to detect potential misconfigurations, security flaws or insecure setups.

Unlike Dockerfile linters that review your build instructions, Dockle inspects the final image, allowing it to catch issues that might not be evident during the build process. This makes it a powerful tool for post‑build validation and production‑grade container security checks.

### Key features of Dockle

Below are some of Dockle’s key features:

- **Security risk detection in container images** – Dockle inspects built container images for common security risks and configuration flaws, such as running as the root user, missing health checks or leaving sensitive files in the image.
- **Enforces Dockerfile best practices** – While Dockle scans the final image, it indirectly enforces Dockerfile best practices by flagging issues that typically originate from poor image design, such as unused packages, lack of version pinning or improper user permissions.
- **Simple CLI usage** – Dockle is easy to use: you simply specify the image name you want to scan; there is no need to manually point to Dockerfiles or set up complex rules for basic usage.
- **Supports CIS Docker Benchmark checks** – Dockle includes checks aligned with the [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker), helping teams comply with widely accepted security standards for container hardening.
- **CI/CD integration ready** – Dockle works well in CI/CD pipelines and can be used in platforms such as GitHub Actions, GitLab CI and CircleCI.
- **Customizable output and exit codes** – Dockle supports multiple output formats and configurable exit codes or exit levels, making it easy to enforce different policies for different environments.
- **Lightweight and fast** – With no heavy dependencies and no need to scan external registries, Dockle executes quickly, which makes it suitable for frequent use in development and CI workflows.

By inspecting image history and metadata, Dockle helps ensure that your Docker images comply with security guidelines and best practices before you push them to production, and it nicely complements Dockerfile linters like [Hadolint](https://github.com/hadolint/hadolint) by validating the final image.

### Dockle check levels

Dockle uses five distinct check levels to classify the issues it detects when scanning a container image. These levels help developers prioritise which findings need immediate attention and which ones are informational.

- **FATAL** – Critical issues that must be addressed immediately and should block a release.
- **WARN** – Warnings about potential problems that are usually not acceptable in production.
- **INFO** – Informational messages about the image that may affect maintainability, observability or performance.
- **SKIP** – Checks that were skipped because the relevant files or conditions were not found.
- **PASS** – Checks that passed successfully.

Understanding these levels makes it easier to decide when a Dockle finding should fail a build and when it can be monitored or documented instead.


## Installing Dockle

  You can install Dockle in multiple ways depending on your environment, workflow or operating system preferences. This makes it easy to integrate it both on developer machines and in automated pipelines.

### macOS (via Homebrew)

  If you do not have Dockle installed yet, you can install it on macOS with Homebrew (https://brew.sh/):

```bash
  brew install goodwithtech/r/dockle
```

### macOS binary

  Dockle can be downloaded as a standalone binary for your platform from the GitHub releases page. This method is lightweight, fast and ideal for scripting or integrating into CI environments where Docker or a package manager might not be available.

  To install Dockle using the standalone binary method, follow these steps:

  - Access the releases page – Go to the official Dockle GitHub releases page (https://github.com/goodwithtech/dockle/releases), where you will find the latest versions for multiple platforms.
  - Download the correct binary – Choose the binary that matches your OS and architecture, such as dockle_0.4.15_macOS-64bit.tar.gz. You can also download it directly from the command line using curl:


```bash
  curl -L -o dockle.tar.gz "https://github.com/goodwithtech/dockle/releases/download/v0.4.15/dockle_0.4.15_macOS-64bit.tar.gz"
```

  - Extract the contents of the tar.gz file – Extract the contents of the archive to make the Dockle binary available in your current directory:


```bash
   tar -xvzf dockle.tar.gz
```

  - Make it executable and move it to your PATH – After downloading, assign execution permissions and move it to a directory included in your system’s PATH. This allows you to run Dockle from anywhere in your terminal:


```bash
    chmod +x dockle-* && sudo mv dockle-* /usr/local/bin/dockle
```

Once the installation is complete, verify that Dockle was installed successfully by entering the following command in the terminal:

```bash
  dockle --version
  # dockle version 0.4.15
```

This method is lightweight and ideal for scripting or for integrating Dockle into local or CI/CD workflows without using a package manager.

### Docker

If you prefer not to install Dockle locally, you can use Docker as an alternative. This method is fully cross‑platform and particularly convenient in CI/CD pipelines or ephemeral environments.

#### Pull the Dockle Docker image

Before you can use Dockle in a container, you need to pull the official image from Docker Hub. This ensures you are using a recent version and avoids installation dependencies on your local system. To download the official Dockle image from Docker Hub, run:


```bash
  docker pull goodwithtech/dockle:latest
```

## How to use Dockle to scan a Docker image

Let’s start by downloading the latest official `mongo` Docker image using the `docker pull` command:

```bash
  docker pull mongo:latest
```
This image is maintained by the MongoDB team and includes everything needed to run a MongoDB instance in a container. By using the `latest` tag we get the most recent stable version, although Dockle will later recommend using a fixed tag for better traceability.

Once the image is downloaded, we can scan it with Dockle to evaluate its security posture and ensure it follows container best practices before using it in a production environment:

```bash
  docker run --rm goodwithtech/dockle:latest mongo:latest
```

Example output (truncated for brevity):

```bash
  WARN - CIS-DI-0001: Create a user for the container
      * Last user should not be root
  WARN - DKL-DI-0006: Avoid latest tag
      * Avoid 'latest' tag
  INFO - CIS-DI-0005: Enable Content trust for Docker
      * export DOCKER_CONTENT_TRUST=1 before docker pull/build
  INFO - CIS-DI-0006: Add HEALTHCHECK instruction to the container image
      * not found HEALTHCHECK statement
  INFO - CIS-DI-0008: Confirm safety of setuid/setgid files
      * setuid file: urwxr-xr-x usr/bin/newgrp
      * setuid file: urwxr-xr-x usr/bin/mount
      * setuid file: urwxr-xr-x usr/bin/chfn
      * setuid file: urwxr-xr-x usr/bin/chsh
      ...
```

The output provided by Dockle is a categorised list of findings. It warns that the container is using the root user (CIS-DI-0001), which is a common security concern, and advises against using the latest tag (DKL-DI-0006) for better version control and traceability. It also highlights the absence of a HEALTHCHECK instruction, which helps ensure the container’s runtime health can be monitored effectively.

In a real project, you would typically address these findings by creating a custom Docker image based on mongo, adding a non‑root user, pinning a specific image tag and defining an appropriate HEALTHCHECK before running Dockle again.

## How to export Dockle scan results

If you want to store your Dockle scan results for further analysis, auditing or integration with other tools, Dockle provides built-in support for exporting output in structured formats such as JSON or SARIF.

To export Dockle scan results in JSON format:

```bash
  docker run --rm -v "$(pwd)":/output goodwithtech/dockle:latest -f json -o /output/scan_results.json mongo:latest
```

Once you have exported the scan results to a file, you can use the cat command to view the contents directly in your terminal:


```bash
  cat scan_results.json
```

The resulting JSON file contains a summary of the number of findings per level (fatal, warn, info, skip, pass) and a detailed list of checks, including their code, title, severity level and associated alerts. This structure makes it easy to parse the results programmatically, feed them into dashboards or import them into other security tools.

In more advanced setups you can also export Dockle results in SARIF format, which is a standard format for static analysis tools and can be consumed by platforms such as GitHub code scanning or other security dashboards.

## Configuring Dockle

Dockle is a flexible tool that allows you to fine‑tune its security scanning behaviour to fit the needs of your project. By creating a `.dockleignore` file in your repository, you can control which checks Dockle evaluates or ignores for your Docker images.

This file lets you specify checks to skip by their code, so you can bypass findings that are not relevant for a particular image or that you have consciously accepted as risk.

Here is an example of how you can configure Dockle using a `.dockleignore` file:

```bash
  cat .dockleignore
```

```
  # .dockleignore
  #
  # Ignore specific checks by their code
  CIS-DI-0001  # Ignore warning about using root as the user in the container
  CIS-DI-0006  # Ignore warning about missing HEALTHCHECK instruction
  DKL-DI-0006  # Ignore warning about using the 'latest' tag
  #
  # Additional checks to ignore as needed
  CIS-DI-0005  # Ignore the check related to enabling Content Trust for Docker
  CIS-DI-0008  # Ignore setuid/setgid file safety checks
  #
  # Include any other checks that are irrelevant or need to be skipped for your image
```
  As with any ignore file, use `.dockleignore` sparingly: it is better to fix important findings where possible and only ignore checks that you fully understand and have documented as accepted risk.

## CI/CD integration: Using Dockle with GitHub Actions

Automating Docker image security checks as part of your CI/CD pipeline is an essential step to ensure that best practices and security guidelines are always followed. GitHub Actions provides a convenient way to automate this process using Dockle.

The following example workflow builds a Docker image and then scans it with Dockle on every push to main or any pull request that touches a Dockerfile:

```yaml
  name: Scan Docker image with Dockle

  on:
    pull_request:
      paths:
        - '**/Dockerfile'
    push:
      branches:
        - main
      paths:
        - '**/Dockerfile'

  jobs:
    dockle-scan:
      runs-on: ubuntu-latest

      steps:
        - name: Checkout code
          uses: actions/checkout@v4

        - name: Build Docker image
          run: docker build -t my-image:latest .

        - name: Run Dockle
          run: |
            docker run --rm \
              -v /var/run/docker.sock:/var/run/docker.sock \
              goodwithtech/dockle:latest \
              my-image:latest
```

With this setup, GitHub Actions builds your image and then runs Dockle against it. If Dockle finds issues such as running as root, missing health checks or leftover sensitive files, the job will fail and the workflow logs will show the detailed findings.

For more advanced use cases you can replace the raw docker run step with the official dockle GitHub Action, which adds options such as configurable exit levels and output formats, and can produce JSON or SARIF reports that integrate with GitHub code scanning.

## Frequently asked questions (FAQs)

  - **What is Dockle used for?**  
    Dockle is a post‑build Docker image scanner that checks container images for security misconfigurations and best‑practice violations. It helps developers and DevOps teams ensure images are production‑ready and aligned with guidelines such as the CIS Docker Benchmark.

  - **How is Dockle different from Hadolint?**  
    Dockle analyses built Docker images, while Hadolint is a Dockerfile linter that checks Dockerfile syntax and best practices before the image is built. Using both tools together gives you coverage across both the build instructions and the final image.

  - **Is Dockle a vulnerability scanner?**  
    No. Dockle focuses on configuration‑level issues, such as containers running as root or lacking health checks. For vulnerability scanning (CVEs), you should use tools like Trivy or Grype.

  - **Can Dockle be used in CI/CD pipelines?**  
    Yes. Dockle is designed to integrate easily into CI/CD workflows such as GitHub Actions, GitLab CI, CircleCI, Jenkins and others. You can add Dockle as a step in your build and deployment pipelines to automatically scan images.

  - **Does Dockle work with private Docker registries?**  
    Yes. Dockle can scan any image that your Docker daemon has access to, whether it is pulled from public registries or built locally from private repositories.

  - **How do I suppress specific warnings in Dockle?**  
    You can create a `.dockleignore` file in your project directory to skip specific checks by ID (for example, `CIS-DI-0001` or `DKL-DI-0006`). This lets you tailor Dockle to your environment or compliance requirements.

  - **Does Dockle require Docker to be installed?**  
    Not necessarily. You can run Dockle as a container using the official Docker image, so even if you do not have the Dockle binary installed natively, you can still use it where Docker is available.

## Conclusion

By integrating Dockle into your container security workflow, you can greatly enhance the quality and safety of your Docker images. Dockle helps ensure that:

  - Docker images follow container security best practices.
  - Misconfigurations such as missing HEALTHCHECK instructions or running as root are flagged early.
  - Builds are production‑ready and aligned with standards like the CIS Docker Benchmark.
  - CI/CD pipelines automatically enforce compliance and prevent insecure images from being deployed.

Dockle goes beyond traditional Dockerfile linting tools by analysing the final image, offering insights into issues that only appear post‑build. It encourages teams to adopt a proactive approach to container hardening and secure image delivery.

Whether you are working in a fast‑moving DevOps team or deploying critical infrastructure, adding Dockle to your workflow is a lightweight but powerful step toward ensuring secure, consistent and standards‑compliant Docker images. As with any tool, Dockle works best when combined with thoughtful configuration: customise checks using `.dockleignore`, and pair it with other tools like Hadolint or Trivy for full coverage across your container lifecycle.

## Resources

  - Dockle GitHub repository – Official source code, releases and documentation: https://github.com/goodwithtech/dockle
  - CIS Docker Benchmark – Industry‑recognised standards for container security practices: https://www.cisecurity.org/benchmark/docker
  - Trivy – Scans images, filesystems and Git repositories for known vulnerabilities: https://github.com/aquasecurity/trivy
  - Hadolint – Pre‑build linter for Dockerfiles to enforce syntax and security best practices: https://github.com/hadolint/hadolint
  - Grype – Open‑source tool for scanning container images for vulnerabilities with SBOM support: https://github.com/anchore/grype
  - Docker Hub – Central repository for Docker images, including official containers like `mongo`: https://hub.docker.com/
  - GitHub Actions documentation – Learn how to automate workflows and integrate Dockle into your CI/CD pipeline: https://docs.github.com/en/actions

