# is-default
indicates whether or not the checked out branch is the default branch (main, master, etc.)

for usage, see the [action.yaml](https://github.com/exadrift/github-actions-agent/blob/main/is-default/action.yaml)

employ the conditional to determine if on the default branch (`main`/`master`/etc):

```
- if: ${{ steps.<step-name>.outputs.is-default == 'true' }}
```
