---
title: Cypress Vue Testing
tags: [Tests, Internal Documentation, Frontend]

---

# Cypress Vue Testing

## Setting Up and Running the Testing Environment

1. Clone the repo you wish to test to your machine via `git clone <repo github url>`
2. Install package and developer dependencies via `npm install` in the local cloned folder
3. In the terminal, start up the package locally using command `npm run dev`
4. In another terminal, open the Cypress interface using command  `npm run cypress:open`
5. In the Cypress interface, choose testing type you wish to do: ‘end to end’ or ‘component’, and choose your desired web browser to begin testing
6. Tests can be done via the Cypress interface on your machine, but they will also be automatically run each time there is a new commit to the repo on Github. This is triggered via a Github Action configuration file at `.github/main.yml`.
7. Both Cypress’ local interface and sections in the Github interface will feature green check marks when tests have completed successfully

## A Note on Contributing Tests for the Neurobagel Annotation Tool

Our Nuxt-based annotation tool uses the Cypress JavaScript testing framework for end to end and component testing. While Cypress is an easy to use testing framework built upon past precedents and technologies like MochaJS, below are a few of our project-specific recommendations and guidelines for the creation of tests during feature development, enhancements, and bug fixes. This information was accrued in two ways: reading Cypress’ documentation and codebase and firsthand experience with the framework guiding what works best for the app we have designed.

## How to Write a Test

1. Determine what kind of test you're writing: end to end or component
  - If creating a brand new set of tests, create a js file in the appropriate folder using the `.cy.js` file extension
2. Create `describe`, `context`, and `it` functions.
  - `describe` is a container function mostly used to the describe the suite of tests in the file
  - `context` is a container function used to group different kinds of tests within the file and also is used to describe that context
  - `it` defines the individual, specific test you are writing
3. Both `describe` and `context` can contain `beforeEach` functions that will be run before every `context` or before every `it`, respectively
  - Use the `appSetup` function in your `beforeEach` for the `describe` function. This calls a set of common app configuration commands.
4. Writing individual tests
  - Tests contain three primary sections: setup, action, and assert. They can be repeated as needed depending on the complexity of the test. But a good guideline to follow is to keep tests as simple as possible.
  - It is possible to work with multiple datasets for one test file. The paradigm that has been setup can be seen, for example, in `annotation-pagetests.cy.js`. The idea is that you utilize json files in the `fixtures` folder that describe each dataset and can use functions `datasetMeetsTestCriteria` and `loadAppState` to reflect both the data needs of your test and the dataset that's being used for the test.

## Further Tips

### When to use `cy.get` and if we can rely on the DOM state

Since we are using the server-side rendering version of Vue/Nuxt, it is acceptable to check the DOM for state. (In a client-side rendered app, it would not be.) Therefore we can utilize `cy.get` and its corresponding timeout time to watch for the appearance of objects on the page. However, there are other means for checking the app state. This includes checking the annotation tool Vuex store and can be done via the `getNuxtStoreValue` command.

### Making testing tests easier

Cypress also includes functionality to aid test writers iteratively develop tests and run select groups of tests in sequence.

_.only_ and  _.skip_

This Cypress syntax allows test writers to 'only' run certain tests in a file and 'skip' others. This syntax is added directly after the `it` function name. To only run one individual test in a file, you would write `it.only(...`

### Notable files, functions, and variables

_$nuxt_

This variable is a direct reference to the Nuxt object. Though it should be used sparingly, this gives us direct access to the Vuex store that the annotation uses – and all of its actions and getters. 

_support/commands.js_

This file contains functions that are linked to the Cypress object. The idea is to place highly reused functionality here. Any function defined here using the `Cypress.Commands` syntax may be invoked within a test function via `cy.`

_assertButtonStatus_

Checks to see if a button with given data-cy attribute is enabled or disabled

_assertNextPageAccess_

Since each page in the annotation tool uses a standard naming convention for the nav and next page buttons, this function is a quick means of determining of the page's requirements for proceeeding to the next one have been met.

_datasetMeetsTestCriteria_

This function takes an object specifying page-specific criteria for a test and compares it to the config json of the dataset being used (also passed in).

_dispatchToNuxtStore_ and _getNuxtStoreValue_

These functions are quick means of getting access to the Vuex store, either for calling actions or getting values via getters. This is possible because of Cypress' access to the `$nuxt` object.

_loadAppState_

This function gets the Vuex store state to where it needs to be for a particular page. It sets up the store with desired test criteria based on the current dataset being used.

## End to End Testing

### Organization

End to end (e2e) tests are split between two folders: `app` and `page`. The `app` folder is where full and semi-e2e test files should be placed. A 'full' e2e test moves from the home page to the download page while a `semi-` e2e test may start with any page and end with any page. The idea here is to test functionality on one page, navigate through its interface, and test its output(s) on subsequent pages. The `page` folder is where page-specific tests should be placed. These are tests that are more focused on testing all of the components on a page and their interactions. For the purpose of these tests, the state of the app needed to begin testing the page is loaded programmatically (See `loadAppState` below for description.)

## Component Testing

In contrast to end to end testing in Cypress, component testing requires a slightly different mindset. Below are some important things to keep in mind when creating a component for testing and when setting up the actual test itself.

### Component creation and test creation

1. Creating a brand new component with component testing already in mind gives us the advantage of simplifying what will eventually be a more complex system of components interacting within a larger web app.

2. As such, begin creating a component with clear input and output data in mind. What are the conditions which the component needs to successfully set itself up? What is considered passable output data?

3. During the development of our apps we have discovered that it is best to decouple a component from calls to the store almost entirely, and that it's better to emit output data back up to a more central parent component or page. This helps us for component testing as 1) the Vuex store is not initially available and 2) it negates the need to set it up for the component test.

4. Overall, the notion of component testing itself is to simplify testing and reduce the brittleness of the overall app. The idea is that if each component of an app can successfully handle inputs and correctly satisfy its output conditions then the app itself will be less prone to errors when new features or new input data are introduced.

### Component test setup

1. End to end tests are a fully-fleshed out piece of the Cypress testing suite. They recreate the environment in which your app sets itself up and begins. However, component tests have a much more stripped down setup functionality. This can be seen by looking at what is available in the 'this' object during a test. The Vuex `store`, for instance, is not present. This results in the need to produce several pieces of input for component creation.

2. The value of props must be added during the mounting of a component. They are included in the second argument of the `cy.mount` command as a `propsData` key in an object. (In Vue3, this is simply `props` instead.)

3. The value of injects must also be added during the mounting of a component. This is different than props values however because injects are essentially considered a 'global' variable. Keep in mind that the reason we sometimes use injects in our code is to avoid overly-cluttered passing props down to children and/or back up to parents. Since component testing is still in beta, it appears the best way to add inject variables is through the `mixins` property of that same second argument in which we pass `propsData`. What we are doing is acting as if data regularly injected into the component is instead a local data variable. (This pseudo-injection can be seen in `https://github.com/cypress-io/cypress/blob/master/npm/vue2/src/index.ts` where `installMixins` is called alongside a few other setup functions are called for filters, plugins, and global components.)

4. There is a difference, however, in where we place inject _data_ vs _methods_. This makes sense when we think of the mixin for injects as something that will be merged into the component object definition. For _data_ we create a `data: function() { return { key: value } }` similar to how we would when creating a Vue component (minus the fancier `data()` syntax). And for _methods_ we add a method within the `methods: { }` component property.

5. Required plugins must also be listed when mounting a component. They are simply included as a list of strings in the `plugins` property of the second argument of the `cy.mount` command. If you are unsure which to add, take a look at the `plugins` property in `nuxt.config.js`. Just include the name of the plugin. There is no need to add the prefix `@/plugins/` as Cypress knows where to look for plugin code.
