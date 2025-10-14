# GitHub Actions 

GitHub Actions is tool, integrated into GitHub, which allows you to automate workflows.

It works by monitoring for defined events in or to your repository, and triggers a workflow in response. This makes GitHub Actions particularly useful for building a CI/CD pipeline.

To create an action first create a hidden folder in your repo, a sub-folder called workflows, and in there a deploy.yaml file.

[image]

In our case you should have your website files and resources, with the correct structure, already stored in your repository. You also have a local copy of our repository that we can continue working on.

When we save the changes to our local repository it is now out of sync with the remote repository, so we need to commit our changes.

>We'll commit directly to the main branch, but in the real-world you may have several branches until your change reaches production.

When a new commit is made against the main branch I want to initiate a workflow which will synchronize the new changes with the files in my storage bucket, which is hosting my live website.

If I was manually synchronising local files with a bucket in AWS I can use the command `aws s3 sync . s3://[bucket name]/`, so I really just need to run that command, but automatically from the repository, not from my computer.

|Command|Description|
|---|---|
|`aws s3 sync . s3://[bucket name]/`|syncronises the contents of the current location `.` with your remote bucket|

What else do we need to consider?

We haven’t got time to learn a whole new language, so here’s the code and we can just start dissecting it.

```yaml
name: Sync with S3

on:
  push:
    branches:
      - main

jobs:
  sync:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Install AWS CLI
        run: |
          sudo apt-get update
          sudo apt-get install -y awscli

      - name: Sync with S3
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: eu-west-2  # Replace with your AWS region
        run: |
          aws s3 sync . s3://[bucket name]/
```

## Breakdown

### Workflow Name

```yaml
name: Sync with S3Show more lines
```

This names the workflow "Sync with S3", which will appear in the GitHub Actions tab.

### Trigger

```YAML
on:
  push:
    branches:
      - main
```

This workflow runs automatically whenever code is pushed to the main branch.

### Job Definition

```YAML
jobs:  
  sync:
    runs-on: ubuntu-latest
```

- Defines a job called `sync`.
- It runs on the `latest` version of `Ubuntu` provided by GitHub-hosted runners.

### Checkout Repository

```YAML
- name: Checkout Repository
  uses: actions/checkout@v2
```

Uses the official GitHub Action to clone the repository into the runner so it can access the files.

### Install AWS CLI

```YAML
- name: Install AWS CLI
  run: |
    sudo apt-get update
    sudo apt-get install -y awscli
```

Updates the package list and installs the AWS CLI tool, which is needed to interact with AWS services like S3.

### Sync with S3

```YAML
- name: Sync with S3
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_DEFAULT_REGION: eu-west-2  
  run: |
    aws s3 sync . s3://[bucket name]/
```

- `Environment variables` are set using `GitHub Secrets` for secure access to AWS credentials.
- `AWS_DEFAULT_REGION` is set to `eu-west-2` (London).
- The `aws s3 sync` command uploads the contents of the repository to the specified S3 bucket.

---

The code is written in `YAML`, which is similar to `JSON` but with simpler syntax.

Different levels of Indentation are shown by a double-space, and you don’t need to open and close curly braces everywhere.

It’s still just KVPs though!

When GitHub Actions tries to complete the necessary steps, it will be prompted for authentication.

See below for how to do this in each of the main public cloud providers:

|Cloud|Method|
|---|---|
|AWS|Create a `user account` and `access keys` with the appropriate permissions|
|GCP|Create a `Service Account` with an associated `encryption key`|
|Azure|Use an `Access Key` from your `Storage Account`|

These credentials should then be added as repository secrets, which can be referenced in your GitHub Actions template.
