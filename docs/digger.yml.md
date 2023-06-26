# digger.yml

You can configure Digger by dropping a `digger.yml` file at the root level of your repo

```yaml
projects:
- name: my-first-app
  dir: app-one
- name: my-second-app
  dir: app-two
auto_merge: true
collect_usage_data: true
```



## Example using all keys

```yaml
collect_usage_data: true
auto_merge: false
projects:
- name: prod
  dir: prod
  workspace: default
  terragrunt: false
  workflow: prod
  include_patterns: ["../modules/**"]
  exclude_patterns: []
- name: staging
  dir: staging
  workflow: staging
  include_patterns: ["../modules/**"]
  exclude_patterns: []
generate_projects: 
  include: "../projects/**"
  exclude: "../.terraform/**"
workflows:
  staging:
    env_vars:
      state:
      - name: TF_LOG
        value: trace
      commands:
      - name: TF_LOG
        value: trace
    plan:
      steps:
      - init:
        extra_args: ["backend-config=../backend.hcl"]
      - run: "echo hello world"
      - plan
    apply:
      steps:
      - run: "echo hello world"
        shell: zsh
      - init
      - apply:
        extra_args: ["-compact-warnings"]
    workflow_configuration:
      on_pull_request_pushed: ["digger plan"]
      on_pull_request_closed: ["digger unlock"]
      on_commit_to_default: ["digger apply"]
  prod:
    env_vars:
      state:
      - name: AWS_ACCESS_KEY_ID
        value_from: PROD_BACKEND_TF_ACCESS_KEY_ID
      - name: AWS_SECRET_ACCESS_KEY
        value_from: PROD_BACKEND_TF_SECRET__ACCESS_KEY
      commands:
      - name: AWS_ACCESS_KEY_ID
        value_from: PROD_TF_ACCESS_KEY_ID
      - name: AWS_SECRET_ACCESS_KEY
        value_from: PROD_TF_SECRET_ACCESS_KEY
    plan:
      steps:
      - run: "checkov -d ."
      - init
      - plan
    apply:
      steps:
      - run: "terraform fmt -check -diff -recursive"
        shell: zsh
      - init
      - apply
    workflow_configuration:
      on_pull_request_pushed: ["digger plan"]
      on_pull_request_closed: ["digger unlock"]
      on_commit_to_default: ["digger unlock"]

```

## Reference

### Top-level

| Key                  | Type                                               | Default | Required | Description                                            | Notes |
| -------------------- | -------------------------------------------------- | ------- | -------- | ------------------------------------------------------ | ----- |
| collect\_usage\_data | boolean                                            | true    | no       | allows collecting anonymised usage and debugging data  |       |
| auto\_merge          | boolean                                            | false   | no       | automatically merge pull requests when all checks pass |       |
| projects             | array of [Projects](digger.yml.md#project)         | \[]     | no       | list of projects to manage                             |       |
| generate\_projects   | [GenerateProjects](digger.yml.md#generateprojects) | {}      | no       | generate projects from a directory structure           |       |
| workflows            | map of [Workflows](digger.yml.md#workflows)        | {}      | no       | workflows and configurations to run on events          |       |

### Project

| Key               | Type             | Default | Required | Description                                                        | Notes                                                                                                     |
| ----------------- | ---------------- | ------- | -------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| name              | string           |         | yes      | name of the project                                                | must be unique                                                                                            |
| dir               | string           |         | yes      | directory containing the project                                   |                                                                                                           |
| workspace         | string           | default | no       | terraform workspace to use                                         |                                                                                                           |
| terragrunt        | boolean          | false   | no       | whether to use terragrunt                                          |                                                                                                           |
| workflow          | string           | default | no       | workflow to use                                                    | default workflow will be created for you described in workflow section                                    |
| include\_patterns | array of strings | \[]     | no       | list of directory glob patterns to include                         | for example shared terraform modules folder                                                               |
| exclude\_patterns | array of strings | \[]     | no       | list of directory glob patterns to exclude                         | for example .terraform folder                                                                             |
| depends\_on       | array of strings | \[]     | no       | list of project names that need to be completed before the project | it doesn't force terraform run, but affects the order of commands for projects modified in the current PR |

### GenerateProjects

| Key     | Type   | Default | Required | Description                         | Notes |
| ------- | ------ | ------- | -------- | ----------------------------------- | ----- |
| include | string |         | no       | glob pattern to include directories |       |
| exclude | string |         | no       | glob pattern to exclude directories |       |

### Workflows

| Key                     | Type                                                         | Default | Required | Description                                | Notes |
| ----------------------- | ------------------------------------------------------------ | ------- | -------- | ------------------------------------------ | ----- |
| env\_vars               | [EnvVars](digger.yml.md#envvars)                             | {}      | no       | environment variables to set for per stage |       |
| plan                    | [Plan](digger.yml.md#plan)                                   | {}      | no       | plan stage configuration                   |       |
| apply                   | [Apply](digger.yml.md#apply)                                 | {}      | no       | apply stage configuration                  |       |
| workflow\_configuration | [WorkflowConfiguration](digger.yml.md#workflowconfiguration) | {}      | no       | describes how to react to CI events        |       |

### EnvVars

| Key      | Type                                    | Default | Required | Description                                               | Notes                                                                     |
| -------- | --------------------------------------- | ------- | -------- | --------------------------------------------------------- | ------------------------------------------------------------------------- |
| state    | array of [EnvVar](digger.yml.md#envvar) | \[]     | no       | environment variables to set for terraform init stage     | can be use to set different credentials for remote backend for example    |
| commands | array of [EnvVar](digger.yml.md#envvar) | \[]     | no       | environment variables to set for other terraform commands | can be use to set different credentials for actual managed infrastructure |

### EnvVar

| Key         | Type   | Default | Required | Description                                                  | Notes                                                                                                                                                                                                                                                                                                                    |
| ----------- | ------ | ------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| name        | string |         | yes      | name of the environment variable                             |                                                                                                                                                                                                                                                                                                                          |
| value\_from | string |         | yes      | name of the other environment variable to get the value from | this can be used for secrets. For example you set a secret from some secret manager (e.g. github secrets) as environment variable and the remap it to another variable. E.g. setting DEV\_TF\_ACCESS\_KEY as a secret in github action, but then remap it into AWS\_ACCESS\_KEY during terraform apply command execution |
| value       | string |         | yes      | value of the environment variable                            | this value will have a preference over value\_from field if both are set                                                                                                                                                                                                                                                 |

### Plan

| Key   | Type                                | Default | Required | Description                            | Notes |
| ----- | ----------------------------------- | ------- | -------- | -------------------------------------- | ----- |
| steps | array of [Step](digger.yml.md#step) | \[]     | no       | list of steps to run during plan stage |       |

### Apply

| Key   | Type                                | Default | Required | Description                             | Notes |
| ----- | ----------------------------------- | ------- | -------- | --------------------------------------- | ----- |
| steps | array of [Step](digger.yml.md#step) | \[]     | no       | list of steps to run during apply stage |       |

### WorkflowConfiguration

| Key                       | Type                                                                   | Default | Required | Description                                                   | Notes |
| ------------------------- | ---------------------------------------------------------------------- | ------- | -------- | ------------------------------------------------------------- | ----- |
| on\_pull\_request\_pushed | array of enums\[digger plan, digger apply, digger lock, digger unlock] | \[]     | no       | list of stages to run when pull request is pushed             |       |
| on\_pull\_request\_closed | array of enums\[digger plan, digger apply, digger lock, digger unlock] | \[]     | no       | list of stages to run when pull request is closed             |       |
| on\_commit\_to\_default   | array of enums\[digger plan, digger apply, digger lock, digger unlock] | \[]     | no       | list of stages to run when commit is pushed to default branch |       |

### Step

| Key   | Type                                                    | Default | Required | Description          | Notes                                              |
| ----- | ------------------------------------------------------- | ------- | -------- | -------------------- | -------------------------------------------------- |
| init  | [Init](digger.yml.md#init-apply-plan-as-object)/string  | {}/""   | no       | terraform init step  | if missing from array of steps, it will be skipped |
| plan  | [Plan](digger.yml.md#init-apply-plan-as-object)/string  | {}/""   | no       | terraform plan step  | if missing from array of steps, it will be skipped |
| apply | [Apply](digger.yml.md#init-apply-plan-as-object)/string | {}/""   | no       | terraform apply step | if missing from array of steps, it will be skipped |
| run   | [Run](digger.yml.md#run-as-object)/string               | {}/""   | no       | shell command to run | if missing from array of steps, it will be skipped |

### Init/Apply/Plan as object

| Key         | Type             | Default | Required | Description                                          | Notes |
| ----------- | ---------------- | ------- | -------- | ---------------------------------------------------- | ----- |
| extra\_args | array of strings | \[]     | no       | extra arguments to pass to terraform init/plan/apply |       |

### Run as object

| Key   | Type   | Default | Required | Description                     | Notes         |
| ----- | ------ | ------- | -------- | ------------------------------- | ------------- |
| shell | string |         | yes      | shell to use to run the command | zsh/bash etc. |

### Default workflow

Default workflow will be created for you if you don't specify any workflows in the configuration. It will have the following configuration:

```yaml
workflow_configuration:
  on_pull_request_pushed: [digger plan]
  on_pull_request_closed: [digger unlock]
  on_commit_to_default: [digger unlock]
```

## Workflow configuration explanation

Workflow configuration describes how to react to CI events. It has 3 sections:

* on\_pull\_request\_pushed - describes what to do when pull request is created or updated
* on\_pull\_request\_closed - describes what to do when pull request is closed
* on\_commit\_to\_default - describes what to do when pull request is merged into default branch



## Projects

A project in Digger corresponds to a directory containing Terraform code. Projects  are treated as standalone independent entities with their own locks. Digger will not prevent you from running plan and apply in different projects simultaneously.

You can run plan / apply in a specified project by using the -p option in Github PR comment:

```
digger apply -p my-second-app
```

## Include / exclude patterns

{% hint style="info" %}
This docs page needs improvement. Please consider contributing to [docs](https://github.com/diggerhq/docs). Here is the [relevant PR](https://github.com/diggerhq/digger/pull/301) implementing this feature
{% endhint %}

You can specify wildcard and glob patterns in digger.yml to include multiple directories into a project. A common use case for this is if you have multiple environment folders and they import from a common `modules` directory:\


```
development/
  main.tf
production/
  main.tf
modules/
  shared_moduleA/
  dev_only_module/
```

If you wanted to trigger plans for all `modules/` folder in both dev and prod projects you would include them in the `include_patterns`  key.  Similarly you put anything which you want to ignore in the `exclude_patterns` key ( exclude takes precedence over includes).

```
projects:
  - name: dev
    dir: ./development
    include_patterns: ["./modules/**"]
    workflow: default_workflow
  - name: prod
    dir: ./production
    include_patterns: ["./modules/**"]
    exclude_patterns: ["./modules/dev_only_module/**"]
```

