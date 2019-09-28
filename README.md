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

Since yarn has run, you have a /node_modules folder. We don’t want to commit any of these packages, so let’s add a .gitignore.

```json
.log
.DS_Store
.jest-*
lib
node_modules
```

Okay, back to Babel. To set up the global configuration for Babel, we’ll need a babel.config.js file in the root of the repository.

```json
module.exports = {
    plugins: ['babel-plugin-styled-components'],
    presets: ['@babel/preset-env', '@babel/preset-react']
};
```

This file tells Babel how to compile our packages. Now, let’s create a script to execute Babel. We’ll add this to our package.json.

```javascript
"scripts": {
    "build": "lerna exec --parallel -- babel --root-mode upward src -d lib --ignore **/*.story.js,**/*.spec.js"
}
```

Let’s dissect this command. lerna exec will take any command and run it over all of the different packages. This command instructs Babel to run in parallel over every package, pulling from the /src folder and compiling into the /lib folder. We don’t want to include any tests or stories (which we’ll get to later) in the compiled output.

Using --root-mode upward is the special sauce to using Yarn workspaces. This tells Babel the node_modules are located in the root instead of nested inside each of the individual packages. This prevents each package from having the same node_modules and extracts them up to the root. We’ll be utilizing a similar approach for testing later.

## React

We have completed the infrastructure for a Monorepo. Let’s create some packages for it to consume. We’ll be using React and styled-components to develop our UI components, so let’s install those first.

```javascript
$ yarn add --dev -W react react-dom styled-components
```

Then, inside of /packages let's create a folder called /button and set up our first package.

```json
{
    "name": "button",
    "version": "1.0.0",
    "main": "lib/index.js",
    "module": "src/index.js",
    "dependencies": {
        "react": "16.8.5",
        "react-dom": "16.8.5",
        "styled-components": "4.1.3"
    },
    "peerDependencies": {
        "react": "^15.0.0 || ^16.0.0",
        "react-dom": "^15.0.0 || ^16.0.0",
        "styled-components": "^4.0.0"
    }
}
```

This file informs consumers the module will live inside the /src folder and the output ran through Babel (main) will live inside /lib. This will be the main entry point into the package. Listing the peerDependencies helps ensure consumers are including the correct packages.

We’ll also want to link our root dependencies to our newly created package. Let’s create a script to do this inside our package.json.

```shell
"scripts": {
  "bootstrap": "lerna bootstrap --use-workspaces"
}
```
Now we can simply run yarn bootstrap to install and link all dependencies.

Okay, now let’s create our first component:Button.

```javascript
import styled from 'styled-components';

const Button = styled.button`
    background: red;
    color: #fff;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    font-weight: 300;
    padding: 9px 36px;
`;

export default Button;
```

Let’s test if Babel is configured properly. We should be able to run yarn build and see a /lib folder created for our new package.
