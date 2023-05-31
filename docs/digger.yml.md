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

You can specify wildcard and glob patterns in digger.yml to include multiple directories into a project:

```
projects:
  - name: gke_shared_cluster_dev
    dir: ./gke_shared_cluster/development
    include_patterns: ["./modules/single_project/**"]
    workflow: default_workflow
  - name: gke_shared_cluster_non-prod
    dirs: ./gke_shared_cluster/non-production
    include_patterns: ["./modules/single_project/**"]
    workflow: default_workflow
  - name: gke_shared_cluster_prod
    dir: ./gke_shared_cluster/production
    include_patterns: ["./modules/single_project/**"]
```
