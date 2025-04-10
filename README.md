# Dart Sass for Meteor.js

This is a build plugin for Meteor.js that compiles Sass files using Dart Sass.

> [!IMPORTANT]
> As of `v5.0.0`, this package **migrated from LibSass to Dart Sass**.
> If you are upgrading from an earlier version, please read the [Migration Guide](https://github.com/illusionfield/fourseven-scss/blob/master/MIGRATION-GUIDE-v5.md), as this README does not cover all necessary migration steps.

## What's New in `v5.1.0`

> [!NOTE]
> Starting from `v5.1.0`, this package **no longer declares **``** as a direct dependency**. This gives you the freedom to install your preferred `sass` or `sass-embedded` version via `npm`.

For example:

```sh
meteor npm install --save-dev sass
```

or

```sh
meteor npm install --save-dev sass-embedded
```

This gives you flexibility to:

- pin specific Sass versions (e.g., to avoid `@import` deprecation warnings),
- use faster alternatives like [`sass-embedded`](https://www.npmjs.com/package/sass-embedded),
- or switch implementations without modifying the plugin.

## Installation

Install via Meteor's package system:

```sh
meteor add fourseven:scss
meteor npm install --save-dev sass
```

If you're using this in a Meteor package, add it to your `onUse` block:

```js
Package.onUse((api) => {
  ...
  api.use('fourseven:scss');
  ...
});
```

## Compatibility

| Meteor Version | Recommended `fourseven:scss` Version |
| -------------- | ------------------------------------ |
| 1.0 - 1.1      | 3.2.0                                |
| 1.2 - 1.3.1    | 3.4.2                                |
| 1.3.2+         | 3.8.0\_1                             |
| 1.4.0          | 3.8.1                                |
| 1.4.1+         | 4.5.4                                |
| 1.6+           | 4.12.0                               |
| 2.10+          | 5.0.0+                               |

## Usage

After installation, this package will automatically detect all `.scss` and `.sass` files in your project, compile them using [Dart Sass](https://www.npmjs.com/package/sass), and include the resulting CSS in the client bundle. Files can be located anywhere in your project.

### File Types

- **Source files**: Standard `*.scss` and `*.sass` files that are compiled.
- **Imports/Partials**: Files prefixed with `_`, or those marked as `isImport: true` in `package.js`:

```js
api.addFiles('x.scss', 'client', { isImport: true });
```

Each compiled source file produces CSS, which is merged using Meteor’s `standard-minifiers`.

### Importing Styles

You can import styles from:

- **Other Meteor packages**
- **Your app**
- **npm modules**
- **Outside the Meteor project** (e.g., absolute paths or symlinked folders — with the limitation that Meteor's file watcher will not detect changes in these files automatically)

#### From another package

```scss
@use "meteor:{my-package:pretty-buttons}/buttons/styles" as buttons; // Assigns a namespace "buttons"

.my-button {
  @extend buttons.pretty-button; // Uses the imported class with its namespace
}
```

or

```scss
@use "meteor:{my-package:pretty-buttons}/buttons/styles" as *; // Imports everything into the global scope

.my-button {
  @extend .pretty-button; // No namespace required
}
```

#### From your app

```scss
@use "{}/client/styles/imports/colors.scss" as *;

.my-nav {
  background-color: @primary-branding-color; // Use a color from the app's style palette
}
```

#### From npm modules

```scss
@use "~module-name/stylesheet"; // Imports a module from node_modules
```

> [!TIP]
> If the target is a directory, it will search for an `index.scss`, `_index.scss`, `index.sass`, or `_index.sass` file inside that directory.

## Custom Configuration

Create a `.scss.config.json` file at the root of your project to customize compiler behavior.

### Supported Options

- `quietDeps` — suppress dependency warnings
- `verbose` — show detailed compiler output
- `includePaths` — set custom import paths

```json
{
  "quietDeps": true,
  "verbose": false,
  "includePaths": [
    "{}/some/very/long/path/for/holding/my/mixins/folder"
  ]
}
```

### Compile options

Currently, this file supports only the `quietDeps` and `verbose` options, but more options will be available in the future.
Learn more about the available options in the [Sass compiler options](https://sass-lang.com/documentation/js-api/interfaces/options).

### Global include path

When working with long and complex project structures,
specifying the full path to a (e.g. mixin) library in every SCSS file can be cumbersome.
Configuring a global `includePaths` option allows you to define shorter, more maintainable import paths.

Additionally, if the directory structure of the libraries changes,
you only need to update the configuration file once instead of modifying multiple SCSS files.

```scss
@use "mixins"; // This file is actually placed in another directory!
```

> [!NOTE]
> This plugin does not use [Sass’s built-in loadPaths](https://sass-lang.com/documentation/js-api/interfaces/options/#loadPaths) option. Instead, it implements a custom method tailored to the Meteor environment, ensuring seamless integration with Meteor’s build system and package resolution.

## CSS Output Features

### Source Maps

Enabled by default for better debugging.

### Autoprefixer Support

This package supports autoprefixing via PostCSS, if used with Meteor 2.10+ and plugin `v5.0.0+`.

```sh
meteor add standard-minifier-css
meteor npm install --save-dev autoprefixer postcss postcss-load-config
```

For details, refer to the [Meteor CSS minifier docs](https://docs.meteor.com/packages/standard-minifier-css.html).

## Using `sass-embedded` (Optional)

From `v5.1.0`, you may use [`sass-embedded`](https://www.npmjs.com/package/sass-embedded`) for faster builds.

### Benefits

- 🚀 **Performance**:
  Dart Sass Embedded runs as a native executable, and communicates with Node.js through a binary protocol.
  This makes it **significantly faster** than the standard JavaScript-based `sass` package.
  ([source](https://sass-lang.com/blog/embedded-host-protocol))

- 📦 **Decoupled from Node.js internals**:
  Unlike the JS-only version, `sass-embedded` relies on the Dart SDK and does not require a JS-based implementation of the Sass compiler, resulting in better long-term maintainability.

- 🔧 **More control**:
  It enables advanced use-cases where developers want full control over the Sass engine, its lifecycle, or version pinning.

> [!NOTE]
> `sass-embedded` is **not cross-platform by default**. It installs binaries for the current platform only via [`optionalDependencies`](https://github.com/sass/embedded-host-node/blob/main/package.json).
> Because of this, it cannot be bundled directly in a Meteor package for all architectures.
> For more details, see the [official GitHub repo](https://github.com/sass/embedded-host-node).

## Debug Mode

To troubleshoot issues, enable debug mode:

```sh
export DEBUG_PACKAGE_SASS=true
```

This prints detailed logs and stack traces.
