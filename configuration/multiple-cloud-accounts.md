# Multiple cloud accounts

You can configure Digger to use different accounts for storing locks and for the target infra to plan / apply. This can even be different cloud providers - eg locks in AWS while managing GCP.

{% hint style="info" %}
This feature is under active development - [pull request](https://github.com/diggerhq/digger/pull/209)\
It is not yet available in a stable release.\
To use it as-is, [configure Digger to run off a branch](https://docs.digger.dev/configuration/versioning)
{% endhint %}

Set variables in action like this

```
env:
   PROJECT_1_ACCESS_KEY: ${{secrets.PROJECT_1_ACCESS_KEY}}
   STATE_ACCESS_KEY: ${{secrets.PROJECT_1_ACCESS_KEY}}
```

Then configure variables mapping in digger.yml

```
projects:
- name: dev
  workflow: dev
workflows:
  dev:
    envs:
      state:
        - name: AWS_ACCESS_KEY_ID
          valueFrom: STATE_ACCESS_KEY
      commands:
        - name: AWS_ACCESS_KEY_ID
          valueFrom: PROJECT_1_ACCESS_KEY
```
