# Buildkite Agentic Pipeline Example: PR Fix

[![Build status](https://badge.buildkite.com/cc9ff459886b1cbbf10409eb0207a3879e03e66634701d8fc7.svg)](https://buildkite.com/cnunciato/buildkite-agentic-pr-fix-example)
[![Add to Buildkite](https://img.shields.io/badge/Add%20to%20Buildkite-14CC80)](https://buildkite.com/new)

A [Buildkite](https://buildkite.com) pipeline that uses [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) and Buildkite model providers to diagnose and fix broken PR builds.

## Setting up

### Step 1: Start with a repository to fix

1. Pick an existing GitHub repository with a Buildkite pipeline. This is the repo you'll fix with this pipeline.
1. Make an intentionally broken pull request on that repository.

### Step 2: Prepare your agentic pipeline

1. [Create a new GitHub repository](https://github.com/new?template_name=buildkite-agentic-pr-fix-example&template_owner=cnunciato) using this repository as a template.
1. [Create a new Buildkite pipeline](https://buildkite.com/organizations/~/pipelines/new) with your forked/templated repository with a Linux [hosted agents](https://buildkite.com/docs/pipelines/hosted-agents) cluster.
1. [Create a new pipeline trigger](https://buildkite.com/~/buildkite-agentic-pr-fix-example/settings/triggers/new?trigger_type=webhook) for the pipeline, then copy the generated webhook URL to your clipboard.
1. Navigate to the **Settings** > **Webhooks** page of your forked/templated repository.
1. Add a new GitHub webhook, paste the Buildkite webhook URL from your clipboard into the Payload URL field, choose **Select individual events**, choose **Pull requests** only, and save.
1. Create two new [Buildkite secrets](https://buildkite.com/docs/pipelines/security/secrets/buildkite-secrets) with the following names:
   1. `GITHUB_TOKEN`: A GitHub [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) with `repo` and `pull_request` read/write scopes.
   1. `API_TOKEN_BUILDKITE`: A Buildkite [API access token](https://buildkite.com/docs/apis/managing-api-tokens) with `read_builds`, `read_build_logs`, and `read_pipelines` permissions.

### Step 3: Trigger a fix!

1. In the pull request you opened in Step 1, create a new label, `fix-build` (this is configurable in `.buildkite/pipeline.yml`), and apply it to the pull request.
1. In a few moments, you should see a new comment appear on your PR acknowledging that the agent has picked up the task. Follow the link to the Buildkite dashboard to have a look.
1. Once the agent determines a fix, you should see a new PR submitted with a comment explaining what was done. Have a look at that PR, and if it looks good, approve and merge.

That's it! Your broken PR should now be fixed.

## Handling multiple repositories

A single agentic pipeline can handle builds originating from multiple GitHub repositories. Just create a new pipeline trigger and associated GitHub webhook for each one (as described in Step 2 above), and apply the `fix-build` label as needed.

## How it works

The pipeline listens for webhooks originating from GitHub pull-request events. When a `pull_request.labeled` event is detected, the pipeline runs the `handler` script, which adds a step to the running pipeline that uses Claude to diagnose and fix the failure (with the `agent.sh` script), [annotating](https://buildkite.com/docs/apis/rest-api/annotations) the build (with the `parser` script) as it goes. When the fix is complete, Claude commits the changes to a new branch, pushes the branch to the repository, and submits a new PR on the original, failed PR branch for review.
