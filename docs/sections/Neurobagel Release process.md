---
title: Neurobagel Release process
tags: [Process, Deployment]

---

# Neurobagel Release process

For all tools, our goal is to have the `main` branch releasable at all times.
That means:

- all linters and static tools are passing
- all tests are passing
- all new features since last release have appropriate documentation (as part of PR)
- the build is successful

**Note**: For features that take a lot of work to implement,
having `main` always be releasable requires an extra step.
For example a feature to "make this experiment reproducible"
might take several PRs to implement. 
To still have `main` be in a releasable state,
we will have to hide the in-progress feature behind 
a [feature flag](https://www.atlassian.com/continuous-delivery/principles/feature-flags).

The goal is that we **could** release always,
but we **choose** to release only when it makes sense.
Related: the work to **do** an actual release should be minimal,
and only involve the actual release process:

- changelog editing
- version increment picking
- celebrating :tada:

It should not include any fixes, changes, or the writing of new documentation.

## When to release

We will decide when to make a release as a team.
The goal for releases is:

- **Release often** - A release should not be a major thing that we only do every couple of months. 
- **Release regularly** - As much as we can, the releases should be at regular intervals. This may mean that different tools have different release intervals.
- **Release when it makes sense** - The idea is that we only push released versions to our deployments. So not every merged PR will get deployed. But we will want to deploy and thus release whenever we fix an important bug or a new feature.


## Auto-releasing using `intuit/auto`

### Context
- This is done using a workflow called `release.yaml` and the `.autorc` configuration file in each repo
  - Note that the workflow uses PATs (not `GITHUB_TOKEN`, which doesn't have sufficient permissions) that have, among other permissions, R+W permissions to both issues and pull requests

### About PR labels
- `auto` uses custom labels applied to **PRs** in a repo to determine when a release is made, which PRs are included in the release (and under what CHANGELOG heading), and to calculate the version bump for the release (major/minor/patch)
  - Each of these PR labels has one associated `releaseType` (defined in `.autorc`) that basically describes the version bump that should happen if a release were to be made containing only a PR with that label applied, and also the version bump if the label were paired with others (of potentially different `releaseType`s) on the same PR
    - Possible label `releaseType`s: "major", "minor", "patch", "skip", "none", "release" (**note that these do not refer to the labels themselves, which can have different names**)
    - Multiple labels can have the same `releaseType`
    - Labels with a `releaseType` of "major"/"minor"/"patch" are called SEMVER labels - these are used to calculate the final version bump for a release

### Automatically creating the PR labels in a repo
Note: as long as the label properties don't change, this step only needs to be done once in the `planning` repo, since the labels will then be propagated to all other repos via our label syncing workflow.

1. Install auto using binary on local machine
```bash
# replace the auto version if needed
curl -LJO https://github.com/intuit/auto/releases/download/v11.0.4/auto-linux.gz

gunzip auto-linux.gz ~/auto
chmod +x ~/auto

# check that it works
~/auto --help
```

2. `cd` into the repo in which you want to create the labels
```bash
cd neurobagel/planning
```

3. Ensure the `.autorc` file is available at the root of the repo

4. Create a `.env` file in the repo root with the variable `GH_TOKEN` containing the value of a PAT with repo R+W access to at least issues and code (contents)
```bash
GH_TOKEN=value_of_your_pat_here
```

5. Run `auto create-labels` from the repo root

6. Check that the list of created labels corresponds to the ones in your `.autorc`

### Our configuration
You can find our PR label configuration here: https://github.com/neurobagel/workflows/blob/main/template_configs/.autorc
The default PR label, if no labels are applied, is `pr-patch` (which also has `"releaseType": "patch"`).

Our current `auto` configuration only releases _when a PR with the `release` **label** applied has been merged_.

When this happens, all PRs that have been merged since the last release are collected and `auto` looks at the PR label(s) on each to figure out:
  - if the PR should be included in the changelog
    - if a PR has a label of `skip-release` (has `"releaseType": "skip"`) this always takes precedence over other SEMVER labels on the same PR
  - if so, what changelog heading it should go under (based "changelogTitle" label attribute in `.autorc`)

Finally, `auto` looks for the highest-priority `releaseType` among the included PRs based on their labels, and increments either the major, minor, or patch version accordingly.

### Notes from troubleshooting automatic release workflow

- A non-default token (not `GITHUB_TOKEN`) needs to be used to create the actual release, otherwise the release is blocked from successfully triggering another workflow (e.g., building Docker image)
- Besides the release, the changelog needs to be updated and committed to the default branch
    - Because of our `main` branch protection rules (PR required, 1 approval required), we originally used the `protected-branch` plugin for auto, which auto-creates a PR for the changelog, and then auto-reviews it and merges it
        - This requires a separate, additional token (possibly for a different user?) than the one used to create the release, b/c a PR cannot be approved by the PR author
            - In our case, the `NB_PAT_RELEASE` and `NB_PAT_RELEASE_PROTECTED` tokens were created under different users, which worked out
#### Problem
- Because PRs for changelogs were now auto-approved & merged immediately after creation, pre-commit CI (which **always** runs on every commit to a PR) would not have enough time to run checks before the PR is merged and would thus often fail on these PRs, leading to red checks on `main` = bad
    - Relevant issue: https://github.com/neurobagel/workflows/issues/87
    - Here, skip keywords for pre-commit CI did not have any effect because [they only work on main branch commits](https://github.com/pre-commit-ci/issues/issues/224) (this is the difference between the `pre-commit - pr` and the `pre-commit - push` checks), not PRs
    - We needed a way to either push directly to the main branch (without needing to open a PR) from the wf, or force it to wait for the pre-commit CI
- **What didn't work**
    1. Configure `pre-commit - pr` as a `requiredStatusChecks` for `protected-branch` (https://www.npmjs.com/package/@auto-it/protected-branch?activeTab=readme#usage), to try and force this check before the PR is merged
        - Result: release failed: `requiredStatusChecks` relies on a token that has the `Checks` permission, which is currently not supported for user PATs (this is a [known issue](https://github.com/orgs/community/discussions/129512) for fine-grained PATs) - only apps can have this permission
            - classic user PATs also don't work for this
    2. Make `pre-commit - pr` a required status check for PRs at the repo level (in branch protection settings)
        - Result: release failed: workflow would error out because required pre-commit check was still pending
    3. Explicitly temporarily enable & disable force pushes in release workflow, as described in https://github.com/intuit/auto/issues/1491#issuecomment-1610120918
        - Result: release failed, with error that PR must be opened
    4. Creating a basic, no-code app ["Neurobagel Bot"](https://github.com/organizations/neurobagel/settings/installations/52978124) (`neurobagel-bot`) under the Neurobagel organization to authenticate with, to try using an app installation access token for releasing + updating the changelog, following the [GH docs](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow), and making sure it was allowed to bypass required PRs in the repo settings
        - Result: release succeeded, but changelog checks still failed
            - `pre-commit` now was able to be triggered by the `protected-branch` plugin, but pre-commit CI would still run separately and would fail due to the PR being merged first
- **What worked**
    - (it looks like this IS actually documented in the [auto docs](https://intuit.github.io/auto/docs/build-platforms/github-actions#running-with-branch-protection) from [this issue](https://github.com/intuit/auto/issues/2266#issuecomment-1532628040), and I just missed it somehow...)
    - The PATs originally being used for the release wf belonged to org admins who should (theoretically) be allowed to bypass required PRs based on current rules
        - Same for the bot app, which was allowed to bypass required PRs
        - So, it was strange when we removed the `protected-branch` plugin (which always creates a PR?), we were still getting the error in the wf failure that a PR was required
    - The culprit was that a more privileged access token needed to be given to the earlier `actions/checkout` step itself, otherwise the limited default auth level (the basic GitHub token) is used and persists in subsequent steps. This prevented more privileged git commands (e.g., pushing directly to a protected branch) from succeeding in subsequent steps, even though those steps had an appropriate token
    - Refs:
        - https://github.com/marketplace/actions/checkout#checkout-v4 (maybe worth looking into `persist-credentials: false`?)
        - https://stackoverflow.com/a/72515781
        - https://stackoverflow.com/questions/63733822/cant-push-to-protected-branch-in-github-action

#### Working configuration
- `neurobagel-bot` is installed in all repos
- Add `neurobagel-bot` as app allowed to bypass required PRs in repo settings
    - ensure that the app ID is stored as an actions variable, and private key as an actions secret
- Remove `protected-branch` plugin from auto (so custom reviewer PAT also no longer required)
- In release.yaml, obtain & use installation access token for `neurobagel-bot` in both `actions/checkout` and subsequent steps for the release creation

Related issues:
- https://github.com/neurobagel/planning/issues/64

### Create a release
1. To ensure that PRs since the last release end up under the appropriate heading in the changelog, when opening a PR, the author should always manually apply at least one PR label from our `.autorc` config (note: the `pr-patch` label is used by default behind the hood if no labels are applied, but we should aim to intentionally apply these to avoid having to modify changelog sections after a release)
    - For most changes, the author should choose the most relevant* label starting with the `pr-` prefix
    - If a PR should be excluded from the changelog (e.g., some small automation PR that is not user-relevant), the 'skip-release' label should be applied
2. To auto-release after the PR you are working on is merged, apply the `release` label to the PR along with a relevant `pr-` prefixed label (note to double check: if no `pr-` labels are applied, the default one used _should_ be `pr-patch`)
3. To include extra release notes in the changelog, you can do so by modifying the PR description following these instructions: https://intuit.github.io/auto/docs/generated/changelog#additional-release-notes
4. Once your release PR is merged, check that the release was created successfully with the expected tag + SEMVER (you can also double check that the `release.yaml` workflow in the repo succeeded), and double check the changelog

*Each PR included in the release will be assigned to a single changelog section based upon the applied label with the highest `releaseType` that has a `changelogTitle`. 
So, be careful when applying multiple `pr-` labels to the same PR - make sure that the label with the highest priority `releaseType` actually has the changelog section you want the PR to be listed under.
See also https://intuit.github.io/auto/docs/configuration/autorc#changelog-titles)


Docs: https://intuit.github.io/auto/docs

## Auto-pushing to DockerHub [WIP]

Once you have made a new release, 
a new GH workflow will start to:

1. Build a new Docker image from the tagged commit
2. Ensure that the build and test succeed (if not -> :stop_sign: -> :scream: -> :hammer_and_wrench:)
3. Tag the new Docker image with two tags:
    - `latest` to show that it is the latest released version
    - `<version>` to show that it is the specified version
4. Push the Docker image to docker hub
5. (where applicable) -> push the new version to production
    - this may be a manual process

If we encounter a bug or notice things failing somewhere during this process, fixing this bug becomes the top priority for everyone. The fix for the bug becomes the next release and then gets deployed. While the fix is being worked on we will deploy `:oldstable`.

## How to release manually (OLD)

At any given moment, the `main` branch is in a releasable state (see above).
To actually make a release, we use the normal [GitHub release workflow](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository?tool=webui).
Specifically:

1. Click the "new Release" button
2. Add a new "tag"
    - the tag has to follow semantic versioning: https://semver.org/ (see below)
    - check out the changelog (see below) to decide whether to make a patch, minor, or major version increase
3. Add a title. The title has to match the "tag"
    - i.e. for a new release called `"v0.2.4"` the Release title would also be `"v0.2.4"`
4. Use the "generate release notes" button to add the `git log` since the last release
    - This also adds a section about new contributors!
5. Edit the changelog (see also the [Example release notes template](https://github.com/neurobagel/documentation/wiki/Release-process/#example-release-notes-template))
    - for minor and major versions (optional for patches) add a short description of the release on top of the changelog
    - e.g. "This release introduces new functionality for EEG data" or "This release introduces a breaking change in the XYZ workflow ..."
    - sort the changelog by type of change (i.e. a section with all of the `[FIX]` changes, one for all the `[FEAT]`, and so on)
    - remove irrelevant changes (e.g. dependabot version bumps?)
6. Store the release as a draft
7. Discuss the ready-to-go release during standup
    - gives folks a chance to take a last look
    - possibility for feedback on the release notes
8. Release if no objections :tada: 

## Example release notes template (OLD)

## Summary
_This release introduces X._
_This release includes a breaking change to Y._

## What's Changed
_(Aim to group changes of the same type/prefix)_

### New or improved features ✨

### Changes to the data model for inputs/outputs :gear:

### Documentation updates 📜

### Bug fixes 🛠️

### Other changes 🧹
