---
layout: post
title: "Scan Docker Images for Vulnerabilities with Trivy (CLI and GitHub Actions)"
date: 2026-02-03
---
Building secure containers isn’t just about writing good code, it’s also about knowing what’s inside your images. Even small, trusted base images can hide critical vulnerabilities and that’s where Trivy comes in.

Trivy is a fast, lightweight and easy to use vulnerability scanner for Docker images and other artifacts. With just one command, you can check your containers for known security issues in operating system packages, libraries or application dependencies, and even scan for secrets or IaC misconfigurations. In this guide, we’ll focus on scanning Docker images from the CLI and GitHub Actions, and see how to tune Trivy with ignore files and configuration.

> **What is Trivy used for?**
Trivy is a vulnerability and security scanner that analyzes Docker and OCI images, filesystems and repositories for known CVEs, secrets and misconfigurations. It helps you detect issues like vulnerable OS packages or libraries, leaked credentials, and insecure IaC or Kubernetes manifests before your containers are deployed to production.

## Table of contents

- [What is Trivy?](#what-is-trivy)
- [Why scan Docker images?](#why-scan-docker-images)
- [Key components Trivy scans in container images](#key-components-trivy-scans-in-container-images)
- [Understanding Trivy vulnerability severity levels](#understanding-trivy-vulnerability-severity-levels)
- [Installing Trivy](#installing-trivy)
  - [macOS (via Homebrew)](#macos-via-homebrew)
  - [macOS Binary](#macos-binary)
  - [Docker](#docker)
- [How Trivy works](#how-trivy-works)
- [Scanning Docker images with Trivy](#scanning-docker-images-with-trivy)
- [Ignoring vulnerabilities with .trivyignore](#ignoring-vulnerabilities-with-trivyignore)
- [Customizing Trivy with a configuration file](#customizing-trivy-with-a-configuration-file)
- [Integrating Trivy With GitHub actions](#integrating-trivy-with-github-actions)
- [Frequently asked questions (FAQs)](#frequently-asked-questions-faqs)
- [Conclusion](#conclusion)
- [Resources](#resources)

## What Is Trivy?

[Trivy](https://trivy.dev/latest/) is an open-source security scanner created by [Aqua Security](https://www.aquasec.com/) that helps developers and DevOps teams find vulnerabilities and misconfigurations early in the delivery pipeline. It focuses on simplicity: you point it at an artifact, and it tells you what’s wrong.

Trivy can scan a wide range of targets, including:

- [Docker](https://www.docker.com/) and [OCI](https://opencontainers.org/) container images.
- Local filesystems and [Git](https://git-scm.com) repositories.
- [Kubernetes](https://kubernetes.io/) manifests and [Helm charts](https://helm.sh/docs/topics/charts/).
- Infrastructure as Code (IaC) such as [Terraform](https://www.hashicorp.com/en/products/terraform).

Under the hood, Trivy analyzes operating system packages, libraries, and application dependencies and compares them against continuously updated vulnerability databases (NVD, distro feeds and vendor advisories). It also detects secrets and misconfigurations, then assigns a severity to each finding so you can decide what to fix first.

In practice, this means you can run a single command like `trivy image your-image:tag` during development or in CI and immediately see whether a given Docker image is safe to ship.

## Why scan Docker images?

Scanning Docker images is critical for securing your applications because every image you build or pull bundles OS packages, language runtimes and dependencies that may contain known vulnerabilities. Even minimal base images like _Alpine Linux_ have had serious issues in the past, so “small” does not mean “safe”.

By running Trivy as part of your workflow, you can:

- Detect security issues early in development, before images are pushed to a registry or deployed.
- Automate vulnerability checks in CI/CD pipelines and use exit codes to block risky releases.
- Prioritize fixes using severity levels (CRITICAL, HIGH, MEDIUM, LOW) instead of treating every finding the same.
- Continuously re-scan existing images as new CVEs appear, without changing your application code.

In practice this reduces the chance of shipping exploitable packages, helps you meet compliance requirements around container security, and gives your team a clear, repeatable process for keeping images up to date and safe.

## Key components Trivy scans in container images

When you scan a container image with Trivy, it looks inside the filesystem and the image configuration to find different kinds of security issues. The main components it inspects are:

- **OS packages and package managers:** Trivy detects vulnerabilities in packages installed via `apk`, `apt`, `yum` and others (e.g. BusyBox, OpenSSL, zlib), which suelen venir del propio base image.
- **Application dependencies:** It scans language-specific dependencies (for example `pip`, `npm`, `bundler`, `pipenv`) to catch issues in your application stack, not just in the underlying OS.
- **Image configuration and metadata:** Trivy can inspect the image config for risky settings and secrets embedded in environment variables, labels or entrypoint commands.
- **Secrets in files:** By default it also looks for hard-coded credentials such as API keys, tokens or private keys inside files packaged in the image.
- **Misconfigurations (optional scanners):** You can enable additional checks for Dockerfile and IaC misconfigurations via `--scanners misconfig` or `--scanners vuln,misconfig,secret` when you want a broader security review from the same tool.

Having all of these checks in a single scan means you get a consolidated view of OS CVEs, library issues, leaked secrets and configuration problems for each image you build or pull.

## Understanding Trivy vulnerability severity levels

Before showing individual findings, Trivy maps each vulnerability to a severity level so you can decide what to fix first. These levels are usually derived from CVSS scores provided by upstream sources.

A typical mapping looks like this:

```bash
| CVSS base score | Severity  |
|-----------------|-----------|
| 9.0–10.0        | CRITICAL  |
| 7.0–8.9         | HIGH      |
| 4.0–6.9         | MEDIUM    |
| 0.1–3.9         | LOW       |
```
In practice you’ll often run image scans focusing only on the top end, for example:

```bash
pgarcia@AnnMargret:~$ trivy image --severity HIGH,CRITICAL your-image:tag
```
This lets you keep your reports actionable in CI/CD by failing builds only when serious issues are detected, while still allowing you to review Medium and Low findings periodically. 

## Installing Trivy

You can install Trivy in multiple ways depending on your environment and workflow. This makes it easy to use both on developer machines and in automated CI/CD pipelines.

### macOS (via Homebrew)

If you don’t have Trivy installed yet, you can install it on macOS with Homebrew:

```bash
pgarcia@AnnMargret:~$ brew install trivy
```

### macOS Binary

Trivy can be downloaded as a standalone binary from the GitHub releases page, which is ideal for scripts or CI environments where a package manager or Docker might not be available.

Go to the official Trivy GitHub releases page and find the latest version for your platform.

Download the correct binary for your OS and architecture, for example with ```curl```:

```bash
pgarcia@AnnMargret:~$ curl -L "https://get.trivy.dev/trivy?type=tar.gz&version=0.63.0&os=macos&arch=amd64" -o trivy.tar.gz
```

* **Extract the contents of the tar.gz file -** Extract the contents of the archive and makes the Trivy binary available in your current directory for use or installation.

```bash
pgarcia@AnnMargret:~$ tar -xvzf trivy.tar.gz
```

* **Make it executable and move to your _PATH_ -** After downloading, you'll need to assign execution permissions and move it to a directory included in your system's PATH. This allows you to run Trivy from anywhere in your terminal.

```bash
pgarcia@AnnMargret:~$ chmod +x trivy && sudo mv trivy /usr/local/bin/trivy
```

Once the installation is complete, verify that Trivy installation was successful by entering the following command in the terminal:

```bash
pgarcia@AnnMargret:~$ trivy --version
Version: 0.63.0
Java DB:
  Version: 1
  UpdatedAt: 2025-05-08 03:45:11.673792333 +0000 UTC
  NextUpdate: 2025-05-11 03:45:11.673792173 +0000 UTC
  DownloadedAt: 2025-05-08 17:04:40.758671 +0000 UTC
```

This method is lightweight and ideal for scripting or integration with local or CI/CD workflows without requiring Docker.

### Docker

If you prefer not to install Trivy locally, you can run it as a Docker container. This is a common option in CI/CD or ephemeral environments.
First, pull the official image from Docker Hub:

```bash
pgarcia@AnnMargret:~$ docker pull aquasec/trivy:latest
```
Then you can run scans by mounting the Docker socket or target directories into the Trivy container, depending on whether you want to scan local images or filesystems.

## How Trivy works

When you run Trivy for the first time, it downloads its vulnerability databases so it can match packages and dependencies against the latest known CVEs. You can skip this step on later runs with flags like `--skip-update` if needed, for example in air‑gapped or performance-sensitive environments.

For container images, Trivy works roughly in three stages:[web:290][web:328]

1. **Fetch image and layers:** Trivy pulls the image (locally or from a registry) and reads its manifest, config and filesystem layers.
2. **Analyze packages and files:** It walks the file tree inside the image, detects OS packages and application dependencies, and optionally looks for secrets or misconfigurations.
3. **Match against databases and report:** Detected components are compared with the local vulnerability database, each issue is assigned a severity, and the results are rendered in the chosen format (table, JSON, SARIF, etc.).

Trivy also caches analysis results per image and per layer, which means subsequent scans of the same image—or images that share layers—are much faster. This makes it practical to run image scans on every commit or pull request without adding too much overhead to your pipeline. 

## Scanning Docker images with Trivy

To see Trivy in action, let’s start with a small base image like Alpine Linux. Even though it’s known for being minimal, it can still contain vulnerable packages, so it’s a good example to scan regularly.

First, pull the image you want to analyze, for example `alpine:3.18.12`:

```bash
pgarcia@AnnMargret:~$ docker pull alpine:3.18.12
```

Once the image is available locally, you can run a basic vulnerability scan with Trivy:

```bash
pgarcia@AnnMargret:~$ trivy image alpine:3.13.2
2025-02-17T16:30:02+01:00   INFO    [vuln] Vulnerability scanning is enabled
2025-02-17T16:30:02+01:00   INFO    [secret] Secret scanning is enabled
2025-02-17T16:30:02+01:00   INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-02-17T16:30:02+01:00   INFO    [secret] Please see also https://aquasecurity.github.io/trivy/v0.59/docs/scanner/secret#recommendation for faster secret detection
2025-02-17T16:30:03+01:00   INFO    Detected OS family="alpine" version="3.13.2"
2025-02-17T16:30:03+01:00   INFO    [alpine] Detecting vulnerabilities...   os_version="3.13" repository="3.13" pkg_num=14
2025-02-17T16:30:03+01:00   INFO    Number of language-specific files   num=0
2025-02-17T16:30:03+01:00   WARN    This OS version is no longer supported by the distribution  family="alpine" version="3.13.2"
2025-02-17T16:30:03+01:00   WARN    The vulnerability detection may be insufficient because security updates are not provided

alpine:3.13.2 (alpine 3.13.2)

Total: 4 (CRITICAL: 4)

┌──────────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┬──────────────────────────────────────────────────────────────┐
│   Library    │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                            Title                             │
├──────────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ apk-tools    │ CVE-2021-36159 │ CRITICAL │ fixed  │ 2.12.1-r0         │ 2.12.6-r0     │ libfetch: an out of boundary read while libfetch uses strtol │
│              │                │          │        │                   │               │ to parse...                                                  │
│              │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2021-36159                   │
├──────────────┼────────────────┤          │        ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libcrypto1.1 │ CVE-2021-3711  │          │        │ 1.1.1j-r0         │ 1.1.1l-r0     │ openssl: SM2 Decryption Buffer Overflow                      │
│              │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2021-3711                    │
├──────────────┤                │          │        │                   │               │                                                              │
│ libssl1.1    │                │          │        │                   │               │                                                              │
│              │                │          │        │                   │               │                                                              │
├──────────────┼────────────────┤          │        ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ zlib         │ CVE-2022-37434 │          │        │ 1.2.11-r3         │ 1.2.12-r2     │ zlib: heap-based buffer over-read and overflow in inflate()  │
│              │                │          │        │                   │               │ in inflate.c via a...                                        │
│              │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2022-37434                   │
└──────────────┴────────────────┴──────────┴────────┴───────────────────┴───────────────┴──────────────────────────────────────────────────────────────┘
```

By default, Trivy will show all detected vulnerabilities across severities. If you want to focus only on the most serious issues, you can filter by severity:

```bash
pgarcia@AnnMargret:~$ trivy image --severity HIGH,CRITICAL alpine:3.18.12
```
You can also ignore vulnerabilities that don’t have a fixed version yet, which is useful to keep CI output focused on actionable problems:

```bash
pgarcia@AnnMargret:~$ trivy image --severity HIGH,CRITICAL --ignore-unfixed alpine:3.18.12
```
In the report, pay attention to fields like the package name, vulnerability ID (CVE), installed version and fixed version. These tell you exactly which package needs to be upgraded in your base image or Dockerfile to remediate the issue.

## Ignoring vulnerabilities with .trivyignore

Sometimes you’ll want to ignore specific vulnerabilities in your reports, for example when a CVE is a known false positive, has low practical impact for your use case, or is already being tracked elsewhere.

The simplest way to do this in Trivy is with a `.trivyignore` file placed at the root of your project. Each line contains a single ID (for example a CVE or AVD ID), and any matching findings will be suppressed in the output:

```bash
CVE-2024-12345
CVE-2024-67890
CVE-2025-12345
CVE-2025-67890
```
You can create this file manually based on previous scan results, or generate it as part of a triage step so your regular scans stay focused on new and relevant issues.

By default, Trivy will pick up `.trivyignore` from the current working directory, but you can also point to a specific file path with the `--ignorefile` flag:

```bash
pgarcia@AnnMargret:~$ trivy image --ignorefile .trivyignore alpine:3.18.12
```
Used carefully, ignore files help you keep vulnerability reports actionable without hiding truly critical problems that still need to be fixed.

## Customizing Trivy with a configuration file

For more advanced use cases, Trivy supports a configuration file in YAML. This lets you centralize options like scanners, severities, directories to skip and output format, instead of repeating flags in every command.

By default, Trivy looks for `trivy.yaml` or `.trivy.yaml` in the current working directory, but you can also specify a custom file with the `--config` flag.

Here is a sample configuration:

```yaml
scan:
  scanners:
    - vuln
    - secret
  skip-dirs:
    - node_modules
  skip-files:
    - config/dev.env

vulnerability:
  ignore-unfixed: true
  severity:
    - CRITICAL
    - HIGH
  skip-cve-ids:
    - CVE-2024-12345
    - CVE-2024-67890
    - CVE-2025-12345
    - CVE-2025-67890

output:
  format: table
  template: ""
```
With this configuration, Trivy will scan only for vulnerabilities and secrets, skip noisy paths, focus on HIGH and CRITICAL issues, ignore some known CVEs and render results as a table. To use it during a scan, just run:

To use it during a scan:

```bash
pgarcia@AnnMargret:~$ trivy image --config trivy.yaml alpine:3.18.12
```
## Integrating Trivy with GitHub Actions

Automating container image vulnerability scanning as part of your CI/CD pipeline is crucial for maintaining a secure environment. GitHub Actions makes it straightforward to run Trivy on every push or pull request and surface results directly in the repository.

Start by creating a `.github/workflows/trivy-scan.yml` file in your repository. The example below scans both the repository contents (filesystem mode) and the built Docker image, then uploads results to the Security tab using SARIF:

```yaml
name: trivy-scanning

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

permissions:
  contents: read
  security-events: write

jobs:
  trivy-scanning-job:
    name: trivy-sec-scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner in repo (fs) mode
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: 'fs'
          scan-ref: '.'
          ignore-unfixed: true
          format: 'sarif'
          output: 'trivy-fs-results.sarif'
          severity: 'HIGH,CRITICAL'
          exit-code: '0'

      - name: Upload Trivy fs scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-fs-results.sarif'

      - name: Build Docker image
        run: |
          docker build -t docker.io/devops-counsel/py-app:${{ github.sha }} .

      - name: Run Trivy vulnerability scanner on Docker image
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: 'image'
          image-ref: 'docker.io/devops-counsel/py-app:${{ github.sha }}'
          vuln-type: 'os,library'
          severity: 'HIGH,CRITICAL'
          format: 'sarif'
          output: 'trivy-image-results.sarif'
          ignore-unfixed: true
          exit-code: '1'

      - name: Upload Trivy image scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-image-results.sarif'
```

After the workflow is set up, [GitHub Actions](https://github.com/features/actions) will run Trivy on your Docker images each time a pull request is opened or code is pushed. You can view the results of the vulnerability scan in the Actions tab of your GitHub repository.

If any vulnerabilities are found, the job will fail, and you’ll see the specific security issues and their severity levels from Trivy in the logs. You can then address the vulnerabilities in your Docker image and push the necessary changes to your repository.

This process helps ensure that your container images are always secure and free from critical vulnerabilities before they are deployed to production.

## Frequently Asked Questions (FAQs)

**What does Trivy scan for?**  
Trivy scans for known vulnerabilities in operating system packages, application dependencies, container images, filesystems, Infrastructure as Code (IaC) files, Kubernetes manifests and Git repositories. It can also detect hard-coded secrets such as API keys or passwords.

**Can Trivy be run inside a Docker container?**  
Yes. You can run Trivy using the official `aquasec/trivy` image without installing it locally, which is especially convenient in CI/CD pipelines or ephemeral environments.

**Is Trivy suitable for CI/CD integration?**  
Trivy is widely used in CI/CD pipelines to automatically scan container images, repositories and IaC files on each push or pull request. It integrates well with platforms like GitHub Actions, GitLab CI or CircleCI.

**How often is Trivy’s vulnerability database updated?**  
Trivy updates its vulnerability databases multiple times per day from upstream sources such as NVD, GitHub advisories and distribution feeds, so new CVEs are picked up quickly.

**Can I ignore certain vulnerabilities or configure custom policies?**  
Yes. You can use `.trivyignore` files and YAML configuration to exclude specific CVEs, control which severities are reported, skip directories and choose output formats, adapting Trivy to your project’s risk profile.

**Does Trivy support SBOMs?**  
Trivy can generate and consume Software Bill of Materials (SBOMs) in formats such as SPDX or CycloneDX, which helps with supply chain security and compliance reporting.

## Conclusion

Trivy is a powerful yet easy to adopt tool for securing container images. Its fast scans, broad coverage and flexible integrations make it a natural fit for both developers and DevOps teams.

By running Trivy locally and wiring it into your CI/CD pipeline, you ensure that every image you build is checked for known vulnerabilities, misconfigurations and even leaked secrets before it reaches production. This reduces the risk of shipping exploitable software and gives you confidence that your base images and dependencies are continuously monitored.

Combined with features like `.trivyignore`, YAML configuration and GitHub Actions integration, Trivy lets you start simple and gradually grow into a more opinionated security posture as your needs evolve.

## Resources

- **Trivy GitHub repository** – https://github.com/aquasecurity/trivy  
- **Trivy documentation site** – https://aquasecurity.github.io/trivy/  
- **Trivy vulnerability database** – https://github.com/aquasecurity/vuln-list  
- **Trivy Docker Hub** – https://hub.docker.com/r/aquasec/trivy

