---
id: overview
title: Standards
---

## Problem Statement
* There are differing ideas on what should be covered by unit-testing
* Some unit tests end up being technical debt that does not provide any benefit and slows down development
* Often legacy code does not have any test coverage

## Proposal
* Unit tests should be written alongside all new code and kept up-to-date as code changes
* Unit tests should be meaningful and concise
* Unit tests should provide a high confidence level to future developers
* All new code should have 80% unit test coverage
* Unit tests should be a first class citizen and written with the same principles as production code

## Standards

### Tooling
* [Jest](https://jestjs.io/)

### General
* All branching logic should be covered by a unit test.
* Tests should be given descriptive, unique names.
* Tests should consistently produce the same results (i.e. don't randomize the data you are using in your test)
* Unit tests should only test one part of system at a time, all other functionality should be mocked
* Each test should be independent (it should not rely on what happened in the previous test or on external sources)
* Tests should not test the functionality of third party libraries

### Automation
* Unit tests should provide a barrier to merging into develop by running the tests in the PR build and failing the build if the tests fail.

### Mocking
* Mock all external API calls - no unit test should actually call an endpoint.
* Mock any functions that are not part of the code you are testing. For example: If testing a component that dispatches an action, mock the action.

### Testing React Components
* If a component requires a provided state, that state should include only the necessary parts of state the component requires.
* Error and loading states should be covered by unit tests, if they are present.
* As much as possible, test the component as an independent unit. If rendering the component, always use shallow rendering to avoid testing component children.
* Use something you expect to stay consistent as selectors in your tests. For example, prefer selecting by a static data attribute over a dynamically generated class name.

### Jest Snapshots
* Code should be covered by unit tests first and snapshots should be used supplementally.
* Snapshots should not be added solely to increase test coverage.

### Testing Redux

#### Connected Components
* Export just the component and test that independently of redux.
* Selectors should be tested separately from the component.

#### Actions
* Mock the dispatch function to limit testing of side effects. Test against what is dispatched rather than what the dispatch should do.
* Mock only the part of state needed for the tests.

#### Reducers
* Test all branches in reducer functions, including the default case.
* Mock only the state needed in the reducer.

#### Selectors
* Unit tests for selectors should cover all branches in function, including cases of incomplete or missing data.

## Guidelines

### File Organization
* Unit tests should live as close to the code they are testing as possible. Example: `MyComponent.js` and `MyComponent.test.js` would be in the same directory. Alternatively, a `__tests__` folder could contain all the tests for one directory.

### Automation
* A pre-push git hook should run unit tests before allowing the new code to be pushed to the remote origin.

## Examples
* Examples are available in the Examples directory.

## Additional Notes
