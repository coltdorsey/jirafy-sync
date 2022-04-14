[![Unit Tests](https://github.com/onXmaps/jirafy-sync/actions/workflows/tests.yml/badge.svg)](https://github.com/onXmaps/jirafy-sync/actions/workflows/tests.yml)
# Jirafy Sync
Sync Jira tickets and version based on a given changelog (ideally generated by [Jirafy Changelog](https://github.com/marketplace/actions/jirafy-changelog))

## What does Jirafy Sync do?
This action requires a changelog as input. If the changelog has references to Jira tickets, it will parse the changelog for Jira projects and tickets. For each project, the latest repository release will be the Jira project version created. Each Jira ticket's `fixVersions` property will be updated to the corresponding Jira version created.

## Inputs
Jira credentials are required in order for Jirafy Sync to authenticate to the Jira rest api on your behalf.
- jiraUsername
  - Username of a jira user (preferably one designated for automation tasks)
- jiraToken
  - Corresponding [token](https://id.atlassian.com/manage-profile/security/api-tokens) of the jira user
- jiraHost
  - Jira organization host name (i.e jirafy-sync.atlassian.net)
- changelog
  - A generated changelog
    - **TIP:** Use [Jirafy Changelog](https://github.com/marketplace/actions/jirafy-changelog)
- jiraVersion
  - Github release version # (i.e v2.1.0)
    - **TIP:** Use the Jirafy Changelog output: ${{ steps.changelog.outputs.changelog }}

---

## Jirafy Sync in action
Creates Jira Version

<a href="https://github.com/onXmaps/jirafy-sync/releases/tag/v2.0.0"><img alt="Jirafy Sync Release v2.0.0" src="./jsync_release.png" width="800"></a>

<a href="https://github.com/onXmaps/jirafy-sync/releases/tag/v2.0.0"><img alt="Jirafy Sync Release Detail v2.0.0" src="./jsync_release_detail.png" width="800"></a>

Updates Jira ticket(s) fixVersion property

<a href="https://github.com/onXmaps/jirafy-sync/releases/tag/v2.0.0"><img alt="Jirafy Sync fixVersion v2.0.0" src="./jsync_fixVersion.png" width="800"></a>

---

## Recommended github workflow
- Jirafy Sync works best if [Jirafy Changelog](https://github.com/marketplace/actions/jirafy-changelog) github action generates the changelog. The changelog can be generated given there is already at least one github release in your repository.

- Next you'll need to generate a release. I recommend using [actions/actions/create-release@latest](https://github.com/actions/create-release).

- Finally you'll want to sync the github release with a jira version using [Jirafy Sync](https://github.com/onXmaps/jirafy-sync#jirafy-sync) action. The `${{ github.ref_name }}` value is the tag that triggers the recommended workflow and should set as the input of `jiraVersion` property. The action will create the corresponding Jira project versions that don't already exist, as well as update the fix version of each ticket that was found in the given changelog.

```yaml
name: Jirafy Sync
on:
  push:
    # Sequence of patterns matched against refs/tags
    tags:
      - 'v*' # Push events to matching v*, i.e. v1.0, v1.0.0

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      # To use this repository's private action, you must check out the repository
      - name: Checkout
        uses: actions/checkout@v2

      - name: Jirafy Changelog
        id: changelog
        uses: onXmaps/jirafy-changelog@v1.1.0
        with:
          jiraHost: 'coltdorsey.atlassian.net'
          myToken: ${{ secrets.GITHUB_TOKEN }}

      - name: Create Release
        id: create_release
        uses: actions/create-release@latest
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: ${{ steps.changelog.outputs.changelog }}
          draft: false
          prerelease: false

      - name: Jirafy Sync
        uses: onXmaps/jirafy-sync@v2.0.1
        with:
          changelog: ${{ steps.changelog.outputs.changelog }}
          jiraVersion: ${{ github.ref_name }}
          jiraHost: 'coltdorsey.atlassian.net'
          jiraUsername: ${{ secrets.JIRA_USERNAME }}
          jiraToken: ${{ secrets.JIRA_TOKEN }}
```