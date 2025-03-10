# Authenticating with OIDC on AWS

{% hint style="info" %}
support for AWS OIDC assume role paramter will be in v0.1.23
{% endhint %}

In order to set up OIDC simply swap the AWS Keys with assume role ARN and you are good to go. Here is an example, don't forget to replace the line starting in \*\* with your own ARN for the account.

```
name: CI

on:
  pull_request:
    branches: [ "main" ]
    types: [ closed, opened, synchronize, reopened ]
  issue_comment:
    types: [created]
    if: contains(github.event.comment.body, 'digger')
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: digger run
        uses: diggerhq/digger@v0.1.23
        with:
          setup-aws: true
        **aws-role-to-assume: arn:aws:sts::{AccountID}:assumed-role/{RoleName}/{FunctionName}**
          aws-region: us-east-1
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
