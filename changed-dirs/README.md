# changed-dirs
detects and reports directories containing changes.  these can then be fed directly into a matrix, allowing a single repository to host multiple independent projects

# configuration
configuring this action requires setting up a matrix which contains the result of this action, so that a subsequent action can then operate in parallel against said matrix.

```
name: perform operation against directories containing changes
permissions:
  # the write permission is necessary to create the base tag, which reflects the repo state
  # prior to any state finalization, even after merge, the pre-merge commit point needs to be known
  # in order to see which items need post-merge steps based on prior changes
  contents: write
  checks: read

on:
  push:
    branches:
    - "**"

jobs:
  create-change-matrix:
    runs-on: ubuntu-24.04
    name: get directories containing changes
    outputs:
      # expose the outputs of the changes step, as job outputs
      directories: ${{ steps.get-changed-dirs.outputs.directories }}
    steps:
    - uses: actions/checkout@v7
      with:
        fetch-depth: 0
    - id: get-changed-dirs
      uses: exadrift/github-actions/changed-dirs@changed-dirs-v0

  do-matrix-action:
    # require the matrix above
    needs: create-change-matrix
    runs-on: ubuntu-24.04
    # if the list is empty, don't do anything, otherwise github will error
    if: needs.create-change-matrix.outputs.directories != '[]'
    strategy:
      matrix:
        item: ${{ fromJson(needs.create-change-matrix.outputs.directories) }}
    defaults:
      run:
        # change the working directory of subsequent actions, to the path of matrix item
        working-directory: ${{ matrix.item.path }}
    steps:
    - uses: actions/checkout@v7
      with:
        fetch-depth: 0
    - ...

  update-base-tag:
    runs-on: ubuntu-24.04
    name: update base tag
    needs:
    - do-matrix-action
    steps:
    - name: update base tag
      uses: exadrift/github-actions/changed-dirs@changed-dirs-v0
      with:
        # causes the base tag to be written, essentially finalizing the processed change set
        update-base-tag: true
```

for usage, see the [action.yaml](https://github.com/exadrift/github-actions/blob/main/changed-dirs/action.yaml)
