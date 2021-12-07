# install-package-analysis

install-package-analysis


## 源码

Install package programmatically. Detect package managers automatically (`npm`, `yarn` and `pnpm`).

```bash
npm i @antfu/install-pkg
```

```js
import { installPackage } from '@antfu/install-pkg'
await installPackage('vite', { silent: true })
```

## package.json

```js
{
    "name": "@antfu/install-pkg",
    "version": "0.1.0",
    "scripts": {
    "prepublishOnly": "nr build",
    "dev": "nr build --watch",
    "start": "esno src/index.ts",
    "build": "tsup src/index.ts --format cjs,esm --dts --no-splitting",
    "release": "bumpp --commit --push --tag && pnpm publish",
    "lint": "eslint \"{src,test}/**/*.ts\"",
    "lint:fix": "nr lint -- --fix"
  },
}
```

### nr

### esno

### tsup

### bumpp

```yml
- run: npm i -g pnpm @antfu/ni
- run: nci
- run: nr test --if-present
- run: npx conventional-github-releaser -p angular
```

## 安装

```js
import execa from 'execa'
import { detectPackageManager } from '.'

export interface InstallPackageOptions {
  cwd?: string
  dev?: boolean
  silent?: boolean
  packageManager?: string
  preferOffline?: boolean
  additionalArgs?: string[]
}

export async function installPackage(names: string | string[], options: InstallPackageOptions = {}) {
  const agent = options.packageManager || await detectPackageManager(options.cwd) || 'npm'
  if (!Array.isArray(names))
    names = [names]

  const args = options.additionalArgs || []

  if (options.preferOffline)
    args.unshift('--prefer-offline')

  return execa(
    agent,
    [
      agent === 'yarn'
        ? 'add'
        : 'install',
      options.dev ? '-D' : '',
      ...args,
      ...names,
    ].filter(Boolean),
    {
      stdio: options.silent ? 'ignore' : 'inherit',
      cwd: options.cwd,
    },
  )
}
```

```bash
pnpm install  -D --prefer-offine release-it react antd
```

### detectPackageManager 探测包管理器

```js
import path from 'path'
import findUp from 'find-up'

export type PackageManager = 'pnpm' | 'yarn' | 'npm'

const LOCKS: Record<string, PackageManager> = {
  'pnpm-lock.yaml': 'pnpm',
  'yarn.lock': 'yarn',
  'package-lock.json': 'npm',
}

export async function detectPackageManager(cwd = process.cwd()) {
  const result = await findUp(Object.keys(LOCKS), { cwd })
  const agent = (result ? LOCKS[path.basename(result)] : null)
  return agent
}
```

[find-up](https://github.com/sindresorhus/find-up#readme)

## 总结

整体代码比较简单。
