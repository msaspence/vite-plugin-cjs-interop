# `vite-plugin-cjs-interop`

Vite plugin to unwrap default imports from CJS dependencies during SSR.

## Problem

In Node.js, when a CJS module that contains both a default export and named exports is imported from an ESM module, the default export has to be accessed via the `default` property of the imported module. Vite has some logic to handle this during development, but it doesn't apply to SSR builds with `external` CJS dependencies. You don't get any indication that it won't work in production. TypeScript doesn't complain either.

## Solution

This plugin will automatically unwrap the default export from CJS dependencies that you specify during SSR builds. In other words, the following code:

```js
import foo, { named, named2 as renamed } from "foo";
```

will be transformed into:

```js
const {
  default: foo = __cjsInterop1__,
  named,
  named2: renamed,
} = __cjsInterop1__?.default?.__esModule
  ? __cjsInterop1__.default
  : __cjsInterop1__;
import __cjsInterop1__ from "foo";
```

which takes care of unwrapping the default export and creating a synthetic default export if necessary.

## Installation

```sh
npm install -D vite-plugin-cjs-interop
```

## Usage

```js
// vite.config.js
import { cjsInterop } from "vite-plugin-cjs-interop";

export default {
  plugins: [
    cjsInterop({
      // List of CJS dependencies that require interop
      dependencies: [
        "some-package",
        // Deep imports should be specified separately
        "some-package/deep/import",
      ],
    }),
  ],
};
```

## License

[MIT](LICENSE)
