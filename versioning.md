# Versioning

## Specifying version

To pin a specific release of Digger, you can use `@vX.Y.Z` tag in your workflow file:

```
- name: digger
  uses: diggerhq/digger@v0.1.10
  env:
    ...
```

## Latest stable

To always run the latest release, use `@main` tag in your workflow file:

```
- name: digger
  uses: diggerhq/digger@main
  env:
    ...
```
