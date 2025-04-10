## Dynamically include a gitconfig based on remote

```
[includeIf "hasconfig:remote.*.url:git@github.com:*/**"]
    path = /Users/giorgiovilardo/.gitconfig-github
```