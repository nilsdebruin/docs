# Using Checkov

You can configure Digger to run Checkov policy-as-code as an additional step:

```
projects:
  - name: project_a_d
    dir: ./project_a/development
    workflow: project_a

workflows:
  project_a:
    plan:
      steps:
        - init
        - plan
        - run: checkov -d . --framework terraform
```
