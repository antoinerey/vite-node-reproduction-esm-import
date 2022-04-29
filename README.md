# Reproduction of an issue where vite-node fails when importing ESM dependency.

Upgrading from `vite-node@0.8.5` to any higher version makes this project to fail. I believe that the turning point has been this commit ([c4a25ae](https://github.com/vitest-dev/vitest/commit/c4a25ae)), where some checks to inline dependencies were removed.

```
System:
  OS: macOS 12.0.1
  CPU: (8) x64 Intel(R) Core(TM) i7-8557U CPU @ 1.70GHz
  Memory: 104.10 MB / 16.00 GB
  Shell: 5.8 - /bin/zsh
Binaries:
  Node: 18.0.0 - /usr/local/bin/node
  Yarn: 1.22.17 - /usr/local/bin/yarn
  npm: 8.6.0 - /usr/local/bin/npm
  Watchman: 2022.03.21.00 - /usr/local/bin/watchman
Browsers:
  Brave Browser: 100.1.37.116
  Chrome: 101.0.4951.41
  Firefox: 98.0.1
  Safari: 15.1
npmPackages:
  vite-node: ^0.10.0 => 0.10.0
```

## Steps to reproduce

1. Clone the repository.
2. Install the dependencies (running `yarn`).
3. Execute `yarn start` (or `vite-node index.ts`).

## Expected behaviour

The script is expected to log the content of the `rest` variable, imported from the `msw` package. Something like the following should appear in the terminal:

```
yarn run v1.22.17
$ vite-node index.ts
{
  all: [Function (anonymous)],
  head: [Function (anonymous)],
  get: [Function (anonymous)],
  post: [Function (anonymous)],
  put: [Function (anonymous)],
  delete: [Function (anonymous)],
  patch: [Function (anonymous)],
  options: [Function (anonymous)]
}
✨  Done in 0.78s.
```

## Current behaviour

The latest version of `vite-node` fails to run the script. It tries to import the ESM version of the `msw` package (even though an UMD version is available), and crashes on the `export` syntax. Here's the error that appears in the terminal:

```
yarn run v1.22.17
$ vite-node index.ts
(node:76797) Warning: To load an ES module, set "type": "module" in the package.json or use the .mjs extension.
(Use `node --trace-warnings ...` to show where the warning was created)
/Users/antoine/Code/vitest-reproduction-fail-esm/node_modules/msw/lib/esm/index.js:1
export { i as context } from './index-deps.js';
^^^^^^

SyntaxError: Unexpected token 'export'
    at Object.compileFunction (node:vm:352:18)
    at wrapSafe (node:internal/modules/cjs/loader:1031:15)
    at Module._compile (node:internal/modules/cjs/loader:1065:27)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at ModuleWrap.<anonymous> (node:internal/modules/esm/translators:190:29)
    at ModuleJob.run (node:internal/modules/esm/module_job:185:25)
    at async Promise.all (index 0)
    at async ESMLoader.import (node:internal/modules/esm/loader:281:24)
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

## Notes

- This reproduction tries to import some function from the `msw` package because I discovered the issue while working with it. However, I would say it's not linked to the `msw` package in any way.
