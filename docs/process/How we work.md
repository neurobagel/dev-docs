---
title: How we work
tags: [Process]

---

As a team, we organize our work using a couple of tools and meetings:

## Our [Kanban board](https://github.com/orgs/neurobagel/projects/1)
The Kanban board is the single place where we visualize and track our work as a team. It visualizes what is currently a priority, and shows what is being worked on.

### Overview
The board is organized from left to right:

- Backlog: An ordered list of issues that we have committed to addressing next. Issues are ranked from top to bottom in order of their importance to our aims as a team / project.
- Specify: Whenever someone is free, we take the top issue from the Backlog and pull it into "Specify". Here we check that the issue specification is clear and complete. Here we also split up bigger issues into smaller sub-issues to make sure that all issues on the board have a similar size

### Issues

We do most of our discussion and planning on issues.
To make sure everyone has all the relevant context up top,
make sure the issue description is always up to date and reflects the most recent discussion.

When an issue is done being specified, it should have a couple of TODO checkboxes so the work
required and the progress so far can be seen at a glance.

### Pull requests and reviews

Pull Requests and reviews are how we make sure the implementation meets what is in the issue spec
and is correct and fits our standards.
Pull Requests can take a lot of time for the reviewer, so when you open a PR, it is important that
you keep the changes focused on a single problem and that you provide clear context
to help with the review.

Here are some examples of good context to provide:

- Anything to pay attention to
(e.g., edge case, something not implemented, new dependency introduction for custom solution)
- Related issues
- If PR includes fixes for multiple issues, structure them somehow in the description for review
  - Try to avoid comining multiple issues in a single PR
- Make sure that issue has much context as possible
e.g., with comments, updating description during implementation

### FAQ: How should I ...

**While working on an issue, I found a bug. where should it go**?

- create an issue for the bug in the correct repository
- apply the bug label and decide what severity the bug should have and apply the severity label
- is the bug severe (e.g. does it not have an easy workaround or is not just a nuisance)
    - yes, it is a severe bug
        - add the bug to the project 
        - **does the bug block your currently worked on feature**?
            - yes, I'm blocked by this bug
                - move your issue into "track" and reference the bug issue
                - replace your issue with the bug and begin addressing the bug (or get help if needed)
            - no, I am not blocked by this bug
                - set the status of the bug to "backlog"
                - discuss it during the next standup and advocate for where you think it should be in the priority list
    - no, it is not a severe bug
        - do not add the bug to the project
        - we will periodically revisit these bugs via automation

**I just had an idea for something that might be useful later**

Thoughts that are not part of the current focus 
and still need some further thinking to clarify
can still be useful. 

- make an issue in the appropriate repo
- make a detailed description so that someone with no context in 6 months still understands what you have in mind
    - if you can't (be bothered to) write a detailed enough description, then your idea isn't important enough for an issue
- apply the "feature idea" (or similar) label to it
- do not add it to the project

**I just had and idea, and it is related to the current focus**

If your idea is related to what is currently our main focus
or roadmap issue, then we need to discuss the idea quickly.

- create an issue in the correct repository
- add the issue to the project and set the status to "backlog"
- advocate for your issue during standup and for where you think it should go in the priority ranking
- be ready to have the team disagree with you and demote the issue to a simple feature idea

## Proposed changes to process
Situation: I want to implement an organization-wide change that will improve the quality of code/process/developer life in general, e.g., create or update a specific issue template.

Steps: Mention it during standup (or in the `#Neurobagel` channel) before making a change so that any team members who have suggestions can reach out.

## Daily standup
Every day at 12 pm we meet and talk.

## Automatic Github processes
Neurobagel is built and maintained across several repositories,
but our work is organized at the level of the entire organization.
That means we have to synchronize issues, labels, checks, and 
workflows across many repositories so they are consistent.

### Synchronizing workflows
Because that can get pretty repetitive and annoying,
we use Github automation wherever we can to make our life easier.
Github workflows that are needed in many repositories are 
maintained in our [neurobagel/workflows repository](https://github.com/neurobagel/workflows), specifically in the [template_workflows directory](https://github.com/neurobagel/workflows/tree/main/template_workflows).
To synchronize the template workflows to our other repos,
we use [the `sync_wf` workflow](https://github.com/neurobagel/workflows/blob/main/.github/workflows/sync_wf.yml) that is unique to the workflow repo.
The `sync_wf` workflow listens for changes in the template_workflows 
directory and whenever one of the template workflows is edited,
`sync_wf` opens a pull request against our other neurobagel repositories
to copy or update the workflow there.

The [sync](https://github.com/neurobagel/workflows/blob/main/.github/sync.yml) is a config file (containing repo names) which is read read by the `sync_wf`. To sync required workflows for a repository, the name of the repository needs to be added to the `repos` field of the relevant file in the sync config file.

[Templates](https://github.com/BetaHuhn/repo-file-sync-action?tab=readme-ov-file#using-templates) can be used to propagate different values of variables to each synchronized workflow.
These variables can include strings with typical GitHub variable expansion syntax if each repo's workflow needs to use a secret as part of the workflow, with the following behaviour for expansion in the generated/synchronized workflows:

| In template | Turns to | i.e. |
| ----- | ----- | ----- |
| `${{ secrets.NOQUOTE }}` | `${{ secrets.NOQUOTE }}` | stays the same |
| `'${{ secrets.QUOTED }}'` | `${{ secrets.QUOTED }}` | string removed, same outcome as no quotes |                                                                                            
| `${ secrets.SINGLECURLY }` | `${ secrets.SINGLECURL }` | stays the same |
| `{{ secrets.BETAHUHN }}` | `[object Object]` | has something unfortunate happen to it during expansion in JavaScript land and gets turned into an object -> effectively still benign |

It seems that as long as there is a `$` in front of the curly braces, they stay the same after the template is applied. And if we forget the `$`, it just turns into useless stuff and breaks, but we still don't leak secrets. 


### Synchronizing issues and labels
We also want to use consistent labels across our
repositories without having to re-create them everywhere
or update them manually whenever we want to change them.
All labels that are common to all repositories are maintained
in [our planning repository](https://github.com/neurobagel/planning).

Whenever a label in the planning repository is added or changed,
our [`label_sync` workflow](https://github.com/neurobagel/planning/blob/main/.github/workflows/label_sync.yml) ensures that
this change is propagated to all other repositories.

When we create a new repository it won't have any labels yet.
For this special case, we have a second workflow that can 
be triggered manually here: [https://github.com/neurobagel/planning/actions/workflows/manual_label_population_to_repos.yml](https://github.com/neurobagel/planning/actions/workflows/manual_label_population_to_repos.yml).
This workflow will take **all** labels in the planning repo
and copy / update them to all neurobagel repositories in one go.

Finally we also have some Github automation to make it easier
to track issues across repositories. 
The [`add_iss2project` workflow](https://github.com/neurobagel/workflows/blob/8515e4510f8e3b0aa5fde16c4166fb69a93b8de8/template_workflows/add_iss2project.yml) is maintained
in the workflows repository and synched with all repos from there (see above). 
Whenever a new issue is opened in any of our repositories, 
this workflow adds the issue to our neurobagel project.

**Note**: Because we are synchronisation labels from the planning repo, 
you should only create or edit labels inside of the planning repo.
An exception is if you want a label to only exist in a specific repo.
In this case, you should create the label in this repo, 
it will not be synced or edited by the synchronisation workflow.

### Deleting the same label across multiple repos
Unlike for label creation/editing, we currently do not have an automatic process to delete labels from all repos if it has been deleted in `neurobagel/planning`, as this should be approached with more caution.

Instead, the following script can be automatically used to delete >=1 labels matching a specific label name, color, and description from all Neurobagel repos.

To use it, update the `LABELS_TO_DELETE` variable as needed.

If the label should be deleted from only a subset of repos, replace the value of `REPOS` with a space-separated string of repository names.

```bash
#!/bin/bash

# Get all the Neurobagel repos
REPOS=$(gh repo list neurobagel --json name | jq -r '.[].name')

# NOTE: the color is case-sensitive in the matching, and should not have # in front of it
LABELS_TO_DELETE=(
	"labelone,c5def5,test label 1"
	"labeltwo,7D939A,test label 2"
)

# NOTE: make sure ${REPOS} doesn't have quotes around it
for repo in ${REPOS}; do
	echo "Processing repository: ${repo}..."

	for label in "${LABELS_TO_DELETE[@]}"; do
		# Get the label attributes
		IFS="," read -r label_name label_color label_description <<< "$label"
		
		# Check if the label exists with the specified name, color, and description
		# NOTE: we set the label limit -L to something pretty big to ensure all existing labels are fetched
   		gh label list -R "neurobagel/${repo}" -L 200 --json name,color,description | jq -e ".[] | select(.name == \"$label_name\" and .color == \"$label_color\" and .description == \"$label_description\")" >/dev/null

	if [ $? -eq 0 ]; then
      		gh label delete "$label_name" --repo "neurobagel/${repo}" --yes
    	else
      		echo "Label not found: $label_name"
    	fi
  	done

  	echo -e "Finished processing repository: ${repo}\n"
 done
```

## Testing out GitHub workflows locally using `netkos/act`

[act](https://github.com/nektos/act) allows you to locally run workflows in a repository.
This is useful when you want to quickly check (see also the [Limitations](https://github.com/neurobagel/documentation/wiki/How-we-work#limitations) section below!) the behavior of a workflow you have changed, but that workflow does not run on a PR (normally, you might have to merge your changes first and then directly run the workflow on `main`, which can be a bit annoying).

The documentation for `act` is pretty clear, and can be found in the repository README and their [user guide](https://nektosact.com/). Their [example commands](https://github.com/nektos/act?tab=readme-ov-file#example-commands) are especially helpful.

`act` needs to be installed locally, and needs Docker to work. System-specific installation instructions [here](https://github.com/nektos/act?tab=readme-ov-file#installation).

For example, this is what I did (I don't have homebrew, so I just installed it as a bash script):
```bash
curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# then, move it to my /usr/bin so it's executable as a command
sudo mv bin/act /usr/bin

# remove leftover directory
rm bin

# check if it works
act --help
```

A more detailed example of how to run it on a specific workflow from a repository, e.g., `neurobagel/bagel-cli`:

We can first list which jobs and workflows are available in the repo
```bash
cd bagel-cli
act -l
```

Let's assume we want to run the job `auto-release` from the release.yaml workflow. 
This job requires the `NB_PAT_RELEASE` PAT, we will also need to provide a valid PAT under this variable name for `act` to use.
Here, I used my own fine-grained PAT created with the same permissions as `NB_PAT_RELEASE`, and provided it using the [`--secret-file` option](https://github.com/nektos/act?tab=readme-ov-file#secrets):

I first created a file called `my.secrets`:
```bash
NB_PAT_RELEASE=[my PAT value copied from GitHub]
```

Then, I passed it to act when calling the job:
```bash
act -j auto-release -W .github/workflows/release.yaml --secret-file ../my.secrets
```

You should see a message `Job succeeded` when the job completes successfully (Note: you might still get this message even when not all steps in the job have been entirely completed - see Limitations section below).

**note**: I can also provide a custom event payload to an `act` invocation: https://nektosact.com/usage/index.html?highlight=payloa#using-event-file-to-provide-complete-event-payload

### Limitations
*While `act` aims to replicate the GitHub Actions environment, certain aspects will differ when the workflows are run in your local repo. 
As a result, some local adjustments might be necessary for specific jobs, while the context of other jobs may be too specific/constrained to run properly or to completion using `act`.

For example, given that our `release.yaml` workflow is configured to run only on pushes to the `main` branch, and also relies on merging PRs with the `release` label to actually initiate the release and calculate the version bump, etc., it appears that 1) running in the workflow with `act` on a non-`main` branch does not actually initiate the release creation, and 2) even running it on `main` is not sufficient to cause a release to be published with an appropriate version number.
For example, the run on `main` would end with the following:
```bash
| ✔  success   Calculated SEMVER bump:
| ✔  success   Calculated version bump: none
| ℹ  info      No version published.
| ✔  success   Teardown `auto`
[auto release/auto-release]   ✅  Success - Main Release
[auto release/auto-release] Cleaning up container for job auto-release
[auto release/auto-release] 🏁  Job succeeded
```
(In this case, the local run still finishes with a Success message, which is a little bit misleading!)
These types of jobs likely require additional context 'mocking' to run properly that `act` may not be able to support.

Another apparent limitation is jobs that require some kind of uploader/uploading to an external service, such as test workflows that upload coverage reports to Coveralls/Codecov. 
These jobs, when run locally, seem to consistently fail at the step of uploading/posting the coverage report (regardless of repository or the coverage suite used).
One possible workaround is to skip the step that posts the coverage data, following the instructions [here](https://github.com/nektos/act?tab=readme-ov-file#skipping-steps) (have not tested).

Finally: `act` runs on specific docker images that are purposefully not complete replications of the GH workflow runner environments. 
This means that some commands you have in GH actions won't exist in `act`:
https://github.com/nektos/act/issues/107
You can use a more complete docker image: https://nektosact.com/usage/runners.html
but note that the biggest one is 18GB!

### Other important notes
NOTE 1: Based on the docs, I think you shouldn't _have_ to specify the workflow path in addition to the job ID, but for some reason when I tried it without the `-W` flag, I got a `Could not find any stages to run` error. I think this is an open issue, https://github.com/nektos/act/issues/1993.

NOTE 2: If passing a secrets file to the `act` command which contains `GITHUB_TOKEN`, **the variable cannot be defined inside the file as follows**:
`GITHUB_TOKEN="$(gh auth token)"` (unlike when passing the `GITHUB_TOKEN` directly using `-s GITHUB_TOKEN`), because the .secrets file does not support this kind of expansion syntax. See https://github.com/nektos/act/issues/2130#issuecomment-1859288328 and https://nektosact.com/usage/index.html?highlight=env#envsecrets-files-structure for more info. 
As a work around, you can echo the value into the file instead, e.g.: `echo GITHUB_TOKEN=$(gh auth token) >> my.secrets`

NOTE 2: You can also try a dry-run first, e.g.
```bash
act -j auto-release -W .github/workflows/release.yaml --secret-file ../my.secrets -n
```
but I'm not sure how far the dry-run actually goes/how helpful this is. For example, I first tried a dry-run locally but forgot to provide the secrets file needed for `auto-release`, and still got a "Job succeeded" message. Then when I ran it not as a dry run, the job failed. 🤷 So the dry-run doesn't seem to actually 'catch' all problems by default.

## Working with long-lived development branches

- Maintenance Strategy: keeping `develop` in-sync with `main`
If `main` has received changes/PRs and has moved forward after `develop` was branched off and `develop` is still being worked on, `main` must be merged into `develop` immediately after any changes land in `main`, This keeps development branch up-to-date with main. the strategy is often referred to as `The Back-Merge`: checkout to `develop` branch and merge main into `develop` i.e., `git checkout develop && git merge main`

- Feature strategy: merging feature into `develop`
If a PR is opened against `develop`, it should be squashed and mergeed since the two branches share history and we want to see one clear commit per feature/fix/etc e.g., feat: set up normalized store, rather than 15 commits e.g., chore: fixed typo, etc. 

- Release strategy: merging `develop` into `main`
If `develop` has accumulated the planned features and is ready for release. The changes should merged using a standard merge commit since the desired goal is to perserve the history of `develop`. The merge commit acts as a "container" for that release. If we need to revert the entire release, we can revert this single merge commit

## Daisy chaining pull requests
When addressing multiple issues of the same repository in succession, it's better to not branch off of the feature branch with the latest changes as it may lead to conflicts that if not handled properly may overwrite some of the changes made. For a safer approach, branch off of main for each issue.