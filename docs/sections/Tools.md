---
title: Tools

---

Smart things we know about tools

## [Vue.js devtools](https://devtools.vuejs.org/)
A niche browser extension for Vue development. It offers access to Vue components, timelines, routes, and Vuex.

## JSON

Web based json-schema validator that can create custom URLs to point to specific examples.
Great for issues:
https://www.jsonschemavalidator.net/

## Dependabot

The dependabot config file is stored in the `.github` directory e.g., [query tool's dependabot.yml](https://github.com/neurobagel/query-tool/blob/main/.github/dependabot.yml).

### Ignore

To prevent a specific version bump, add the `ignore` directive to the dependabot.yml file. 
This [snippet](https://github.com/neurobagel/query-tool/blob/d659250626be9a1ae495c6c39061c388d6438442/.github/dependabot.yml#L19-L23) from query tool's dependabot config file instructs dependabot to ignore `nuxt` and `vue` major version updates.

```yaml
version: 2
updates:
  - package-ecosystem: 'npm'
    directory: '/'
    schedule:
      interval: 'weekly'
    ignore:
      - dependency-name: 'nuxt'
        update-types: ["version-update:semver-major"]
      - dependency-name: 'vue'
        update-types: ["version-update:semver-major"]
```

In case of a version update that causes major issues in the project i.e., breaks things, reset the version and add the dependency to be ignored. See [this PR](https://github.com/neurobagel/query-tool/pull/82) of the query tool for a demonstration.



See [GitHub documentation](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#ignore) for more information.

## Netlify

We use netlify to do [deployment previews](https://docs.netlify.com/site-deploys/deploy-previews/) 
during Pull Requests. 
That means that when a PR is opened, 
Netlify deploys the suggested changes to a temporary URL 
and pastes a link into the PR,
so that we can look at the changes as they would look in production.

To enable netlify for a new Repo, 
just connect a Netlify account to Github and have it watch the repo.
For the js tools, you have to provide the build argument `npm run generate`.
See here for [documentation](https://www.netlify.com/blog/2016/09/29/a-step-by-step-guide-deploying-on-netlify/)

Note that the current deployments are configured under @surchs' netlify account.


## Markdown

Pandas can export a dataframe as a markdown table. 
This can be useful to generate markdown table examples on GH from actual files.
See: https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_markdown.html

## NPM

`npm install` doesn't update the `node_modules` directory by default. It installs the dependencies listed in the `package.json` file into the `node_modules` directory. If the dependencies are already installed, `npm install` will not update them unless you specify the --force flag.


## GitHub Actions

### Workflow file syntax
Job names otherwise referred to as job identifier by GitHub may only contain alphanumeric characters, '_', and '-'. IDs must start with a letter or '_' and and must be less than 100 characters.