# GitHub-Workflows
A collection of useful GitHub actions/workflows

## Prerequisites:
- A created repository
- Under repository settings, navigate to **Actions** > **General** and ensure the following options are checked and saved:
  - **Allow actions and reusable workflows**
  - **Read and write permissions**: Workflows have read and write permissions in the repository for all scopes.

## Getting Started:

### Automated Security Helper (ASH)

#### Description:
The security helper tool was created to help you reduce the probability of a security violation in new code, infrastructure, or IAM configurations by providing a fast and easy tool to conduct preliminary security checks as early as possible within your development process.

The security helper supports the following vectors:

- **Code**
  - **Git**
    - `git-secrets`: Finds API keys, passwords, AWS keys in the code.
  - **Python**
    - `bandit`: Finds common security issues in Python code.
    - `Semgrep`: Finds common security issues in Python code.
    - `Grype`: Vulnerability scanner for Python code.
    - `Syft`: Generates a Software Bill of Materials (SBOM) for Python code.
  - **Jupyter Notebook**
    - `nbconvert`: Converts Jupyter Notebook (ipynb) files into Python executables. Scans code with Bandit.
  - **JavaScript; NodeJS**
    - `npm-audit`: Checks for vulnerabilities in JavaScript and NodeJS.
    - `Semgrep`: Finds common security issues in JavaScript code.
    - `Grype`: Vulnerability scanner for JavaScript and NodeJS.
    - `Syft`: Generates a Software Bill of Materials (SBOM) for JavaScript and NodeJS.
  - **Go**
    - `Semgrep`: Finds common security issues in Golang code.
    - `Grype`: Vulnerability scanner for Golang.
    - `Syft`: Generates a Software Bill of Materials (SBOM) for Golang.
  - **C#**
    - `Semgrep`: Finds common security issues in C# code.
  - **Bash**
    - `Semgrep`: Finds common security issues in Bash code.
  - **Java**
    - `Semgrep`: Finds common security issues in Java code.
    - `Grype`: Vulnerability scanner for Java.
    - `Syft`: Generates a Software Bill of Materials (SBOM) for Java.
- **Infrastructure**
  - **Terraform; CloudFormation**
    - `checkov`
    - `cfn_nag`
    - `cdk-nag`: Via importing rendered CloudFormation templates into a custom CDK project with the AWS Solutions NagPack enabled.
  - **Dockerfile**
    - `checkov`

To deploy ASH locally or for more information, visit:
[Automated Security Helper GitHub Repository](https://github.com/awslabs/automated-security-helper)

### Deploying ASH Workflow
1. Under **Actions**, create a new workflow.
2. Rename the file to `run-ash-scan.yaml`. Keep the default directory.
3. Paste the following code into the file:

```yaml
name: ASH SAST Scan

on:
  push:
    branches: [ '**' ]

env:
  ASH_OUTPUT_PATH: ash_output

jobs:
  containerjob:
    name: Run ASH Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout ASH
        uses: actions/checkout@v4
        with:
          path: ./automated-security-helper
          repository: awslabs/automated-security-helper
          ref: v1.3.3
      - name: Checkout app repo
        uses: actions/checkout@v4
        with:
          path: ./repo
      - name: Run ASH scan against repo
        run: |
          export PATH="$(pwd)/automated-security-helper:$PATH"

          ash \
            --source-dir "$(pwd)/repo" \
            --output-dir "${{ env.ASH_OUTPUT_PATH }}"

      - name: Publish ${{ env.ASH_OUTPUT_PATH }}
        uses: actions/upload-artifact@v4
        if: success() || failure()
        with:
          name: ${{ env.ASH_OUTPUT_PATH }}
          path: ${{ env.ASH_OUTPUT_PATH }}

The workflow will execute and scan each time there is a push to the repository.

### Yor by Bridgecrew.io
#### Description:
Yor is an open-source tool that helps add informative and consistent tags across Infrastructure as Code (IaC) frameworks. Yor currently supports Terraform, CloudFormation, and Serverless Frameworks.

Yor can run as a GitHub Action to automatically add consistent tagging logic to your IaC. It can also be used as a pre-commit hook or a standalone CLI.

For more information, visit: Yor GitHub Repository

#### Deploying Yor Workflow
Create a new workflow under Actions in your repository.
Rename the file to Yor GitHub Action.yaml. Leave the directory as is.
Paste the following code into the file:
yaml
name: Yor GitHub Action

on:
  push:
    branches:
      - main

jobs:
  yor-job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Run yor action
        uses: bridgecrewio/yor-action@0.3.0
The branch can be changed to reflect the branch where resources need tagging.
Tags can be customized by adding the following snippet to the workflow file (GitHub tagging used as an example):
yaml
      tag: git_modifiers,git_commit,git_repository,yor_trace
      tag_groups: git,code2cloud
      custom_tags: path/to/plugin.so
      output_format: json
