# rollup.config.ts

## File Metadata
- **Path**: `packages/react/rollup.config.ts`
- **Type**: Text file
- **Size**: 3,670 bytes (142 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';
import sourceMaps from 'rollup-plugin-sourcemaps';
import typescript from 'rollup-plugin-typescript2';
import replace from 'rollup-plugin-replace';
import json from 'rollup-plugin-json';
import { terser } from 'rollup-plugin-terser';

import pkg from './package.json';

const libraryName = 'builder-react';

const resolvePlugin = resolve();

const externalDependencies = Object.keys(pkg.dependencies)
  .concat(Object.keys(pkg.optionalDependencies || {}))
  .filter(item => item !== 'tslib')
  .concat(Object.keys(pkg.peerDependencies || {}));

const options = {
  input: `src/${libraryName}.ts`,
  // Indicate here external modules you don't wanna include in your bundle (i.e.: 'lodash')
  external: ['isolated-vm'],
  watch: {
    include: '../**',
  },
  plugins: [
    typescript({
      include: ['*.js+(|x)', '*.ts+(|x)', '**/*.ts+(|x)'],
      tsconfigOverride: {
        compilerOptions: {
          // No need to type check and gen over and over, we do once at beginning of builder with `tsc`
          declaration: false,
          checkJs: false,
          allowJs: true,
        },
      },
    }),
    replace({
      'process.env.NODE_ENV': JSON.stringify('production'),
    }),
    // Allow json resolution
    json(),
    // Compile TypeScript files
    // Allow bundling cjs modules (unlike webpack, rollup doesn't understand cjs)
    commonjs({
      exclude: ['node_modules/isolated-vm/**'],
      namedExports: {
        'node_modules/react/index.js': [
          'cloneElement',
          'createContext',
          'useContext',
          'Component',
          'createElement',
          'forwardRef',
          'Fragment',
          'useEffect',
          'useState',
        ],
        'node_modules/react-dom/index.js': ['render', 'hydrate'],
        'node_modules/react-is/index.js': ['isElement', 'isValidElementType', 'ForwardRef'],
      },
    }),
    // Allow node_modules resolution, so you can use 'external' to control
    // which external modules to include in the bundle
    // https://github.com/rollup/rollup-plugin-node-resolve#usage
    // resolve({}),
    resolvePlugin,

    // Resolve source maps to the original source
    sourceMaps(),

    terser(),
  ],
};

export default [
  // UMD browser build
  {
    ...options,
    output: {
      file: 'dist/builder-react.browser.js',
      name: 'BuilderReact',
      format: 'umd',
      sourcemap: true,
      amd: {
        id: '@builder.io/react',
      },
    },
  },
  // Main ES and CJS builds
  {
    ...options,
    output: [
      { file: pkg.module, format: 'es', sourcemap: true },
      { file: pkg.main, format: 'cjs', sourcemap: true },
    ],
    external: externalDependencies.concat('node-fetch'),
    plugins: options.plugins
      .filter(plugin => plugin !== resolvePlugin)
      .concat([
        resolve({
          only: [/^\.{0,2}\//],
        }),
      ]),
  },
  // Lite builds
  {
    ...options,
    input: `src/${libraryName}-lite.ts`,
    output: [
      {
        file: `dist/${libraryName}-lite.esm.js`,
        format: 'es',
        sourcemap: true,
      },
      {
        file: `dist/${libraryName}-lite.cjs.js`,
        format: 'cjs',
        sourcemap: true,
      },
    ],
    external: externalDependencies,
    plugins: options.plugins.map(plugin =>
      plugin !== resolvePlugin
        ? plugin
        : resolve({
            only: [/^\.{0,2}\//],
          })
    ),
  },
  // iife build
  {
    ...options,
    output: {
      file: pkg.unpkg,
      format: 'iife',
      name: 'BuilderReact',
      sourcemap: true,
    },
  },
];

```


## High-Level Overview
Indicate here external modules you don't wanna include in your bundle (i.e.: 'lodash')
No need to type check and gen over and over, we do once at beginning of builder with `tsc`
Allow json resolution
Compile TypeScript files
Allow bundling cjs modules (unlike webpack, rollup doesn't understand cjs)

## Detailed Walkthrough


### Imports (8)

- `rollup-plugin-sourcemaps`
- `rollup-plugin-replace`
- `rollup-plugin-node-resolve`
- `rollup-plugin-typescript2`
- `rollup-plugin-commonjs`
- `rollup-plugin-json`
- `./package.json`
- `rollup-plugin-terser`

### Exports (1)

- `default`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `rollup-plugin-sourcemaps` (external)
- `rollup-plugin-replace` (external)
- `rollup-plugin-node-resolve` (external)
- `rollup-plugin-typescript2` (external)
- `rollup-plugin-commonjs` (external)
- `rollup-plugin-json` (external)
- `./package.json` (local)
- `rollup-plugin-terser` (external)

## Performance and Security Notes
🔒 **Security**: Uses environment variables - ensure secrets are not exposed

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
