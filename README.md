# Creating a Monorepo with Lerna & Yarn Workspaces

*In this guide, you will learn how to create a Monorepo to manage multiple packages with a shared build, test, and release process.*

As applications scale, you’ll inevitably reach a point where you want to write shared reusable components which can be used everywhere. Historically, we’ve had separate repositories for each package. However, this becomes a problem for a few reasons:

* It does not scale well. Before you know it, you have dozens of different package repositories repeating the same build, test, and release process.
* It promotes bundling unnecessary components. Do we need to create a new repo for this button? Let’s put it together with this other package. Now we’ve increased the bundle size for something 95% of consumers won’t use.
* It makes upgrading difficult. If you update a base component, you now have to update its consumers, its consumer’s consumers, etc. This problem gets worse as you scale.

To make our applications as performant as possible, we need to have **small bundle sizes**. This means we should only include the code we’re using in our bundle.

Along with this, when developing shared component libraries, we want to have semver over individual pieces instead of the entire package. This prevents scenarios where:

1. Consumer A only needs the package for one component and is on v1.
2. Consumer B uses the package for all the components. They’ve helped create and modify other components in the package and it’s grown large. It’s now on v8.
3. Consumer A now needs a bug fix for the one component they use. They have to update to v8.


## Lerna

Lerna and Yarn Workspaces give us the ability to build libraries and apps in a single repo (a.k.a. Monorepo) without forcing us to publish to NPM until we are ready. This makes it faster to iterate locally when building components that depend on each other.

Lerna also provides high-level commands to optimize the management of multiple packages. For example, with one Lerna command, you can iterate through all the packages, running a series of operations (such as linting, testing, and building) on each package.

Several large JavaScript projects use monorepos including: Babel, React, Jest, Vue, Angular, and more.


## Monorepo

In this guide, we will be utilizing:
* 🐉 Lerna — The Monorepo manager
* 📦 Yarn Workspaces — Sane multi-package management
* 🚀 React — JavaScript library for user interfaces
* 💅 styled-components — CSS in JS elegance
* 🛠 Babel — Compiles next-gen JavaScript
* 📖 Storybook— UI Component Environment
* 🃏 Jest — Unit/Snapshot Testing

Okay, let’s begin! First, let’s create a new project and set up Lerna.

```javascript
$ mkdir monorepo
$ cd monorepo
$ npx lerna init
```

This creates a package.json file for your project.

```json

{
  "name": "root",
  "private": true,
  "devDependencies": {
    "lerna": "^3.13.1"
  }
}
```

You’ll notice a lerna.json file has also been created, as well as a packages folder which will contain our libraries. Let’s now modify our lerna.json file to use Yarn Workspaces. We’ll use independent versioning so we can properly enforce semver for each package.

```json
{
  "packages": ["packages/*"],
  "npmClient": "yarn",
  "useWorkspaces": true,
  "version": "independent"
}
```

We’ll also need to modify our package.json to define where the Yarn workspaces are located.

```json
{
  "name": "root",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "devDependencies": {
    "lerna": "^3.5.1"
  }
}
```

## Babel

Next, let’s add all of the dependencies we will need for Babel 7.

```shell
$ yarn add --dev -W @babel/cli @babel/core @babel/preset-react @babel/preset-env babel-core@7.0.0-bridge.0 babel-loader babel-plugin-styled-components webpack
```

Using -W instructs Yarn to install the given dependencies for the entire workspace. These dependencies are usually shared between all packages.

