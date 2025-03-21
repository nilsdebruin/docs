# Github Actions and GCP

This is a step-by-step guide how to set up Digger in your Github repository with terraform code for GCP. If you'd rather fork a sample repository, or if are using Terragrunt or Workspaces - check out [other tutorials](https://diggerhq.gitbook.io/digger-docs/getting-started)

### 1. Create a GCP bucket for locks

* Go to Cloud Storage → Buckets; Click Create
* Name it something like “digger-lock-myproj” (you'll need the name later)
* Keep all the default settings

### 2. Create IAM service account in GCP

* Go to IAM → Service Accounts (in the side bar)
* Press “create service account” at the top
* Keep all the default settings

{% hint style="info" %}
Make sure your IAM service account has permission to write to the bucket
{% endhint %}

{% hint style="info" %}
Make sure your IAM service account has permissions to execute your Terraform
{% endhint %}

### 3. Add GCP keys to Github Actions Secrets

In your new GCP service account:

* go to Keys; click Add Key&#x20;
* Download json
* Copy contents of the json file

Thenn in Github

* Go to Settings → Secrets and Variables
* Click “New Repository Secret” button
* Add a secret named `GCP_CREDENTIALS` and paste contents of GCP key json there

### 4. Create digger.yml

In your repository, create `digger.yml` file with the following contents:

```yaml
projects:
- name: infra-prod
  dir: prod
```

### 5. Create a workflow file

In your repository, create a file at `.github/workflows/infra.yml`

```yaml
name: Terraform Digger

on:
  pull_request:
    branches: [ "main" ]
    types: [ closed, opened, synchronize, reopened ]
  issue_comment:
    types: [created]
    if: contains(github.event.comment.body, 'digger')
  workflow_dispatch:

jobs:
  digger-terraform:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Checkout Pull Request
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          PR_URL="${{ github.event.issue.pull_request.url }}"
          PR_NUM=${PR_URL##*/}
          echo "Checking out from PR #$PR_NUM based on URL: $PR_URL"
          hub pr checkout $PR_NUM
        if: github.event_name == 'issue_comment'
        
      - id: 'auth'
        uses: 'google-github-actions/auth@v1'
        with:
          credentials_json: '${{ secrets.GCP_CREDENTIALS }}'

      - name: 'Set up Cloud SDK'
        uses: 'google-github-actions/setup-gcloud@v1'

      - name: 'Use gcloud CLI'
        run: 'gcloud info'

      - name: digger
        uses: diggerhq/digger@main
        env:
          LOCK_PROVIDER: gcp
          GITHUB_CONTEXT: ${{ toJson(github) }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GOOGLE_STORAGE_BUCKET: **digger-lock-myproj**
```

{% hint style="info" %}
Rename storage bucket (last line) to your bucket name
{% endhint %}



### 6. Create a PR to verify that it works

Just make any change to Terraform - like add a blank line

An action should start. After some time you should see output of Terraform Plan added as a comment to your PR. Something like this:

```
Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      
 ... (further content omitted)
```
