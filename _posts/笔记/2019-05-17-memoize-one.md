---
layout: note
title: memoize-one
category: 笔记
tags: npm
keywords: 笔记,2019,npm,memoize-one
description: 
---

`引用文章来源:https://www.npmjs.com/package/memoize-one`

# memoize-one

<a name="1sdva"></a>
# memoize-one
A memoization library that only caches the result of the most recent arguments.

<a name="obB53"></a>
## [](https://www.npmjs.com/package/memoize-one#rationale)Rationale
Unlike other memoization libraries, `memoize-one` only remembers the latest arguments and result. No need to worry about cache busting mechanisms such as `maxAge`, `maxSize`, `exclusions` and so on which can be prone to memory leaks. `memoize-one` simply remembers the last arguments, and if the function is next called with the same arguments then it returns the previous result.
<a name="sKZa1"></a>
## [](https://www.npmjs.com/package/memoize-one#usage)Usage
<a name="1nmV1"></a>
### [](https://www.npmjs.com/package/memoize-one#standard-usage)Standard usage

```javascript
import memoizeOne from 'memoize-one';

const add = (a, b) => a + b;
const memoizedAdd = memoizeOne(add);

memoizedAdd(1, 2); // 3
memoizedAdd(1, 2); // 3
// Add function is not executed: previous result is returned
memoizedAdd(2, 3); // 5
// Add function is called to get new value
memoizedAdd(2, 3); // 5
// Add function is not executed: previous result is returned
memoizedAdd(1, 2); // 3
// Add function is called to get new value.
// While this was previously cached,
// it is not the latest so the cached result is lost
```

<a name="2FmCK"></a>
## [](https://www.npmjs.com/package/memoize-one#installation)Installation
```shell
# yarn 
yarn add memoize-one
# npm 
npm install memoize-one --save
```

<a name="dYR51"></a>
## Module usage
<a name="ExmaZ"></a>
### [](https://www.npmjs.com/package/memoize-one#es6-module)ES6 module
> **import** memoizeOne **from** 'memoize-one';

<a name="NZB5Z"></a>
### [](https://www.npmjs.com/package/memoize-one#commonjs)CommonJS
If you are in a CommonJS environment (eg [Node](https://nodejs.org/)), then **you will need to add `.default` to your import**:
> const memoizeOne **=** require('memoize-one').default;

<a name="PW7EM"></a>
## [](https://www.npmjs.com/package/memoize-one#custom-equality-function)Custom equality function
You can also pass in a custom function for checking the equality of two sets of arguments
> const memoized **=** memoizeOne(fn, isEqual);
> type EqualityFn **=** (newArgs**:** mixed[], oldArgs**:** mixed[]) => boolean;

An equality function should return `true` if the arguments are equal. If `true` is returned then the wrapped function will not be called.<br />The default equality function is a shallow equal check of all arguments (each argument is compared with `===`). If the `length` of arguments change, then the default equality function makes no shallow equality checks. You are welcome to decide if you want to return `false` if the `length` of the arguments is not equal

```javascript
const simpleIsEqual: EqualityFn = (
  newArgs: mixed[],
  lastArgs: mixed[],
): boolean =>
  newArgs.length === lastArgs.length &&
  newArgs.every(
    (newArg: mixed, index: number): boolean =>
      shallowEqual(newArg, lastArgs[index]),
  );
```

A custom equality function needs to compare `Arrays`. The `newArgs` array will be a new reference every time so a simple `newArgs === lastArgs` will always return `false`.<br />Equality functions are not called if the `this` context of the function has changed (see below).<br />Here is an example that uses a `lodash.isequal` deep equal equality check
> `lodash.isequal` correctly handles deep comparing two arrays
> 

```javascript
import memoizeOne from 'memoize-one';
import isDeepEqual from 'lodash.isequal';

const identity = x => x;

const shallowMemoized = memoizeOne(identity);
const deepMemoized = memoizeOne(identity, isDeepEqual);

const result1 = shallowMemoized({ foo: 'bar' });
const result2 = shallowMemoized({ foo: 'bar' });

result1 === result2; // false - difference reference

const result3 = deepMemoized({ foo: 'bar' });
const result4 = deepMemoized({ foo: 'bar' });

result3 === result4; // true - arguments are deep equal
```


<a name="AvAr2"></a>
## [](https://www.npmjs.com/package/memoize-one#this)`this`
<a name="vX375"></a>
### [](https://www.npmjs.com/package/memoize-one#memoize-one-correctly-respects-this-control)`memoize-one` correctly respects `this` control
This library takes special care to maintain, and allow control over the the `this` context for **both**the original function being memoized as well as the returned memoized function. Both the original function and the memoized function's `this` context respect [all the `this` controlling techniques](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md):

- new bindings (`new`)
- explicit binding (`call`, `apply`, `bind`);
- implicit binding (call site: `obj.foo()`);
- default binding (`window` or `undefined` in `strict mode`);
- fat arrow binding (binding to lexical `this`)
- ignored this (pass `null` as `this` to explicit binding)
<a name="yZtqe"></a>
### [](https://www.npmjs.com/package/memoize-one#changes-to-this-is-considered-an-argument-change)Changes to `this` is considered an argument change
Changes to the running context (`this`) of a function can result in the function returning a different value even though its arguments have stayed the same:

```javascript
function getA() {
  return this.a;
}

const temp1 = {
  a: 20,
};
const temp2 = {
  a: 30,
};

getA.call(temp1); // 20
getA.call(temp2); // 30
```

Therefore, in order to prevent against unexpected results, `memoize-one` takes into account the current execution context (`this`) of the memoized function. If `this` is different to the previous invocation then it is considered a change in argument. [further discussion](https://github.com/alexreardon/memoize-one/issues/3).<br />Generally this will be of no impact if you are not explicity controlling the `this` context of functions you want to memoize with [explicit binding](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#explicit-binding) or [implicit binding](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#implicit-binding). `memoize-One` will detect when you are manipulating `this` and will then consider the `this` context as an argument. If `this` changes, it will re-execute the original function even if the arguments have not changed.
<a name="Rqj1c"></a>
## [](https://www.npmjs.com/package/memoize-one#when-your-result-function-throw-s)When your result function `throw`s
> There is no caching when your result function throws

If your result function `throw`s then the memoized function will also throw. The throw will not break the memoized functions existing argument cache. It means the memoized function will pretend like it was never called with arguments that made it `throw`.

```javascript
const canThrow = (name: string) => {
  console.log('called');
  if (name === 'throw') {
    throw new Error(name);
  }
  return { name };
};

const memoized = memoizeOne(canThrow);

const value1 = memoized('Alex');
// console.log => 'called'
const value2 = memoized('Alex');
// result function not called

console.log(value1 === value2);
// console.log => true

try {
  memoized('throw');
  // console.log => 'called'
} catch (e) {
  firstError = e;
}

try {
  memoized('throw');
  // console.log => 'called'
  // the result function was called again even though it was called twice
  // with the 'throw' string
} catch (e) {
  secondError = e;
}

console.log(firstError !== secondError);

const value3 = memoized('Alex');
// result function not called as the original memoization cache has not been busted
console.log(value1 === value3);
// console.log => true
```


<a name="XPjZx"></a>
## [](https://www.npmjs.com/package/memoize-one#performance-rocket)Performance 🚀
<a name="L4HaK"></a>
### [](https://www.npmjs.com/package/memoize-one#tiny)Tiny
`memoize-one` is super lightweight at [![](https://img.shields.io/bundlephobia/min/memoize-one.svg?label=#align=left&display=inline&height=20&originHeight=20&originWidth=43&status=done&width=43)](https://www.npmjs.com/package/memoize-one) minified and [![](https://img.shields.io/bundlephobia/minzip/memoize-one.svg?label=#align=left&display=inline&height=20&originHeight=20&originWidth=43&status=done&width=43)](https://www.npmjs.com/package/memoize-one) gzipped. (`1KB` = `1,024 Bytes`)
<a name="VC0AX"></a>
### [](https://www.npmjs.com/package/memoize-one#extremely-fast)Extremely fast
`memoize-one` performs better or on par with than other popular memoization libraries for the purpose of remembering the latest invocation.<br />**Results**

- [simple arguments](https://www.measurethat.net/Benchmarks/ShowResult/4452)
- [complex arguments](https://www.measurethat.net/Benchmarks/ShowResult/4488)

The comparisons are not exhaustive and are primarily to show that `memoize-one`accomplishes remembering the latest invocation really fast. The benchmarks do not take into account the differences in feature sets, library sizes, parse time, and so on.
<a name="nNhzQ"></a>
## [](https://www.npmjs.com/package/memoize-one#code-health-thumbsup)Code health 👍

- Tested with all built in [JavaScript types](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20%26%20grammar/ch1.md).
- 100% code coverage
- [Continuous integration](https://travis-ci.org/alexreardon/memoize-one) to run tests and type checks.
- [`Flow` types](http://flowtype.org/) for safer internal execution and external consumption. Also allows for editor autocompletion.
- Follows [Semantic versioning (2.0)](http://semver.org/) for safer consumption.
- No dependencies


