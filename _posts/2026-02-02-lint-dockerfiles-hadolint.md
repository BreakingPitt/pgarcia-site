---
layout: post
title: "Linting Dockerfiles with Hadolint: Catching Problems Before You Build"
date: 2026-02-02
---

Hadolint is a Dockerfile linter that checks your Dockerfiles for common issues, helps you follow best practices and keeps your images small and secure. Whether you are just starting with Docker or already deep into DevOps, Hadolint gives you fast, actionable feedback on your builds before problems reach production.

>**What is Hadolint used for?**
Hadolint is used to enforce Dockerfile best practices automatically, instead of relying on manual reviews or trial and error. It flags problems like using generic base images, mixing package manager commands across multiple RUN layers, forgetting to clean caches or running containers as root, so you can standardise how Dockerfiles are written across your team and avoid surprises later in CI or production.

## Table of contents
- [What is Hadolint?](#what-is-hadolint)
- [What is a linter?](#what-is-a-linter)
- [Overview of linter types](#overview-of-linter-types)
- [Why lint Dockerfiles?](#why-lint-dockerfiles)
- [Installing Hadolint](#installing-hadolint)
  - [macOS (via Homebrew)](#macos-via-homebrew)
  - [macOS binary](#macos-binary)
  - [Docker](#docker)
- [Example Dockerfile with common issues](#example-dockerfile-with-common-issues)
- [Linting a Dockerfile with Hadolint](#linting-a-dockerfile-with-hadolint)
- [Fixing common Hadolint warnings](#fixing-common-hadolint-warnings)
- [Configuring Hadolint](#configuring-hadolint)
- [CI/CD integration with Hadolint and GitHub Actions](#cicd-integration-with-hadolint-and-github-actions)
- [Frequently asked questions (FAQs)](#frequently-asked-questions-faqs)
- [Conclusion](#conclusion)
- [Resources](#resources)


## What is Hadolint?
Hadolint is an open-source Dockerfile linter that performs static analysis on your Dockerfiles to catch common errors, security issues and bad practices before you build an image. Its goal is to help you apply official Docker best practices so your images are smaller, more secure and easier to maintain.
Instead of manually reviewing each Dockerfile, Hadolint parses it, applies a set of rules and points out problems such as using generic base images, running as root, leaving package caches behind or using deprecated instructions. Because its based on ShellCheck, it can also analyse the shell code inside RUN instructions, detecting bugs and risky patterns in inline scripts as well.

## What is a linter?

A linter is a static analysis tool that scans source code to identify issues such as syntax errors, potential bugs, stylistic inconsistencies and violations of coding standards. Instead of running your code, it analyses it and compares it against a set of rules to highlight problems early in the development process.

These tools play a crucial role in modern software development by helping teams catch problems early, enforce consistent coding practices and maintain high‑quality codebases. By integrating linters into the development workflow, you can reduce technical debt, improve readability and prevent defects before they reach production.

Some key aspects of linters include:

  - **Static analysis** – Linters analyse code without executing it, which allows them to uncover bugs, syntax errors and design flaws that might not be immediately visible at runtime.
  - **Comprehensive error detection** – They can spot everything from simple syntax mistakes and unused variables to more complex issues such as security vulnerabilities or performance anti‑patterns.
  - **Improved code quality** – By enforcing consistent coding standards and best practices, linters promote a clean, maintainable codebase.
  - **High configurability** – Most linters offer flexible configuration options so you can define custom rules, adjust severity levels and tailor the tool to your project or organisation.
  - **Seamless automation** – Linters integrate easily into editors and CI/CD pipelines, enabling automatic checks during development, on every commit or before merging code, so issues are caught before they reach production.

## Overview of linter types

Linters come in various forms, each serving different needs depending on the programming language, project scope or specific areas of concern such as security or formatting. Understanding the main categories helps teams choose the right tool for their workflow.

The most common types of linters include:

  - **Language-specific linters** – Tailored for individual programming languages and able to provide deep, syntax‑aware analysis. Examples include ESLint for JavaScript, Pylint for Python or RuboCop for Ruby.
  - **General-purpose linters** – Designed to support multiple languages and offer broad code quality checks across a codebase that uses more than one stack. These tools are useful when you want consistent standards across different services or components.
  - **Specialised linters** – Focused on particular aspects of code, such as security, style or formatting. For example, some linters specialise in security rules, while others only enforce formatting or naming conventions.

  Hadolint fits into this last category: it is a specialised linter focused on Dockerfiles, combining Docker best‑practice checks with ShellCheck‑based analysis for any shell code inside RUN instructions.

## Why lint Dockerfiles?

Writing Dockerfiles is relatively simple, but doing it well requires discipline and attention to best practices. Without linting, it is easy to introduce subtle issues that later become bloated images, slow builds or hard‑to‑trace security problems in production.

Hadolint helps you avoid these problems by catching them early. Linting your Dockerfiles ensures that you:

  - Reduce image size by eliminating unnecessary layers and commands.
  - Strengthen security by avoiding root usage, outdated base images and risky patterns.
  - Improve consistency and reproducibility with pinned versions and predictable builds.
  - Eliminate deprecated or unsafe Dockerfile instructions before they reach production.
  - Improve maintainability and collaboration by enforcing a common set of standards.

In real‑world scenarios, these benefits translate into:

  - Faster builds and deployments thanks to efficient caching and smaller images.
  - Fewer security vulnerabilities because common misconfigurations are fixed early.
  - Higher developer confidence as linting becomes part of your CI/CD automation.
  - More resilient infrastructure by reducing variability across environments.

With Hadolint, linting becomes a seamless part of the Docker image build lifecycle, helping teams deliver better containers without adding unnecessary complexity to their workflow.

## Installing Hadolint

You can install Hadolint in multiple ways depending on your environment, workflow or operating system preferences. This makes it easy to integrate it both on developer machines and in automated pipelines.

### macOS (via Homebrew)

If you do not have Hadolint installed yet, you can install it on macOS with Homebrew (https://brew.sh/):

```bash
pgarcia@AnnMargret:~$ brew install hadolint
```

### macOS binary

Hadolint can be downloaded as a standalone binary for your platform from the GitHub releases page. This method is lightweight, fast and ideal for scripting or integrating into CI environments where Docker or a package manager might not be available.

To install Hadolint using the standalone binary method, follow these steps:

  - Access the releases page – Go to the official Hadolint GitHub releases page (https://github.com/hadolint/hadolint/releases), where you will find the latest versions for multiple platforms.
  - Download the correct binary – Choose the binary that matches your OS and architecture, such as hadolint-Darwin-x86_64. You can also download it directly from the command line using curl:

```
pgarcia@AnnMargret:~$ curl -L -o hadolint "https://github.com/hadolint/hadolint/releases/latest/download/hadolint-$(uname -s)-$(uname -m)"
```
  - Make it executable and move it to your PATH – After downloading, assign execution permissions and move it to a directory included in your system’s PATH. This allows you to run Hadolint from anywhere in your terminal:

```bash
pgarcia@AnnMargret:~$ chmod +x hadolint && sudo mv hadolint /usr/local/bin/hadolint
```
Once the installation is complete, verify that Hadolint was installed successfully by running:

```bash
pgarcia@AnnMargret:~$ hadolint --version
```
This method is lightweight and ideal for scripting or integrating Hadolint into local or CI/CD workflows without using a package manager.

### Docker

If you prefer not to install Hadolint locally, you can use Docker as an alternative. This method is fully cross‑platform and particularly convenient in CI/CD pipelines or ephemeral environments.

#### Pull the Hadolint Docker image

Before you can use Hadolint in a container, you need to pull the official image from Docker Hub. This ensures you are using a recent version and avoids installation dependencies on your local system. To download the official Hadolint image from Docker Hub, run:

```bash
pgarcia@AnnMargret:~$ docker pull hadolint/hadolint:latest
```

## Example Dockerfile with common issues

To better understand how Hadolint works, let us create a Dockerfile that contains several common issues and anti‑patterns. This example deliberately includes problems that Hadolint will detect:

```dockerfile
# Using a non-specific base image.
FROM python

# Maintainer information.
MAINTAINER testuser@test.com

# Running as root without necessity.
USER root

# Update package indices.Installing dependencies without cleaning the cache.
RUN apt-get update

# Installing dependencies without cleaning the cache.
Run apt-get install -y curl

# No working directory specified.
WORKDIR /

# Copying files without considering the build context.
COPY . .

# Installing global dependencies unnecessarily.
RUN pip3 install -r requirements.txt

# Exposing an unnecessary port
EXPOSE 8000

# Default command to run when the container starts
CMD python3 app.py
```

This Dockerfile contains several issues that Hadolint is able to flag, such as:

  - Use of the deprecated MAINTAINER instruction.
  - A non‑specific base image tag (`FROM python` without a version).
  - Separate RUN commands for apt‑get operations instead of a single combined layer.
  - No cleanup of the apt cache after installing packages.
  - No version pinning for packages.
  - Potentially unsafe use of the root user.
  - Inefficient copying of the entire build context with `COPY . .`.

In the next step we will run Hadolint against this Dockerfile to see how these problems are reported and how we can fix them.

## Linting a Dockerfile with Hadolint

Now that we have a Dockerfile with common issues, let us run Hadolint to see what it finds. If you installed the Hadolint binary locally, you can lint the file with:

```bash
pgarcia@AnnMargret:~$ hadolint Dockerfile
```
If you are using the Docker-based installation, the equivalent command is:

```bash
pgarcia@AnnMargret:~$ docker run --rm -v "$PWD":/mnt hadolint/hadolint hadolint /mnt/Dockerfile
```
Both commands will run the static analysis and display a list of warnings or errors directly in your terminal. A typical output for our example Dockerfile might look like this:

```bash
  Dockerfile:2 DL3006 warning: Always tag the version of an image explicitly
  Dockerfile:5 DL4000 error: MAINTAINER is deprecated
  Dockerfile:8 DL3002 warning: Last USER should not be root
  Dockerfile:11 DL3009 info: Delete the apt-get lists after installing something
  Dockerfile:15 DL3059 info: Multiple consecutive `RUN` instructions. Consider consolidation.
  Dockerfile:15 DL3008 warning: Pin versions in apt-get install. Instead of `apt-get install <package>` use `apt-get install <package>=<version>`
  Dockerfile:15 DL3015 info: Avoid additional packages by specifying `--no-install-recommends`
  Dockerfile:24 DL3042 warning: Avoid use of cache directory with pip. Use `pip install --no-cache-dir <package>`
  Dockerfile:30 DL3025 warning: Use JSON array notation for CMD and ENTRYPOINT arguments
```

Each line shows the file and line number, the rule code (for example ```DL3006``` or ```DL4000```), the severity level (```info```, ```warnin```g or ```error```) and a short description of the problem. This makes it easy to see what needs to be fixed first and to map each finding back to the corresponding rule in the Hadolint documentation.

### Fixing common Hadolint warnings

After reviewing the warnings and errors reported by Hadolint, we can refactor our Dockerfile to follow best practices. Here is an improved version that addresses the issues highlighted earlier:

```Dockerfile
  FROM python:3.11-slim

  LABEL maintainer="testuser@test.com"

  WORKDIR /app

  COPY requirements.txt .

  RUN apt-get update && \
      apt-get install -y --no-install-recommends \
        curl && \
      pip install --no-cache-dir -r requirements.txt && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/* && \
      groupadd -r appuser && \
      useradd -r -g appuser appuser && \
      chown -R appuser:appuser /app

  COPY --chown=appuser:appuser . .

  USER appuser

  EXPOSE 8000/tcp

  CMD ["python3", "app.py"]
```

This updated Dockerfile:

  - Uses a specific, slim base image to reduce size and improve reproducibility.
  - Replaces the deprecated ```MAINTAINER``` instruction with a ```LABEL```.
  - Sets a dedicated working directory with WORKDIR instead of relying on `/`.
  - Combines package installation into a single ```RUN``` layer, cleans apt caches and uses `--no-install-recommends`.
  - Uses a non-root user for the final container, reducing the impact of potential compromises.
  - Copies files with the correct ownership and documents the exposed port using the JSON form of ```CMD``` for better signal handling.

If you run Hadolint again on this version, most of the previous findings will disappear, leaving you with a cleaner, more secure and more efficient Dockerfile.

## Configuring Hadolint

Hadolint is a highly configurable tool, which means you can adapt its behaviour to your project instead of changing your project to fit the defaults. The easiest way to do this is by creating a `.hadolint.yaml` configuration file in your repository.

With this file you can control, among other things:

  - **Ignored rules** – Choose which rules to skip so that known or acceptable patterns do not clutter your output.
  - **Trusted registries** – Define which Docker registries are considered trusted so Hadolint can warn you when images come from unexpected locations.
  - **Required labels and label schema** – Enforce a consistent set of labels across all images, which is useful for compliance and operations.
  - **Severity overrides and failure thresholds** – Promote or demote specific rules and decide which severity level should actually fail your build.

Here is an example `.hadolint.yaml` configuration:

```YAML
  # .hadolint.yaml

  ignored:
    - DL3008    # Ignore version pinning for apt-get in this project
    - DL4006    # Ignore missing label warnings

  trustedRegistries:
    - docker.io
    - my-private-registry.com

  label-schema:
    description: "A custom Docker image"
    version: "1.0"
    vendor: "Your Company"

  override:
    warning:
      - DL3003   # Treat missing labels as a warning instead of an error
    error:
      - DL3026   # Ensure missing USER remains an error
```

You can store this file in your project root so that every run of Hadolint uses the same rules, or pass a custom config path via the `--config` flag when you run `hadolint`. This makes it easy to standardise Dockerfile linting across teams and CI pipelines.

## Frequently asked questions (FAQs)

  - **Can I ignore specific Hadolint rules?**  
    Yes. You can use a `.hadolint.yaml` configuration file to list rule IDs you want to ignore. This is useful for adapting Hadolint to your team’s style guide or suppressing known, acceptable patterns.

  - **Does Hadolint check shell scripts inside Dockerfiles?**  
    Hadolint uses ShellCheck under the hood to analyse shell code inside RUN instructions. It flags common scripting mistakes and unsafe patterns, improving both reliability and readability of inline scripts.

  - **How do I use Hadolint in GitLab CI or other CI systems?**  
    You can run Hadolint in any CI system by installing the binary or using the official Docker image. Add a step that runs `hadolint Dockerfile` (or the Docker-based equivalent) as part of your build pipeline and fail the job when violations are found.

  - **Is Hadolint actively maintained?**  
    Yes. Hadolint is an actively maintained open-source project. It regularly receives updates, new rules and bug fixes, and you can follow development or contribute via its GitHub repository.

  - **Can I lint multiple Dockerfiles at once?**  
    Absolutely. For example, you can use `find` to discover all Dockerfiles in a repo and run Hadolint on each one, which is especially useful in monorepos or microservice architectures.

  - **What do Hadolint severities mean?**  
    Each rule in Hadolint is classified as info, warning or error. You can override the default severity in `.hadolint.yaml` so that only the most critical issues fail your builds, while less important ones are reported as informational.

## Conclusion

By following Hadolint’s recommendations, you can significantly improve your Dockerfiles. The optimised versions are more secure (non-root users, pinned versions), more efficient (fewer layers, better caching), easier to maintain (clear structure, consistent patterns) and more reliable (reproducible builds based on explicit versions).

Beyond catching individual mistakes, Hadolint helps you adopt a mindset of building maintainable, scalable and standardised container images. As your DevOps workflows grow more complex, automating these checks in CI/CD saves time and prevents subtle issues from reaching production. Whether you work solo or in a team, integrating Hadolint into your pipeline is a small change that delivers a big improvement in Dockerfile quality.

## Resources

  - **Hadolint GitHub Repository –** [Source code, releases and documentation.](https://github.com/hadolint/hadolint)
  - **Hadolint rules –** [Full list and explanation of available rules.](https://github.com/hadolint/hadolint/blob/master/docs/RULES.md)
  - **Hadolint Docker Image in Docker Hub –** [Official Hadolint Docker image.](https://hub.docker.com/r/hadolint/hadolint)
  - **ShellCheck –** [Static analysis tool used by Hadolint for shell code.](https://www.shellcheck.net)
  - **ShellCheck GitHub Repository -** [Source code, releases and documentation.](https://github.com/koalaman/shellcheck)

