Project Name: Jenkins with GitHub for SFDX
**1. Branching Strategy and Branch Protection**
Our repository uses a structured branching model to maintain a stable and controlled release process.

main: This branch represents our production-ready code. Direct commits are strictly prohibited. All changes must be merged via a pull request.

develop: This is our main integration branch. All new feature development starts here, and all completed features are merged here.

feature/<feature-name>: All new features and bug fixes are developed in these branches, which are created from develop.

We enforce the following branch protection rules in GitHub to maintain code quality and stability:

Pull Request Required: All changes to develop and main must be submitted via a pull request.

Status Checks Required: The Jenkins pipeline must successfully pass all checks before a pull request can be merged. This ensures a clean validation and deployable state.

Require Reviewers: A designated number of reviewers must approve the pull request.

**2. PR Validation with Jenkins**
This project is configured with a Jenkins pipeline that automatically validates every pull request (PR) to prevent breaking changes from being merged.

Trigger: The Jenkins pipeline is triggered automatically on every PR creation and update (via the GitHub Pull Request Builder plugin).

Validation Deployment: The pipeline performs a validation-only deployment (--checkonly) to a Salesforce org. This step confirms that the metadata is deployable without introducing errors.

PMD Static Code Analysis: The pipeline runs a PMD analysis on all Apex classes. It fails the build if any code quality issues are found, ensuring only clean code is merged.

Automated Tests: The pipeline executes all Apex tests to check for regressions.

**3. PMD Integration for Code Quality**
We use PMD to perform static code analysis on our Apex code as part of the Jenkins pipeline.

The Jenkinsfile downloads and runs PMD with a predefined ruleset (apex-ruleset.xml).

The pipeline will fail the build if PMD detects any issues, providing immediate feedback to the developer.

A detailed PMD report is generated as a build artifact, which can be viewed in Jenkins to see a list of all detected issues.

Email notifications are configured to be sent automatically upon a build failure, alerting the relevant team members.
