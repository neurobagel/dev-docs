---
title: Neurobagel PATs

---

## Existing PATs

Below is a list of PATs which are used in our active workflows and should be renewed immediately when expired.

Secret names and permissions:

### Neurobagel organization

- **NB_PROJECT_PAT**:
  - Creator: @surchs
  - Purpose: we use this PAT to add new issues to the board
  - Org permission: Read and Write access to organization projects
  - Repo permission: Read access to metadata
  - Repo permission: Read and Write access to code, issues, and pull requests
- **NB_PR_PAT**:
  - Creator: @surchs
  - Purpose: We use this PAT to discover PRs opened by bots and to move them to the board
  - Org permission: -
  - Repo permission: Read access to metadata
  - Repo permission: Read and Write access to actions, code, issues, pull requests, and workflows
- **NB_PAT_RELEASE**
  - Creator: @surchs
  - Purpose: We use this PAT to make automatic releases with [intuit/auto](https://github.com/intuit/auto)
  - Org permission: -
  - Repo permission: Read access to metadata
  - Repo permission: Read and Write access to actions, code, commit statuses, deployments, issues, pull requests, repository hooks, and workflows
- **NB_PAT_RELEASE_PROTECTED** 
  - Creator: @alyssadai
  - Purpose: We use this PAT to be able to automatically release on protected branches using the [protected-branch plugin](https://www.npmjs.com/package/@auto-it/protected-branch) for [intuit/auto](https://github.com/intuit/auto). This requires a different PAT than `NB_PAT_RELEASE` since a CHANGELOG PR is auto-created as part of the process, but a PR author cannot approve their own PR.
  - Org permission: -
  - Repo permission: Read access to metadata
  - Repo permission: Read and Write access to actions, code, commit statuses, deployments, issues, pull requests, repository hooks, and workflows
- **LAB_PAT**
  - Creator: @surchs
  - Purpose: We use this PAT to synchronize labels across repositories
  - Org permissions: Read and Write access to organization actions variables and organization projects
  - Repo permissions: Read access to metadata
  - Repo permissions: Read and Write access to actions, issues, and pull requests
- **GH_PAT** in the https://github.com/neurobagel/upptime/ repo
  - Creator: @surchs
  - Purpose: We use this PAT to run the upptime workflow
  - Org permissions: -
  - Repo permissions: Read access to metadata
  - Repo permissions: Read and Write access to actions, code, issues, and workflows

### OpenNeuroDatasets-JSONLD organization
#### .github repo
- **NB_OPENNEURO_ANNOTATIONS_RW**
  - Creator: @alyssadai
  - Purpose: Used to push updated JSONLDs to a branch in the `neurobagel/openneuro-annotations` repo
  - Org permissions: -
  - Repo permissions: Read access to metadata
  - Repo permissions: Read and Write access to actions, code, and workflows
- **ON_WF_PAT**
  - Creator: @surchs
  - Purpose: Used to create/update forks of OpenNeuro and keep OpenNeuroDatasets-JSONLD in sync with OpenNeuroDatasets
  - Permissions: Read and Write access to actions, administration, code, commit statuses, deployments, issues, pull requests, and workflows

## Updating the Personal Access Tokens
Our GH action workflows rely on https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens .
Currently they are created by individual admin users for the @neurobagel org.
However, these PATs expire - e.g. after 60 days.

To keep the lights on for our workflows, two things have to be done when PATs expire:

1. **Renew the PAT**. If you do this while the old PAT is still valid, you
can just use the "renew" button in the Github interface. You will also get a link
to this "renew" button in the email from GitHub that reminds you the PAT will expire
2. **Copy the new PAT into the organization secrets**. When you renew a PAT you 
also change the PAT secret. Make sure to copy that secret before closing the tab,
because you will only be able to see it once. When you have copied it, make
sure to update the organization secrets here: https://github.com/neurobagel/documentation/settings/secrets/actions
with the new value. Make sure you update the correct secret with the PAT that
has the correct permissions. In our GH worklows we refer to the secret by name,
and they need to have the right permissions.


## FYI: Workflows triggered by a PR from a fork

- GitHub secrets (e.g., PATs) are not passed to wfs triggered from a forked repo (e.g., a PR from a fork), _with the exception of `GITHUB_TOKEN`_
- However, when a wf is triggered by a PR from a forked repository, GitHub only grants read access tokens for `pull_request` events, at most. 
So, to allow writing to the PR (e.g., adding a label), the wf trigger event needs to be changed to [`pull_request_target`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target) instead of `pull_request` (see example [above](#create-workflow)).
  - This works because `pull_request_target` alters the [context of the action](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target) and safely (?) grants additional permissions.

See also:
- [GitHub token permissions documentation](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)

## Authenticating using the Neurobagel Bot app
**Neurobagel Bot app:** https://github.com/organizations/neurobagel/settings/installations/52978124

**Associated org secrets and variables:**
`NB_BOT_ID`: variable, app id (not client id!) for app
`NB_BOT_KEY`: secret, private key for app

- Neurobagel Bot (`neurobagel-bot`) was created and is installed under the @neurobagel organization
  - The app has been given access to all repositories in the org
- **App permissions (repository level)**:
  - Read access to metadata
  - Read and write access to actions, administration, checks, code, commit statuses, deployments, issues, pull requests, repository hooks, and workflows
- the app ID and private key for the app can be [used in a workflow](https://docs.github.com/en/enterprise-server@3.11/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow) to generate an installation access token, which allows authentication on behalf of the app (as an app installation which has been granted access to certain repository resources with certain permissions)
  - note: the app's private key is different than the app's **client secret**, which is used to generate user tokens (more info: https://docs.github.com/en/enterprise-cloud@latest/apps/creating-github-apps/about-creating-github-apps/best-practices-for-creating-a-github-app#secure-your-apps-credentials)


**References:**
Modifying a GitHub app:
https://docs.github.com/en/enterprise-cloud@latest/apps/maintaining-github-apps/modifying-a-github-app-registration

Full docs for authenticating with a GitHub app: 
- https://docs.github.com/en/enterprise-server@3.11/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow
- https://docs.github.com/en/enterprise-cloud@latest/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-as-a-github-app-installation

More info on private keys, client secrets, etc.:
https://docs.github.com/en/enterprise-cloud@latest/apps/creating-github-apps/about-creating-github-apps/best-practices-for-creating-a-github-app#secure-your-apps-credentials