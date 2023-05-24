# Specifying version

## Use a pinned stable version of Digger

To pin a specific release of Digger, you can use `@vX.Y.Z` tag in your workflow file:

```
- name: digger
  uses: diggerhq/digger@v0.1.10
  env:
    ...
```

## Use latest commit from a branch

You can also run latest commit from a specific branch

{% hint style="info" %}
Only use this at your own risk in non-production scenarios. This can break things!
{% endhint %}

```
- name: digger
  uses: diggerhq/digger@yolo-lets-do-it
  env:
    ...
```
