---
title: Cypress Tasks
tags: [Tests, Internal Documentation, Frontend]

---

## Description

We can call upon the Cypress function `cy.task` when wanting to execute NodeJS code from within the Cypress testing context. The first use case for this in Neurobagel was for https://github.com/neurobagel/annotation_tool/pull/433 where it was necessary to read the contents of the Cypress downloads folder.

[Cypress documentation](https://docs.cypress.io/api/commands/task) describes this function as such:
>an escape hatch for running arbitrary Node code, so you can take actions necessary for your tests outside of the scope of Cypress.

## Examples

Cypress gives a number of example scenarios where this might be useful including

- Returning the number of files in a folder
- Seeding a test database
- Storing a state in node that you want to persist between spec (Cypress test) files
- Saving a variable across URL visits with different origins
- Performing parallel tasks outside of Cypress
- Running external processes

## Setup and Use

Typical setup of a Cypress task occurs in the `cypress.config.js` that sits at the root of the app folder. In the `defineConfig` section under the key for `e2e` or `component` testing there is a `setupNodeEvents` function inside of which is a call to the Cypress `on` function. (See below.) Inside the second parameter of `on` is where it possible define different tasks you would like to call upon during testing.

```javascript
const nodelib = require("nodelib");

module.exports = defineConfig({

    // setupNodeEvents can be defined in either the e2e or component configuration
    e2e: {

        setupNodeEvents(on, config) {

            on("task", {

                exampleTask(argument) {
                    return nodelib.functionName(argument);
                },
                
                // More task definitions (and other external function definitions) here...
            });
        },
    },
    ...
});
```

This task can then be used in Cypress testing as follows:

```javascript
cy.task("exampleTask", "argumentValue").then(taskReturnValue => {

    // Perform more Cypress testing calls here once the task has been performed and its result returned
});
```