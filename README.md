# Automated Fork Synchronization Using GitHub Actions

Synchronizing a fork with the original repository can be a repetitive task, especially if the original repository is being updated frequently. This `README` guide will walk you through how to set up an automated process using GitHub Actions to sync your fork daily at midnight.

With this setup, your fork will now automatically synchronize with the original repository every day at midnight. You can also manually trigger the sync using the `workflow_dispatch` event from the GitHub Actions interface. Additionally, by leveraging **GitHub Secrets**, you can securely store and use sensitive information in your workflows.

## Setup Instructions

Follow the steps below to set up the automated synchronization:

### 1. Clone forked repository

First, clone the forked repository in the local system.

```bash
git clone https://github.com/USERNAME/FORKED_REPOSITORY.git
cd FORKED_REPOSITORY/
```

### 2. Creating the Workflow Directory

Firstly, you need to create a directory that will contain your workflow file. This directory is typically `.github/workflows` in the root of your repository:

```bash
mkdir -p .github/workflows
```

### 3. Creating the Workflow File

Create a new file named `auto-fork-sync.yml`:

```bash
touch .github/workflows/auto-fork-sync.yml
```

### 4. Defining the Workflow

Paste the following code into your `auto-fork-sync.yml`:

```yaml
name: Sync Fork

on:
  schedule:
    - cron: '0 0 * * *' # This will run the action daily at midnight
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout using TOKEN
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.TOKEN }}

    - name: Configure Git
      run: |
        git config user.name "Your GitHub Username"
        git config user.email "your-email@example.com"

    - name: Sync with Upstream using UPSTREAM_PAT
      run: |
        git remote add upstream https://x-access-token:${{ secrets.UPSTREAM_PAT }}@github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
        git fetch upstream
        git merge upstream/main --allow-unrelated-histories # To deal with divergent histories
        git push origin main
```

### 5. About GitHub Secrets

GitHub Secrets is a feature provided by GitHub to allow you to store and manage sensitive information in your repositories. In the workflow above, you noticed the usage of `${{ secrets.TOKEN }}` and `${{ secrets.UPSTREAM_PAT }}`. These represent secure tokens that you don't want to expose directly in your workflow files.

- `TOKEN`: Represents your personal access token. This gives the action permission to push changes to your repository.
- `UPSTREAM_PAT`: Personal access token for accessing the original repository. This might be required if the original repo is private.

**To set up these secrets:**
1. Go to your forked repository on GitHub.
2. Click on the `Settings` tab.
3. In the left sidebar, click on `Secrets`.
4. Click the `New repository secret` button.
5. Create a secret named `TOKEN` and another named `UPSTREAM_PAT` with appropriate values.

### 6. Committing and Pushing

After setting up the workflow, save the changes and push them to your repository:

```bash
git add .github/workflows/auto-fork-sync.yml
git commit -m "Add auto-sync workflow"
git push origin main
```
