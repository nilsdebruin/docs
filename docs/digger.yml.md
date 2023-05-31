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
generate_projects:
  include: projects/dev/*
  exclude: projects/dev/docs
```

**auto\_merge**: weather to merge pull requests automatically or not, default is false

**collect\_usage\_data**: weather to collect usage data or not, default is false

**generate\_projects**: generate projects based on provided include/exclude patterns, optional



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

