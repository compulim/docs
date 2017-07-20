# Test packages

There are multiple JavaScript test packages. Test package include  very broad stuff and we are dividing them into multiple groups:

* Test framework - how to group all tests
  * Think about: coding style, file hierarchy, etc
* Test runner - how to run the test
  * Think about: test host, process management, rerun test, watcher, etc
* Assertion library - how to test
  * Think about: assertion, setting up expectations, etc

| | Test framework | Test runner | Assertion library |
| - | - | - | - |
| Mocha | It is | It is | It is NOT |
| Chai | It is NOT | It is NOT | It is |
| Jasmine | It is | It is | It is |
| Jest | It is | It is | It is |
| Karma | It is NOT | It is | It is NOT |

In addition, there are more tools to support a test:

* Browser automation
* Code coverage
* Reporting (usually included in CI/CD)

# Test frameworks

All test frameworks listed support asynchronous tests, including asynchronous `beforeEach`/`afterEach` and `it`/`test`.

There are multiple ways to run JavaScript asynchronously, we prefer ES6 `async`/`await` pattern for clarity. For unsupported environment, we can always Babel it.

## Mocha

[http://mochajs.org/](http://mochajs.org/)

"The test framework everyone knows."

### Points to note

* Traditional `describe('apple').it('should be red')` coding style
* Support Visual Studio Online build reports by using [JUnit reporter](https://www.npmjs.com/package/mocha-junit-reporter)
* Visual Studio Test Explorer integration thru [Node.js Tools for Visual Studio](https://github.com/Microsoft/nodejstools#readme) and a `*.njsproj` file
* Support asynchronous tests thru `done()` callback or `Promise` (`async`/`await`)
* CLI is limited, only run all tests or `grep`-ed tests, forcing you to run multiple tests everytime

### Sample code

#### Simple

```js
describe('apple', () => {
  it('should turns red and taste good after 6 months', async () => {
    const apple = new Apple();

    await apple.grow('6 months');
    assert(apple.color, 'red', 'apple should turns red after 6 months');
    assert(apple.taste, 'good', 'apple should taste good');
  });
});
```

#### Comprehensive

```js
describe('apple after 6 months', () => {
  let apple;

  beforeEach(async () => {
    apple = new Apple();
    await apple.grow('6 months');
  });

  it('should turns red', () => {
    assert(apple.color, 'red', 'apple should turns red after 6 months');
  });

  it('should tastes good', () => {
    assert(apple.taste, 'good', 'apple should taste good');
  });
});
```

#### Ultimate

The really right way to code a test, use bunches of `beforeEach`.

```js
describe('apple', () => {
  let apple;

  beforeEach(() => {
    apple = new Apple();
  });

  describe('after 6 months', () => {
    beforeEach(async () => {
      await apple.grow('6 months');
    });

    it('should turns red', () => {
      assert(apple.color, 'red', 'apple should turns red after 6 months');
    });

    it('should taste good', () => {
      assert(apple.taste, 'good', 'apple should taste good');
    });
  });
});
```

### Environments supported

| Environment | Description |
| - | - |
| Node.js | [`mocha`](http://mochajs.org/) |
| Browser | Add `<script src="https://unpkg.org/files/mocha/mocha.js" type="text/javascript" />` |
| Karma | With [`karma-mocha`](https://github.com/karma-runner/karma-mocha) |

## Jasmine

[https://jasmine.github.io/](https://jasmine.github.io/)

"The everything."

### Points to note

* Traditional `describe('apple').it('should be red')` style code
* Verify expectation by `expect(1).toBeTruthy()`. An extensible number of matchers like `toBeTruthy()`, `toBe(1)`, `toThrow()`, etc
* Angular2 works well with Jasmine
* Karma can be used to run tests across multiple browsers

### Sample code

```js
expect(apple.color).toBe('red');
```

### Environments supported

| Environment | Description |
| - | - |
| Node.js | [`jasmine-npm`](https://github.com/jasmine/jasmine-npm) |
| Browser | Thru `SpecRunner.html`, or including some `<script>` tags |
| Chrome | [`gulp-jasmine-browser`](https://npmjs.com/package/gulp-jasmine-browser) + [`simple-headless-chrome`](https://www.npmjs.com/package/simple-headless-chrome) |
| PhantomJS | [`gulp-jasmine-browser`](https://npmjs.com/package/gulp-jasmine-browser) + [`phantomjs`](https://www.npmjs.com/package/phantomjs) |
| Karma | [`karma-jasmine`](https://github.com/karma-runner/karma-jasmine) |

## Jest

[https://facebook.github.io/jest/](https://facebook.github.io/jest/)

"Write less, test more."

Almost like `describe().it('should')`, it is `describe().test('something')`. Scoping with `describe()` is optional.

### Points to note

* Can be easily code-converted from Mocha `describe().it('should')` style
* Excellent tooling
  * CLI with watching and incremental testing supported
  * On-the-fly Visual Studio Code [extension](https://marketplace.visualstudio.com/items?itemName=Orta.vscode-jest)
* Tests React by pairing server-side rendering (into strings) and snapshot testing
* Although Jest is based on Jasmine, it planned to move away from it

### WTF? No browser testing?

Jest is designed to pair up with React. And React is designed to made up of pure JavaScript code (i.e. no DOM access). Thus, your test code should also be pure JavaScript.

Although Jest doesn't run on browser directly, it doesn't need to. Running a pure JavaScript test on a browser is like testing a JavaScript VM instead of your code, it should works 98% of the time. We want to test how our app behave on a specific browser, including layout and interactions. Tests that run on browser should pair up with a browser automation library to test the actual behavior, and a screenshot test for layout.

For browser UI testing, you can either:

* Test React by outputting as server-side rendering, which is essentially a string
* Control the browser to do some stuff, then dump the Redux store in JSON, or take a screenshot

### Snapshot testing

[https://facebook.github.io/jest/docs/en/snapshot-testing.html](https://facebook.github.io/jest/docs/en/snapshot-testing.html)

"Life is short, forget about expectations."

It is like visual regression test in a textual way. Without expectations, it is like saving 50% of time.

First, you write the test and dump the output, Jest will record the dump as baseline. On the next test run, Jest will automatically verifies against the baseline. Baseline files are expected to commit into source control.

If test failed and you think the test result is good, delete/update the baseline.

Since you are also committing baseline files to source control, you will be able to easily verify your commit side-by-side.

If the snapshot is complicated to compare in strings, you can always write your own pretty-printer.

### Sample code

#### Simple

"Only if you don't like snapshot testing"

```js
test('apple should turns red and tastes good after 6 months', async () => {
  expect.assertions(1);

  const apple = new Apple();

  await apple.grow('6 months');

  expect(apple.color).toBe('red');
  expect(apple.taste).toBe('good');
});
```

#### Snapshot-oriented

"TDD is not winning UI developers heart."

```js
test('apple should be expected after 6 months', async () => {
  expect.assertions(1);

  const apple = new Apple();

  await apple.grow('6 months');

  expect(apple.color).toMatchSnapshot();
});
```

### Environments supported

| Environment | Description |
| - | - |
| Node.js | Runs with plain JavaScript, or React/Native code |
| Browser | Not supported, but you can BYOB, bring-your-own-browser |

# Assertion library

## Chai

[http://chaijs.org/](http://chaijs.org/)

Chai is a library to replace traditional `assert(actual, expected)` code. It can pairs up with any test frameworks. Do remind that Chai `should` coding style do global pollution, `expect()` is recommended.

### Sample code

#### Should

```js
apple.should.have.property('color').equal('red');
```

### Expect

```js
expect(apple).to.have.property('color').equal('red');
```

# Test runner

In short, most test frameworks comes with test runner. An extra runner could help run the tests in different environments or enhance the tooling.

## Karma

Karma primarily help you to ship your test code to multiple managed browsers. It is in two parts:

* Watcher + server: watch `*.js`, send test code to browser, and get results
* Web page: receive test code, host a URL in `<iframe>`, run your test code, send results back, teardown

Because tests are running in an `<iframe>`, interaction could be limited.

# Browser automation

Every engine/library/framework comes with different APIs, learning curve varies. Some run on top of WebDriver and some thru JavaScript injection.

Don't overtest. Browsers are external systems and your tests are not designed to test the browser capability, but how your app behave in the system. One should expect browser tests to fail often in a deterministic way.

## How to select a good one

### Async checks

One thing important to choose a good browser automator is its ability to do async checks: check if an element has text "Hello, World!". We could wait until element is visible and then get the text. But this doesn't always works because element could be visible and split-second later, the text appears.

React is an asynchronous rendering engine. We could tap into the render loop but emitting artificial event after `componentDidMount`/`componentDidUpdate` event on root container.

What we should do is repeating the test for a period of time until it has succeed. This is CPU-expensive but reduce time to run the test.

Tests could be as cheap as grabbing an element, or very expensive like taking a screenshot.

### Interactions

A nice-to-have ability is moving the mouse, for example, test the hovering state of a tooltip component.

### Taking screenshots

The third thing is the ability to do screenshot, either for snapshot testing or recording failing cases for debugging purpose.

## How to control a browser

There are 3 ways to control a browser, in order of higher controllability:

1. WebDriver, feels like an accessibility library
2. Browser scripting, run test code out-of-browser
3. Injected JavaScript, run test code inside browser

The following table list technologies and the way to control the browser:

| | WebDriver | Browser scripting | Injected JavaScript |
| - | - | - | - |
| W3C WebDriver | Yes | No | No |
| PhantomJS | Yes | Yes | Yes (thru browser scripting) |
| CasperJS | Yes | Yes | Yes (thru browser scripting) |
| Karma | No | No | Yes |
| NightmareJS | No | Yes | Yes (thru browser scripting) |

## Engine/library/framework

### Selenium/W3C WebDriver protocol

(Type: WebDriver)

Very comprehensive RESTful API for browser automation. The core protocol and everyone is layering on top of it.

* Every browser has its own "WebDriver driver", e.g. ChromeDriver spawn `Chrome.exe`, EdgeDriver spawn Edge
  * Edge driver don't forget your cookies and local storage
* Select element by CSS or XPath
* Feels like an accessibility library

### PhantomJS

(Type: both WebDriver and browser scripting)

The popular headless browser based on outdated Chrome on Qt.
Browser automation can be done by injecting JavaScript or WebDriver.

* Native WebDriver support, called GhostDriver
* Two JavaScript VMs, one for scripting the browser, another is the actual browser
* No ES6 support
* Based on Qt
  * Upside: real headless, including virtual mouse
  * Downside: font and graphics rendering very differently than our usual Chrome

### CasperJS

(Type: browser scripting)

Run PhantomJS with Node.js. You could do the same thing with [phantomjs](https://npmjs.com/package/phantomjs) package.

* Browser automation is done thru PhantomJS scripting, not WebDriver

### Karma

(Type: injected JavaScript)

In addition to test runner, Karma is also a library for browser automation. It injects JavaScript code after page load. Because test code lives in the browser JavaScript VM, they are very limited on interactions.

### NightmareJS

(Type: browser scripting)

Use Electron instead of PhantomJS, so you got up-to-date Chrome with ES6 support.

* No WebDriver support
* No `mousemove` event
* Use [Niffy](https://github.com/segmentio/niffy) for visual regression test, a.k.a. [perpetual diffing](https://segment.com/blog/perceptual-diffing-with-niffy/)
* Commands are queued and return only *single* result, i.e. no if-then statement

### WebDriver.IO

(Type: WebDriver)

JavaScript wrapper library around the bare WebDriver protocol.

* Commands are queued

# Code coverage

There are multiple code coverage tools and [Istanbul](https://istanbul.js.org/) is the one that support Cobertura format, which can be reported in Visual Studio Online builds.

## Istanbul

[https://istanbul.js.org/](https://istanbul.js.org/)

Outputs code coverage in multiple formats including HTML, LCOV, and Cobertura. Cobertura format is supported by Visual Studio Online build management tool.
