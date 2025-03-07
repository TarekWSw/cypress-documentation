---
title: 'Test Isolation'
sidebar_position: 35
---

:::info

## <Icon name="graduation-cap"></Icon> What you'll learn

- What is test isolation
- How it impacts E2E Testing vs Component Testing
- Test isolation trade-offs

:::

## What is Test Isolation?

:::tip

<Icon name="check-circle" color="green"></Icon> **Best Practice:** Tests should
always be able to be run independently from one another **and still pass**.

:::

As stated in our mission, we hold ourselves accountable to champion a testing
process that actually works, and have built Cypress to guide developers towards
writing independent tests from the start.

We do this by cleaning up state _before_ each test to ensure that the operation
of one test does not affect another test later on. The goal for each test should
be to **reliably pass** whether run in isolation or consecutively with other
tests. Having tests that depend on the state of an earlier test can potentially
cause nondeterministic test failures which make debugging challenging.

Cypress will start each test with a clean test slate by restoring and clearing
all:

- [aliases](/api/commands/as)
- [clock mocks](/api/commands/clock)
- [intercepts](/api/commands/intercept)
- [spies](/api/commands/spy)
- [stubs](/api/commands/stub)
- [viewport changes](/api/commands/viewport)

In additional to a clean test slate, Cypress also believes in running tests in a
clean browser context such that the application or component under test behaves
consistently when ran. This behavior is described as `testIsolation`.

The test isolation is a global configuration and can be overridden for
end-to-end testing at the `describe` level with the
[`testIsolation`](/guides/references/configuration#Global) option.

## Test Isolation in End-to-End Testing

Cypress supports enabling or disabling test isolation in end-to-end testing to
describe if a suite of tests should run in a clean browser context or not.

### Test Isolation Enabled

When test isolation is enabled, Cypress resets the browser context _before_ each
test by:

- clearing the dom state by visiting `about:blank`
- clearing [cookies](/api/cypress-api/cookies) in all domains
- clearing
  [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
  in all domains
- clearing
  [`sessionStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
  in all domains

Because the test starts in a fresh browser context, you must re-visit your
application and perform the series of interactions needed to build the dom and
browser state for each test.

Additionally, the [`cy.session()`](/api/commands/session) command will inherit
this configuration and will clear the page and current browser context when
establishing a browser session. This is so tests can reliably pass when run
standalone or in a randomized order.

### Test Isolation Disabled

When test isolation is disabled, Cypress will not alter the browser context
before the test starts. The page does not clear between tests and cookies, local
storage and session storage will be available across tests in that suite.
Additionally, the [`cy.session()`](/api/commands/session) command will only
clear the current browser context when establishing the browser session - the
current page is not cleared.

### Quick Comparison

| testIsolation | beforeEach test                                                                                                                                     | cy.session()                                                                                                                                        |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `true`        | - clears page by visiting `about:blank`<br/>- clears cookies in all domains<br/>- local storage in all domains<br/>- session storage in all domains | - clears page by visiting `about:blank`<br/>- clears cookies in all domains<br/>- local storage in all domains<br/>- session storage in all domains |
| `false`       | does not alter the current browser context                                                                                                          | <br/>- clears cookies in all domains<br/>- local storage in all domains<br/>- session storage in all domains                                        |

## Test Isolation in Component Testing

Cypress does not support configuring the test isolation behavior in component
testing.

When running component tests, Cypress always resets the browser context _before_
each test by:

- unmounting the rendered component under test
- clearing [cookies](/api/cypress-api/cookies) in all domains
- clearing
  [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
  in all domains
- clearing
  [`sessionStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
  in all domains

## Test Isolation Trade-offs

It is important to note that disabling test isolation may improve the overall
performance of end-to-end tests, however, it can also cause state to "leak"
between tests. This can make later tests dependent on the results of earlier
tests, and potentially cause misleading test failures. It is important to be
extremely mindful of how tests are written when using this mode, and ensure that
tests continue to run independently of one another.

The best way to ensure your tests are independent is to add a `.only()` to your
test and verify it can run successfully without the test before it.
