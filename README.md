# karma-parallel-2 (karma-parallel)

[![npm version](https://img.shields.io/npm/v/karma-parallel.svg?style=flat-square)](https://www.npmjs.com/package/karma-parallel)
[![npm downloads](https://img.shields.io/npm/dm/karma-parallel.svg?style=flat-square)](https://www.npmjs.com/package/karma-parallel)
[![Build Status](https://travis-ci.org/joeljeske/karma-parallel.svg?branch=master)](https://travis-ci.org/joeljeske/karma-parallel)
[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/joeljeske/karma-parallel)
[![dependencies Status](https://david-dm.org/joeljeske/karma-parallel/status.svg)](https://david-dm.org/joeljeske/karma-parallel)
[![devDependencies Status](https://david-dm.org/joeljeske/karma-parallel/dev-status.svg)](https://david-dm.org/joeljeske/karma-parallel?type=dev)

> A Karma JS plugin to support sharding tests to run in parallel across multiple browsers. Now supporting code coverage! *This temporarily replaces karma-parallel to fix an issue while using karma within an IDE*

# Overview

This is intended to speed up the time it takes to run unit tests by taking advantage of multiple cores. From a single
karma server, multiple instances of a browser are spun up. Each browser downloads all of the spec files, but when a
`describe` block is encountered, the browsers deterministically decide if that block should run in the given browser.

This leads to a way to split up unit tests across multiple browsers without changing any build processes.

## Installation

The easiest way is to install `karma-parallel` as a `devDependency`.

**Using NPM**

```bash
npm i karma-parallel --save-dev
```

**Using Yarn**

```bash
yarn add karma-parallel --dev
```


## Examples

### Basic Installation

```javascript
// karma.conf.js
module.exports = function(config) {
  config.set({
    // NOTE: 'parallel' must be the first framework in the list
    frameworks: ['parallel', 'mocha' /* or 'jasmine' */],
  });
};
```

### Additional Configuration

```javascript
// karma.conf.js
module.exports = function(config) {
  config.set({
    // NOTE: 'parallel' must be the first framework in the list
    frameworks: ['parallel', 'mocha' /* or 'jasmine' */],
    plugins: [
        // add karma-parallel to the plugins if you encounter something like "karma parallel No provider for framework:parallel"
        require('karma-parallel'),
        ...
    ],
    parallelOptions: {
      executors: 4, // Defaults to cpu-count - 1
      shardStrategy: 'round-robin'
      // shardStrategy: 'description-length'
      // shardStrategy: 'custom'
      // customShardStrategy: function(config) {
      //   config.executors // number, the executors set above
      //   config.shardIndex // number, the specific index for the shard currently running
      //   config.description // string, the name of the top-level describe string. Useful //     for determining how to shard the current specs
      //
      //   // Re-implement a round-robin strategy
      //   window.parallelDescribeCount = window.parallelDescribeCount || 0;
      //   window.parallelDescribeCount++;
      //   return window.parallelDescribeCount % config.executors === config.shardIndex
      // }
      }
    }
  });
};
```


## Options

`parallelOptions [object]`: Options for this plugin

`parallelOptions.executors [int=cpu_cores-1]`: The number of browser instances to
use to test. If you test on multiple types of browsers, this spin up the number of
executors for each browser type.

`parallelOptions.shardStrategy [string='round-robin']`: This plugin works by
overriding the test suite `describe()` function. When it encounters a describe, it
must decide if it will skip the tests inside of it, or not.

* The `round-robin` style will only take every `executors` test suite and skip the ones in between.
* The `description-length` deterministically checks the length of the description for each test suite use a modulo of the number of executors.
* The `custom` allows you to use a custom function that will determine if a describe block should run in the current executor. It is a function that is serialized and re-constructed on each executor. The function will be called for every top level describe block and should return true if the describe block should run for a the current executor. The function is called with an object containing 3 properties; `executors` the total number of executors, `shardIndex` the 0-based index of the current executor, and the `description` the string passed to the describe block (useful for gaining context of the current description). You can create complex and custom behaviors for grouping specs together based on your needs. Note: multiple instances function of this function will exist; one for each spec, this means you cannot rely on a global state inside this function. Note: this function is serialized and re-created. This means you cannot use **any** closure variables. You must only reference parameters to this function (or globals that you may have setup outside of karma-parallel in your spec files.)

`parallelOptions.aggregatedReporterTest [(reporter)=>boolean|regex=/coverage|istanbul|junit/i]`: This is an
optional regex or function used to determine if a reporter needs to only received aggregated events from the browser shards. It is used to ensure coverage reporting is accurate amongst all the shards of a browser. It is also useful for some programmatic reporters such as junit reporters that need to operate on a single set of test outputs and not once for each shard. Set to null to disable aggregated reporting.


## Important Notes

**Why are there extra tests in my output?**

If this plugin discovers that you have focused some tests (fit, it.only, etc...) in other browser instances, it will add an extra focused test in the current browser instance to limit the running of the tests in the given browser. Similarly, when dividing up the tests, if there are not enough tests for a given browser, it will add an extra test to prevent karma from failing due to no running tests.

**Run Some Tests in Every Browser**

If you want to run some tests in every browser instance, add the string `[always]` at the beginning of the top-level describe block that contains those tests. Example:

```
describe('[always] A suite that runs on every shard', function() {
    // Define it() blocks here
});
```

**Code Coverage**

Code coverage support is acheived by aggregating the code coverage reports from each browser into a single coverage report. We accomplish this by wrapping the coverage reporters with an aggregate reporter. By default, we only wrap reporters that pass the test `parallelOptions.aggregatedReporterTest`. It should all *just work*.

----

For more information on Karma see the [homepage].

## See Also

[`karma-sharding`](https://github.com/rschuft/karma-sharding)

This similar project works by splitting up the actual spec files across the browser instances.

Pros:

* Reduces memory by only loading *some* of the spec files in each browser instance

Cons:

* Requires the spec files to reside in separate files, meaning it is not compatible with bundlers such
as [`karma-webpack`](https://github.com/webpack-contrib/karma-webpack) or [`karma-browserify`](https://github.com/nikku/karma-browserify) as used with most front end cli projects (e.g. @angular/cli)



[homepage]: http://karma-runner.github.com

