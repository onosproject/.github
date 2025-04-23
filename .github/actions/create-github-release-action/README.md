<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# ⬆️ Create Github Release Action

This action creates a new release in the calling repo using the provided version.

## create-github-release-action

## Usage Example

<!-- markdownlint-disable MD046 -->

```yaml
steps:
  - name: Create Github release
    uses: onos-project/.github/.github/actions/create-github-release-action@main
    with:
      version: ${{ steps.version-change.outputs.version }}
    env:
      GH_TOKEN: ${{ secrets.GH_ONOS_PAT }}
```

<!-- markdownlint-enable MD046 -->

## Inputs

<!-- markdownlint-disable MD013 -->

| Variable Name | Required | Description  |
| ------------- | -------- | ------------ |
| VERSION       | True     | Version to release on Github |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Variable Name | Description   |
| ------------- | ------------- |
| None          |               |

<!-- markdownlint-enable MD013 -->

## Notes

This uses the built-in `gh` CLI. A GH_TOKEN with proper permissions should be
included in the env.
