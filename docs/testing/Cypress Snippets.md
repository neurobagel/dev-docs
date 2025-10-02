---
title: Cypress Snippets
tags: [moved]

---

## Cypress unit testing

Aside from browser-based testing, cypress is also just a neat test runner for other JS testing libraries. 
So we can use it to run unit tests. 
This is useful for testing store methods like getters and mutations that don't have any UI elements.

All of the information below is taken from this very cool blog post: https://dev.to/bahmutov/unit-testing-vuex-data-store-using-cypress-io-test-runner-3g4n
The vuex testing docs are also good: https://vuex.vuejs.org/guide/testing.html and because cypress is just the test runner, we can use the syntax of the tests from there directly as well.

### Unit test a store mutation

If our mutation is called `increment` like so:

```javascript
export const mutations = {
  increment(state) {
    ...
  },
}
```

```javascript
import { mutations } from '../../src/counter'

describe('mutations', () => {
  context('increment', () => {
    const { increment } = mutations
    it('INCREMENT', () => {
      const state = { count: 0 }
      increment(state)
      // see https://on.cypress.io/assertions
      // for "expect" examples
      expect(state.count).to.equal(1)
    })
  })
})
```

### Unit test a store getter

If we have defined a store getter like so
```javascript
export const getters = {
  myGetter (state, someArg ) {
    return state.someVariable.filter(value => "someLogic")
  }
}
```

Then we can test the getter like so:

```javascript
import { getters } from './getters'

describe('getters', () => {
  it('filteredProducts', () => {
    // mock state
    const state = {
      someVariable: [
        { id: 1, title: 'Apple', category: 'fruit' },
        { id: 2, title: 'Orange', category: 'fruit' },
        { id: 3, title: 'Carrot', category: 'vegetable' }
      ]
    }
    // mock getter
    const filterCategory = 'fruit'

    // get the result from the getter
    const result = getters.filteredProducts(state, { filterCategory })

    // assert the result
    expect(result).to.deep.equal([
      { id: 1, title: 'Apple', category: 'fruit' },
      { id: 2, title: 'Orange', category: 'fruit' }
    ])
  })
})

```

See also: https://vuex.vuejs.org/guide/testing.html#testing-getters

## Cypress Component test snippets

Cypress mounts the component using the [vue-test-utils](https://v1.test-utils.vuejs.org/) library.
That means, even if the Cypress documentation doesn't say so,
you can use anything that the `vue-test-utils` documentation has
enabled. 

For example, Cypress doesn't tell you how to write `listeners` for
events emitted by your component, but you can look up in the 
[vue-test-utils docs](https://v1.test-utils.vuejs.org/api/options.html) how to do it.

Also take a look at:
- the [Cypress component test docs](https://docs.cypress.io/guides/component-testing/quickstart-vue) (somewhat incomplete)
- [vue-test-utils mount docs](https://v1.test-utils.vuejs.org/api/#mount) (Recall that Cypress has a slightly different syntax for assertions, using the `cy.xyz()` commands.)
- the [Vue testing handbook](https://lmiller1990.github.io/vue-testing-handbook/vuex-in-components.html) (This has some nice patterns, but is not specifically for Cypress.)

Also note that we are using Vue2 (specifically Nuxt2) and that many docs now start assuming that you use Vue3 (and thus different store libraries, component APIs, test library version, etc). So sometimes examples may be written for Vue3 and not work for us on Vue2.

### Pass Info Into the Component
When a component relies on information being passed to it by other parts of the app, then we need to:
1. Simulate ("mock") this information when the component is tested
in isolation during a component test. 
2. Pass the simulated information to the component in a way that 
looks as if it came from the expected source.

Components in Vue can receive information in a number of different ways:
- as props
- via `provide` / `inject`
- from a global store
- ...

Below are examples for how you can mock each of these and pass them to the 
component during a component test.

#### Import a Component

```javascript
import ComponentName from "~/components/component-name.vue";
```

#### Pass a Prop
```javascript
cy.mount(ComponentName, {  
    propsData: {
        myProp: "hasSomeValue"
    }
});
```

#### Import Plugins
```javascript
cy.mount(ComponentName, {
    plugins: ["bootstrap-vue", "vue-select"]
});

#### Pass a Vuex Getter
```javascript
cy.mount(ComponentName, {  
    computed: {
        myGetter: () => {
            return "myGetterValue";
        }
    }
});
```

#### Pass a Vuex Getter that Takes an Argument (or Simulating a mapGetters Field)
```javascript
cy.mount(ComponentName, {  
    computed: {
        myGetter: () => (myGetterArgument) {
            const myReturnValue = doSomethingWith(myGetterArgument);
            return myReturnValue;
        }
    }
);
```

**Note**: It is usually better to completely mock the return value of 
a getter rather than trying to import the getter from the store.
This is for three reasons:

1. Getters in Vuex take iterative inputs ([see e.g. here](https://github.com/neurobagel/annotation_tool/blob/a5f85a9a8e1a97d6f13c525e59f4b1c1c75dfc4d/store/index.js#L300C2-L308))
in the store and unit tests ([see here](https://github.com/neurobagel/annotation_tool/blob/a5f85a9a8e1a97d6f13c525e59f4b1c1c75dfc4d/cypress/unit/store-getter-getColumnDescription.cy.js#L19))
**but** when we use them in components they behave like regular functions ([see here](https://github.com/neurobagel/annotation_tool/blob/a5f85a9a8e1a97d6f13c525e59f4b1c1c75dfc4d/components/column-linking-table.vue#L67)). 
If we import a getter during non-e2e testing (i.e. when the app is not running),
we would first have to turn the getter into a regular function ourselves
by passing an also mocked store object to it.
2. Getter can have other getters as dependencies. 
Not only will all of these getters have to be made into normal functions
as well, but so do their dependencies in turn, and so on.
Additionally all of these getters depend on different parts of the 
store to be mocked, so we can quickly approach a situation where
we are almost mocking the entire app at runtime, just to make a 
component test "easier".
3. Importing getters is less readable than just showing what
information the mocked getter will give to the component.
This is maybe the most important reason, because readability
is key for our tests.

**In short**: mock getters as simple return objects everywhere.
The only exception is when you are unit-testing the getter itself.


#### Pass a Vuex State Field (or Simulating a mapState Field)
```javascript
cy.mount(ComponentName, {  
    mocks: {
        $store: {
            state: { myStateField: "myStateValue" }
        }
    }
});
```

### Respond to information coming out of the component

Components may have to send information to other parts of the app when certain events occur
or conditions are met. When the component is tested in isolation during a component test,
these other parts of the app don't exist. However, during a component test we only care about the component.
So the only thing we need to test is that the component emits the right type of information
in the right situations. We don't have to simuate other parts and how they would respond to 
this information (this is done during full integration tests instead). 

As with passing data to components, there are different ways data can come out of components:

- emitted events
- store mutations and actions

Below are some snippets for how you can listen to these types of data flows in a component test.


#### Listen to a Vuex State Mutation Being Committed
```javascript
const mockStore = {  
    commit: () => { }  
};  
cy.spy(mockStore, "commit").as("commitSpy");

cy.mount(ComponentName, {  
    mocks: {  
        $store: mockStore  
    }
});

cy.get("something").click(); // Or, do something to evoke the mutation
cy.get("@commitSpy").should("have.been.calledOnce");
```
You can chain assertions off of `should`, see [the docs](https://docs.cypress.io/guides/references/assertions#Sinon-Chai).


#### Listen to a Vuex State Action Being Dispatched
```javascript
const mockStore = {  
    dispatch: () => { }  
};  
cy.spy(mockStore, "dispatch").as("dispatchSpy");

cy.mount(ComponentName, {  
    mocks: {  
        $store: mockStore  
    }
});

cy.get("something").click(); // Or, do something to evoke the mutation
cy.get("@commitSpy").should("have.been.calledOnce");
```

#### Check that a Specific Payload was Sent with the Mutation / Action
```javascript
const mockStore = {  
    commit: () => { }  
};  
cy.spy(mockStore, "commit").as("commitSpy");

cy.mount(ComponentName, {  
    mocks: {  
        $store: mockStore  
    }
});

cy.get("something").click(); // Or, do something to evoke the mutation
cy.get("@commitSpy").should("have.been.calledWith", "myMutationName", {
    payloadField: "payLoadValue"});
```

#### Listen for an emitted event
```javascript

const mySpy = cy.spy().as("mySpy");
cy.mount(ComponentName, {
    listeners: {
        emitEventName: mySpy,
    }
});

// Perform an interaction to evoke the emit
cy.get(`.<class-name>`).click();

// Check to see if emit was called
cy.get("@mySpy").should("have.been.called");

// Something to note is that 'emitEventName' must maintain the type of case used by the component (i.e. kebab-case, or camel case)
```

#### Check an emitted event's payload
```javascript
const mySpy = cy.spy().as("mySpy");
cy.mount(ComponentName, {
    listeners: {
        emitEventName: mySpy,
    }
});

// Perform an interaction to evoke the emit
cy.get(`.<class-name>`).click();

// Check to see if emit was called with the correct payload
cy.get("@mySpy").should("have.been.calledWith", {
    payloadField: "payLoadValue"
});

// Something to note is that 'emitEventName' must maintain the type of case used by the component (i.e. kebab-case, or camel case)
```

#### Listen for a method call
```javascript
cy.spy(ComponentName.methods, 'methodName').as("mySpy");
cy.mount(ComponentName);

// Perform an interaction to evoke the method
cy.get(`.<class-name>`).click();

// Check to see if method was called
cy.get("@mySpy").should("have.been.called");

// Something to note is that 'methodName' must maintain the type of case used by the component (i.e. kebab-case, or camel case)
```

#### Check a method call's payload
```javascript
// Given methodName(value-1, value-2)

cy.spy(ComponentName.methods, 'methodName').as("mySpy");
cy.mount(ComponentName);

// Perform an interaction to evoke the method
cy.get(`.<class-name>`).click();

// Check to see if method was called
cy.get("@mySpy").should("have.been.called", "value-1", "value-2");

// Something to note is that 'methodName' must maintain the type of case used by the component (i.e. kebab-case, or camel case)
```

#### Provide different responses based on a condition when intercepting requests
The following snippet is from `Checkbox.cy.js` in the query tool. When the `submitQuery` button is clicked for the first time the condition `isFirstClick` is set to false and `response` is replied by `cy.intercept` and thereafter every click of the `submitQuery` button will result in `response2` being replied.

```JavaScript
let isFirstClick = true;

cy.intercept('GET', 'query/*', (req) => {
  if (isFirstClick) {
    isFirstClick = false;
    req.reply(response);
  } else {
    req.reply(response2);
  }
});
```

## Miscellaneous

### Stubbing a function from an external dependency

Context: In the code we're using compile method Ajv library to validate a data dictionary against a schema.
In order to mock an invalid result from the validation we:

- create a stub function to replace the validation function returned by `Ajv.prototype.compile`
- add mock validation errors to the `errors` property of the stub function
- stub the `Ajv.prototype` to replace `compile` with our stub function created earlier

```TypeScript
const validateStub = cy.stub().returns(false) as ValidateFunction;

validateStub.errors = [{ instancePath: '/column1' }, { instancePath: '/column2' }];

cy.stub(Ajv.prototype, 'compile').callsFake(() => validateStub);
```