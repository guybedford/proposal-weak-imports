# ECMAScript proposal: Weak Imports

Champion: TBD

Status: Stage 0

## Synopsis

Syntax for weakly importing module bindings without named exports validation errors.

## Motivation

### Module Upgrade Paths

Since imported bindings are always strict, modules with changing APIs through new exports added to their interfaces require detection using the `import * as M` syntax.

For example, consider a module implementing both `oldFeature` and `newFeature`:

```js
import { oldFeature, newFeature } From 'module';
```

We would like the ability to use `oldFeature`, but only use `newFeature` if it is available, but if we use the old version of the module without the feature, an error will be thrown in the environment, meaning users instead must write:

```js
import * as M from 'module';
if (M.newFeature) // ...
```

This forces users away from the more explicit syntax of ES modules into an object pattern to support these use cases.

### Empty Modules

Consider the case where `'module'` itself was not resolved at all in the target environment (eg a missing standard library).

In this case, the fallback mechanism in the environment might in some cases want to provide an `empty` module implementation to allow the import to resolve, but just not have any features present.

The issue again, is that the empty module would have no named exports, so any package we might not resolve will require switching to the object import form.

### Web Assembly

Web Assembly needs to support these same types of conditional upgrade paths with very similar reasons to the above (see discussion at https://github.com/WebAssembly/design/issues/1281).

In the ES Module integration of Web Assembly, custom logic might be created to handle and populate weak imports.

It could be beneficial to work towards a linking-level primitive that sits at a higher level than both systems.

## Solution: Weak Imports

Weak imports allow us to continue to rely on the strong semantics of named imports, while catering for upgrade paths and empty modules in certain cases:

```js
import { feature, weak newFeature } from 'module';
```

When the import is found, it is resolved like any other binding. But if the binding was not resolved, instead of throwing an error we populate the binding as an `undefined` value that can be tested directly:

```js
import { feature, weak newFeature } from 'module';

if (newFeature) newFeature();

// *undefined* in the weak unresolved case
typeof newFeature
```

### FAQ

#### Shouldn't the weak import binding be uninitialized?

If the weak import binding were to be uninitialized, there would be no way to determine if it is defined or not, since `typeof x` for an unintialized binding will always throw.

#### Shouldn't the weak import binding be a special binding type?

We could define a new primitive value representing an empty weak import binding, and define its behaviours through the ECMAScript primitive handling, but adding a new primitive value to the language is not a process that should be taken lightly, and avoided if possible. If there were more general uses for the concept of weak bindings, this approach could certainly be explored further.

#### Will weak imports encourage looser use of ES Module semantics?

The goal of weak imports is exactly the converse - to ensure that the strong semantics of import bindings can be retained, despite variations of export shape
and availability between envrionments.

#### Why not use dynamic import?

Using dynamic import as a way around this problem is the other natural solution for feature detections. Code like:

```js
const { feature } = await import('maybe-module');
if (feature) feature();
```

The problem is that this is just as loose semantically as using`import * as M` for these use cases. In addition we're no longer able to treat the modules as direct static dependencies, and lose the live bindings properties as well.

## Specification

* [Ecmarkup source](https://github.com/guybedford/proposal-weak-imports/blob/master/spec.html)
* [HTML version](https://guybedford.github.io/proposal-weak-imports/)

## References
* [Weak Imports Issue for WebAssembly](https://github.com/WebAssembly/design/issues/1281)
