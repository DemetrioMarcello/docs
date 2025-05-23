---
title: Disabling or limiting GitHub Actions for your organization
intro: 'You can enable, disable, and limit GitHub Actions for an organization.'
redirect_from:
  - /github/setting-up-and-managing-organizations-and-teams/disabling-or-limiting-github-actions-for-your-organization
versions:
  fpt: '*'
  ghes: '*'
  ghec: '*'
permissions: Organization owners{% ifversion custom-org-roles %} and users with the "Manage organization Actions policies" and "Manage runners and runner groups" fine-grained permissions{% endif %} can enable, disable, and limit {% data variables.product.prodname_actions %} for an organization. {% ifversion custom-org-roles %}<br><br>For more information, see [AUTOTITLE](/organizations/managing-peoples-access-to-your-organization-with-roles/about-custom-organization-roles).{% endif %}
topics:
  - Organizations
  - Teams
shortTitle: Disable or limit actions
---

{% data reusables.actions.enterprise-github-hosted-runners %}

## About {% data variables.product.prodname_actions %} permissions for your organization

{% data reusables.actions.disabling-github-actions %} For more information about {% data variables.product.prodname_actions %}, see [AUTOTITLE](/actions/learn-github-actions).

You can enable {% data variables.product.prodname_actions %} for all repositories in your organization. {% data reusables.actions.enabled-actions-description %} You can disable {% data variables.product.prodname_actions %} for all repositories in your organization. {% data reusables.actions.disabled-actions-description %}

Alternatively, you can enable {% data variables.product.prodname_actions %} for all repositories in your organization but limit the actions {% ifversion actions-workflow-policy %}and reusable workflows{% endif %} a workflow can run.

## Managing {% data variables.product.prodname_actions %} permissions for your organization

You can choose to disable {% data variables.product.prodname_actions %} for all repositories in your organization, or only allow specific repositories. You can also limit the use of public actions{% ifversion actions-workflow-policy %} and reusable workflows{% endif %}, so that people can only use local actions {% ifversion actions-workflow-policy %}and reusable workflows{% endif %} that exist in your {% ifversion ghec or ghes %}enterprise{% else %}organization{% endif %}.

> [!NOTE]
> You might not be able to manage these settings if your organization is managed by an enterprise that has overriding policy. For more information, see [AUTOTITLE](/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-github-actions-in-your-enterprise).

{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
1. Under "Policies", select an option.

   {% indented_data_reference reusables.actions.actions-use-policy-settings spaces=3 %}
1. Click **Save**.

{% data reusables.actions.allow-specific-actions-intro %}

{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
1. Under "Policies", select {% data reusables.actions.policy-label-for-select-actions-workflows %} and add your required actions{% ifversion actions-workflow-policy %} and reusable workflows{% endif %} to the list.
1. Click **Save**.

## Limiting the use of self-hosted runners

{% data reusables.actions.disable-selfhosted-runners-overview %}

{% ifversion ghec or ghes %}

> [!NOTE]
> If your organization belongs to an enterprise, creation of self-hosted runners at the repository level may have been disabled as an enterprise-wide setting. If this has been done, you cannot enable repository-level self-hosted runners in your organization settings. For more information, see [AUTOTITLE](/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-github-actions-in-your-enterprise).

{% endif %}

If a repository already has self-hosted runners when you disable their use, these will be listed with the status "Disabled" and they will not be assigned any new workflow jobs.

![Screenshot of the "Runners" list showing a self-hosted runner with the status "Disabled."](/assets/images/help/actions/actions-runners-disabled.png)

{% data reusables.actions.disable-selfhosted-runners-note %}

{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
1. Under "Runners," use the dropdown menu to choose your preferred setting:
   * **All repositories** - self-hosted runners can be used for any repository in your organization.
   * **Selected repositories** - self-hosted runners can only be used for the repositories you select.
   * **Disabled** - self-hosted runners cannot be created at the repository level.
1. If you choose **Selected repositories**:
   1. Click {% octicon "gear" aria-label="Select repositories" %}.
   1. Select the check boxes for the repositories for which you want to allow self-hosted runners.
   1. Click **Select repositories**.

{% ifversion fpt or ghec %}

## Configuring required approval for workflows from public forks

{% data reusables.actions.workflow-run-approve-public-fork %}

You can configure this behavior for an organization using the procedure below. Modifying this setting overrides the configuration set at the enterprise level.

{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
{% data reusables.actions.workflows-from-public-fork-setting %}

{% data reusables.actions.workflow-run-approve-link %}
{% endif %}

## Enabling workflows for private repository forks

{% data reusables.actions.private-repository-forks-overview %}

{% ifversion ghec or ghes %}If a policy is disabled for an enterprise, it cannot be enabled for organizations.{% endif %} If a policy is disabled for an organization, it cannot be enabled for repositories. If an organization enables a policy, the policy can be disabled for individual repositories.

{% data reusables.actions.private-repository-forks-options %}

### Configuring the private fork policy for an organization

{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
{% data reusables.actions.private-repository-forks-configure %}

## Setting the permissions of the `GITHUB_TOKEN` for your organization

{% data reusables.actions.workflow-permissions-intro %}

You can set the default permissions for the `GITHUB_TOKEN` in the settings for your organization or your repositories. If you select a restrictive option as the default in your organization settings, the same option is selected in the settings for repositories within your organization, and the permissive option is disabled. If your organization belongs to a {% data variables.product.prodname_enterprise %} account and a more restrictive default has been selected in the enterprise settings, you won't be able to select the more permissive default in your organization settings.

{% data reusables.actions.workflow-permissions-modifying %}

### Configuring the default `GITHUB_TOKEN` permissions

By default, when you create a new organization,{% ifversion ghec or ghes %} the setting is inherited from what is configured in the enterprise settings.{% else %} `GITHUB_TOKEN` only has read access for the `contents` and `packages` scopes.{% endif %}

{% data reusables.profile.access_profile %}
{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
{% data reusables.actions.workflows.github-token-access %}
1. Click **Save** to apply the settings.

### Preventing {% data variables.product.prodname_actions %} from creating or approving pull requests

{% data reusables.actions.workflow-pr-approval-permissions-intro %}

By default, when you create a new organization, workflows are not allowed to create or approve pull requests.

{% data reusables.profile.access_profile %}
{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
1. Under "Workflow permissions", use the **Allow GitHub Actions to create and approve pull requests** setting to configure whether `GITHUB_TOKEN` can create and approve pull requests.
1. Click **Save** to apply the settings.

## Managing {% data variables.product.prodname_actions %} cache storage for your organization

Organization administrators can view {% ifversion ghes %}and manage {% endif %}{% data variables.product.prodname_actions %} cache storage for all repositories in the organization.

### Viewing {% data variables.product.prodname_actions %} cache storage by repository

For each repository in your organization, you can see how much cache storage a repository is using, the number of active caches, and if a repository is near the total cache size limit. For more information about the cache usage and eviction process, see [AUTOTITLE](/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy).

{% data reusables.profile.access_profile %}
{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
1. In the left sidebar, click **{% octicon "play" aria-hidden="true" aria-label="play" %} Actions**, then click **Caches**.
1. Review the list of repositories for information about their {% data variables.product.prodname_actions %} caches. You can click on a repository name to see more detail about the repository's caches.

{% ifversion ghes %}

### Configuring {% data variables.product.prodname_actions %} cache storage for your organization

{% data reusables.actions.cache-default-size %}

You can configure the size limit for {% data variables.product.prodname_actions %} caches that will apply to each repository in your organization. The cache size limit for an organization cannot exceed the cache size limit set in the enterprise policy. Repository admins will be able to set a smaller limit in their repositories.

{% data reusables.profile.access_profile %}
{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.settings-sidebar-actions-general %}
{% data reusables.actions.change-cache-size-limit %}

{% endif %}
