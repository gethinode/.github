# Dependabot auto-merge

Dependabot patch and minor updates can enable auto-merge after the workflow verifies the dependency metadata and the signed PR head. Major updates and unverified changes go to manual QA.

## Configure a calling repository

1. Create a dedicated GitHub App and install it on the repositories taking part in the rollout. Give it repository write access to **Contents**, **Pull requests**, and **Workflows**. Workflow access covers Dependabot updates that change workflow files. Keep the App subject to the same required checks as other merge actors.
2. Set the Actions variables `AUTOMERGE_APP_CLIENT_ID` and `AUTOMERGE_APP_SLUG` in the calling repository. The slug is the App’s URL identifier, without the `[bot]` suffix.
3. Store the App’s private key as the Actions secret `AUTOMERGE_APP_PRIVATE_KEY` in each calling repository. For the reusable workflow, pass that named secret or use `secrets: inherit`. The key can mint tokens for every repository where the App is installed, so keep the App installation and the secret limited to the participating repositories.
4. Create the `requires-manual-qa` label. Enable repository auto-merge and configure the required checks on the base branch.

The workflow uses `pull_request_target` on the maintained base branch. It mints a short-lived token for the calling repository and uses that token to enable auto-merge. The workflow token handles labels, reviews, and disarming. If the App credential is unavailable or its slug differs from the configured slug, the eligible update is held for manual review.

## Verify the rollout

- Check that an eligible patch arms under the configured App and merges after the required checks pass. Verify the push-triggered release workflow on the resulting merge commit.
- Check that a major update gets `requires-manual-qa` with auto-merge off.
- Check that an update moved to manual QA has any workflow-owned auto-merge removed. Keep the App slug configured while outstanding auto-merge requests may still belong to that App.

See [GitHub App installation token scoping](https://github.com/actions/create-github-app-token#repositories) and [GitHub workflow token behavior](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow#triggering-a-workflow-from-a-workflow).
