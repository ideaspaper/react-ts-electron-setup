# React with TypeScript and Electron Setup

## Initializing Project

```
npx create-react-app my-app --template typescript
```

**References**

- [Create React App - Adding TypeScript](https://create-react-app.dev/docs/adding-typescript/)
- [npm - scripts](https://docs.npmjs.com/cli/v9/using-npm/scripts)

## Electron

Electron is a framework for building desktop applications using JavaScript, HTML, and CSS. By embedding Chromium and Node.js into its binary, Electron allows you to maintain one JavaScript codebase and create cross-platform apps that work on Windows, macOS, and Linux — no native development experience required.

```
npm i @types/electron-devtools-installer electron-devtools-installer electron-is-dev electron-reload
npm i -D concurrently electron electron-builder wait-on cross-env
```

`electron/tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "sourceMap": true,
    "strict": true,
    "outDir": "../build",
    "rootDir": "../",
    "noEmitOnError": true,
    "typeRoots": ["node_modules/@types"]
  }
}
```

`electron/main.ts`

```ts
import { app, BrowserWindow } from 'electron';
import * as path from 'path';
import * as isDev from 'electron-is-dev';
import installExtension, {
  REACT_DEVELOPER_TOOLS,
} from 'electron-devtools-installer';

let win: BrowserWindow | null = null;

function createWindow() {
  win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  });

  if (isDev) {
    win.loadURL('http://localhost:3000/index.html');
  } else {
    // 'build/index.html'
    win.loadURL(`file://${__dirname}/../index.html`);
  }

  win.on('closed', () => (win = null));

  // Hot Reloading
  if (isDev) {
    // 'node_modules/.bin/electronPath'
    require('electron-reload')(__dirname, {
      electron: path.join(
        __dirname,
        '..',
        '..',
        'node_modules',
        '.bin',
        'electron',
      ),
      forceHardReset: true,
      hardResetMethod: 'exit',
    });
  }

  // DevTools
  installExtension(REACT_DEVELOPER_TOOLS)
    .then((name) => console.log(`Added Extension:  ${name}`))
    .catch((err) => console.log('An error occurred: ', err));

  if (isDev) {
    win.webContents.openDevTools();
  }
}

app.on('ready', createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (win === null) {
    createWindow();
  }
});
```

`package.json`

```json
{
  ...
  "author": "Your name",
  "description": "Your project name",
  ...
  "homepage": ".",
  "main": "build/electron/main.js",
  ...
  "scripts": {
    ...
    "postinstall": "electron-builder install-app-deps",
    "electron:dev": "concurrently \"cross-env BROWSER=none npm start\" \"wait-on http://127.0.0.1:3000 && tsc -p electron -w\" \"wait-on http://127.0.0.1:3000 && tsc -p electron && electron .\"",
    "electron:build": "npm run build && tsc -p electron && electron-builder",
    ...
  },
  ...
  "build": {
    "extends": null,
    "files": [
      "build/**/*"
    ],
    "directories": {
      "buildResources": "assets"
    }
  },
  ...
}
```

**References**

- [Electron](https://www.electronjs.org/docs/latest/)
- [react-typescript-electron-sample-with-create-react-app-and-electron-builder](https://github.com/yhirose/react-typescript-electron-sample-with-create-react-app-and-electron-builder)

## ESLint

ESLint is a tool for identifying and reporting on patterns found in ECMAScript/JavaScript code.

```
npm i -D eslint
```

`.eslintrc.js`

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:jsx-a11y/recommended',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['react', '@typescript-eslint', 'jsx-a11y'],
  rules: {
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-empty-function': 'off',
    '@typescript-eslint/ban-ts-comment': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off', // !
    '@typescript-eslint/no-extra-non-null-assertion': 'off', // !!
    '@typescript-eslint/explicit-module-boundary-types': 'error',
    'no-async-promise-executor': 'off',
    'no-extra-boolean-cast': 'off',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

`.eslintignore`

```
# don't ever lint node_modules
node_modules
node_modules/*

# don't lint build output (make sure it's set to your correct build folder name)
dist
build
electron

# don't lint nyc coverage output
coverage
src/serviceWorker.ts
```

`package.json`

```json
...
  "scripts": {
    ...
    "lint:fix": "eslint '*/**/*.{js,ts,tsx}' --fix",
    "lint": "eslint '*/**/*.{js,ts,tsx}'"
  },
...
```

**References**

- [npm - eslint](https://www.npmjs.com/package/eslint)

## Prettier

Prettier is an opinionated code formatter. It enforces a consistent style by parsing your code and re-printing it with its own rules that take the maximum line length into account, wrapping code when necessary.

```
npm i -D prettier
```

`.prettierrc`

```json
{
  "printWidth": 80,
  "singleQuote": true,
  "trailingComma": "all"
}
```

`.prettierignore`

```
build
dist
coverage
```

`package.json`

```json
...
  "scripts": {
    ...
    "tidy": "prettier '*/**/*.{js,ts,tsx,json,md,html}' --write"
  },
...
```

**References**

- [npm - prettier](https://www.npmjs.com/package/prettier)

## ESLint vs Prettier

[Source](https://blog.logrocket.com/using-prettier-eslint-automate-formatting-fixing-javascript/#managing-eslint-rules-avoid-conflict-prettier)

## lint-staged

Run linters **against staged Git files** and don't let 💩 slip into your code base!

```
npm i -D lint-staged
```

`package.json`

```json
...
  "lint-staged": {
    "*.{js,ts,tsx,json,md,html}": [
      "prettier --write"
    ]
  }
...
```

**References**

- [npm - lint-staged](https://www.npmjs.com/package/lint-staged)

## Husky

Husky is a package that allows custom scripts to be ran against your Git repository. These scripts trigger actions in response to specific events, so they can help you automate your development lifecycle.

```
npm i -D husky
npm pkg set scripts.prepare="husky install"
npm run prepare
npx husky add .husky/pre-commit "npx lint-staged && npm run lint && npm run test -- --watchAll=false --passWithNoTests"
```

`.husky/pre-commit`

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged && npm run lint && npm run test -- --watchAll=false --passWithNoTests
```

**References**

- [npm - husky](https://www.npmjs.com/package/husky)
- [git - Customizing Git - Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
- [Altlassian - Git Hooks](https://www.atlassian.com/git/tutorials/git-hooks)
