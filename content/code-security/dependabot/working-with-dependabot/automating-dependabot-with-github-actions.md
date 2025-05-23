---
title: Automating Dependabot with GitHub Actions
intro: 'Examples of how you can use {% data variables.product.prodname_actions %} to automate common {% data variables.product.prodname_dependabot %} related tasks.'
permissions: '{% data reusables.permissions.dependabot-various-tasks %}'
versions:
  fpt: '*'
  ghec: '*'
  ghes: '*'
type: how_to
topics:
  - Actions
  - Dependabot
  - Version updates
  - Security updates
  - Repositories
  - Dependencies
  - Pull requests
shortTitle: Use Dependabot with Actions
redirect_from:
  - /code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/automating-dependabot-with-github-actions
---

{% ifversion dependabot-on-actions-opt-in %}

>[!NOTE] This article explains how to automate {% data variables.product.prodname_dependabot %}-related tasks using {% data variables.product.prodname_actions %}. For more information about running {% data variables.product.prodname_dependabot_updates %} using {% data variables.product.prodname_actions %}, see [AUTOTITLE](/code-security/dependabot/working-with-dependabot/about-dependabot-on-github-actions-runners) instead.
{% endif %}

You can use {% data variables.product.prodname_actions %} to perform automated tasks when {% data variables.product.prodname_dependabot %} creates pull requests to update dependencies. You may find this useful if you want to:

* Ensure that {% data variables.product.prodname_dependabot %} pull requests (version updates and security updates) are created with the right data for your work processes, including labels, names, and reviewers.

* Trigger workflows to send  {% data variables.product.prodname_dependabot %} pull requests (version updates and security updates) into your review process or to merge automatically.

{% data reusables.dependabot.enterprise-enable-dependabot %}

## About {% data variables.product.prodname_dependabot %} and {% data variables.product.prodname_actions %}

> [!IMPORTANT]
> If {% data variables.product.prodname_dependabot %} is enabled for a repository, it will always run on {% data variables.product.prodname_actions %}, **bypassing both Actions policy checks and disablement at the repository or organization level**. This ensures that security and version update workflows always run when Dependabot is enabled.

{% data variables.product.prodname_dependabot %} creates pull requests to keep your dependencies up to date. You can use {% data variables.product.prodname_actions %} to perform automated tasks when these pull requests are created. For example, fetch additional artifacts, add labels, run tests, or otherwise modify the pull request.

{% data reusables.dependabot.working-with-actions-considerations %} For more information, see [AUTOTITLE](/code-security/dependabot/troubleshooting-dependabot/troubleshooting-dependabot-on-github-actions).

Here are several common scenarios for pull requests that can be automated using {% data variables.product.prodname_actions %}.

## Fetching metadata about a pull request

Most automation requires you to know information about the contents of the pull request: what the dependency name was, if it's a production dependency, and if it's a major, minor, or patch update. You can use an action to retrieve information about the dependencies being updated by a pull request generated by {% data variables.product.prodname_dependabot %}.

Example:

{% raw %}

```yaml copy
name: Dependabot fetch metadata
on: pull_request

permissions:
  pull-requests: write
  issues: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' && github.repository == 'owner/my_repo'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@d7267f607e9d3fb96fc2fbe83e0af444713e90b7
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      # The following properties are now available:
      #  - steps.metadata.outputs.dependency-names
      #  - steps.metadata.outputs.dependency-type
      #  - steps.metadata.outputs.update-type
```

{% endraw %}

For more information, see the [`dependabot/fetch-metadata`](https://github.com/dependabot/fetch-metadata) repository.

## Labeling a pull request

If you have other automation or triage workflows based on {% data variables.product.github %} labels, you can configure an action to assign labels based on the metadata provided.

Example that flags all production dependency updates with a label:

{% raw %}

```yaml copy
name: Dependabot auto-label
on: pull_request

permissions:
  pull-requests: write
  issues: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' && github.repository == 'owner/my_repo'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@d7267f607e9d3fb96fc2fbe83e0af444713e90b7
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Add a label for all production dependencies
        if: steps.metadata.outputs.dependency-type == 'direct:production'
        run: gh pr edit "$PR_URL" --add-label "production"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
```

{% endraw %}

## Automatically approving a pull request

You can automatically approve {% data variables.product.prodname_dependabot %} pull requests by using the {% data variables.product.prodname_cli %} in a workflow.

Example:

{% raw %}

```yaml copy
name: Dependabot auto-approve
on: pull_request

permissions:
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' && github.repository == 'owner/my_repo'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@d7267f607e9d3fb96fc2fbe83e0af444713e90b7
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Approve a PR
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

{% endraw %}

## Enabling automerge on a pull request

If you want to allow maintainers to mark certain pull requests for automerge, you can use {% data variables.product.prodname_dotcom %}'s automerge functionality. This enables the pull request to be merged when any tests and approvals required by the branch protection rules are successfully met.

For more information, see [AUTOTITLE](/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request) and [AUTOTITLE](/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule).

You can instead use {% data variables.product.prodname_actions %} and the {% data variables.product.prodname_cli %}. Here is an example that automerges all patch updates to `my-dependency`:

{% raw %}

```yaml copy
name: Dependabot auto-merge
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' && github.repository == 'owner/my_repo'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@d7267f607e9d3fb96fc2fbe83e0af444713e90b7
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Enable auto-merge for Dependabot PRs
        if: contains(steps.metadata.outputs.dependency-names, 'my-dependency') && steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --merge "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

{% endraw %}

> [!NOTE]
> If you use status checks to test pull requests, you should enable **Require status checks to pass before merging** for the target branch for {% data variables.product.prodname_dependabot %} pull requests. This branch protection rule ensures that pull requests are not merged unless **all the required status checks pass**. For more information, see [AUTOTITLE](/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule).

## {% data variables.product.prodname_dependabot %} and {% data variables.product.prodname_actions %} policies

Normally, whether a workflow can run in a repository depends on {% data variables.product.prodname_actions %} **policy checks** and whether {% data variables.product.prodname_actions %} is **enabled** at the organization or repository level. These controls can restrict workflows from running—especially when external actions are blocked or {% data variables.product.prodname_actions %} is disabled entirely.

However, when {% data variables.product.prodname_dependabot %} is enabled for a repository, its workflows will always run on {% data variables.product.prodname_actions %}, **bypassing both Actions policy checks and disablement**.

* {% data variables.product.prodname_dependabot %} workflows are not blocked by Actions disablement or enterprise policy restrictions.
* The actions referenced within these workflows are also allowed to run, even if external actions are disallowed.

{% ifversion dependabot-on-actions-opt-in %}
For more information, see [AUTOTITLE](/code-security/dependabot/working-with-dependabot/about-dependabot-on-github-actions-runners).
{% endif %}

## Investigating failed workflow runs

If your workflow run fails, check the following:

* You are running the workflow only when the correct actor triggers it.
* You are checking out the correct `ref` for your `pull_request`.
* Your secrets are available in {% data variables.product.prodname_dependabot %} secrets rather than as {% data variables.product.prodname_actions %} secrets.
* You have a `GITHUB_TOKEN` with the correct permissions.

For information on writing and debugging {% data variables.product.prodname_actions %}, see [AUTOTITLE](/actions/learn-github-actions).

For more tips to help resolve issues with workflows, see [AUTOTITLE](/code-security/dependabot/troubleshooting-dependabot/troubleshooting-dependabot-on-github-actions).
