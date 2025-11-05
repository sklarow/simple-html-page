# Simple HTML Page

This repository is part of a study about `repository_dispatch` on GitHub Actions. It demonstrates how to build and upload artifacts, then trigger automated E2E tests in another repository.

## Overview

This repository contains a simple HTML page (`index.html`) that is built and uploaded as an artifact. When the artifact is created, the workflow automatically triggers E2E tests in the [test-simple-html-page](https://github.com/sklarow/test-simple-html-page) repository using GitHub's `repository_dispatch` event.

In this example, we just upload the `index.html`, but this can be extended for Android/iOS builds, if APK/IPA files are provided as artifacts at the end of the build. Or if the build deploys changes to a specific environment, you can safely assume that you can run tests on that environment with the given changes.

## How It Works

### This Repository

This repository has a workflow (see [`.github/workflows/build.yml`](.github/workflows/build.yml)) that:

1. **Builds and uploads** an artifact containing `index.html` when code is pushed to the `main` branch
2. **Triggers the test repository** via `repository_dispatch` using the GitHub API, passing metadata about the workflow run

### Test Repository

The [test-simple-html-page](https://github.com/sklarow/test-simple-html-page) repository listens for `repository_dispatch` events and:

1. **Receives the trigger** with metadata about this repository and workflow run
2. **Downloads the artifact** from this repository using the GitHub API
3. **Runs Playwright tests** against the downloaded HTML file
4. **Uploads test results** as artifacts

## Workflow Architecture

```
┌─────────────────────────────────┐
│  simple-html-page repository   │
│                                 │
│  1. Push to main                │
│  2. Upload artifact (index.html)│
│  3. Trigger repository_dispatch  │
└──────────────┬──────────────────┘
               │
               │ HTTP POST to GitHub API
               │
               ▼
┌─────────────────────────────────┐
│ test-simple-html-page repository│
│                                 │
│  1. Receive repository_dispatch │
│  2. Download artifact via API   │
│  3. Run Playwright tests        │
│  4. Upload test results         │
└─────────────────────────────────┘
```

## Authentication & Tokens

### Why Tokens Are Required

GitHub Actions workflows need authentication to:

1. **Trigger cross-repository workflows**: The `GITHUB_TOKEN` provided automatically in workflows has limited permissions and **cannot trigger workflows in other repositories**. A Personal Access Token (PAT) with appropriate scopes is required.

### Required Secrets

#### In This Repository (`simple-html-page`)

You need to create a secret named `PAT_FOR_TRIGGER`:

- **Type**: Personal Access Token (PAT) - Fine-grained personal access tokens
- **Required scope**: `Contents`
- **Usage**: Used to trigger the `repository_dispatch` event in the test repository

### Creating a Personal Access Token (PAT)

1. Go to GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Click "Generate new token"
3. Give it a descriptive name (e.g., "Cross-repo automation"), expiration and the repository access
4. Select the `Contents` permission
5. Generate and copy the token
6. Add it as a secret in your repository settings (Settings → Secrets and variables → Actions)

⚠️ **Important**: Store tokens securely and never commit them to your repository. If a token is compromised, revoke it immediately and generate a new one.

## Project Structure

```
.
├── index.html              # Simple HTML page
├── .github/
│   └── workflows/
│       └── build.yml       # Build and trigger workflow
└── README.md               # This file
```

## Workflow Details

The workflow (`build.yml`) performs the following steps:

1. **Checkout code**: Retrieves the repository code
2. **Upload artifact**: Uploads `index.html` as an artifact named `html-page`
3. **Trigger automation**: Sends a `repository_dispatch` event to the test repository with:
   - Event type: `trigger-e2e-tests`
   - Client payload containing:
     - `run_id`: The GitHub Actions run ID
     - `repo`: The repository name

## Resources

- [GitHub Actions: repository_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch)
- [GitHub API: Create a repository dispatch event](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
- [GitHub Actions: Upload artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

## License

ISC
