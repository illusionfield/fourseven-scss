# Dart Sass for Meteor.js

This is a build plugin for Meteor.js that compiles Sass files using Dart Sass.

## Before you begin

> [!IMPORTANT]
> In `v5.0.0` this package has introduced a major change — migrating underlying library `LibSass` to `Dart Sass`. If you are updating an existing project from an earlier version of this plugin to `5.0.0`, be sure to read the [Migration Guide](https://github.com/illusionfield/fourseven-scss/blob/master/MIGRATION-GUIDE-v5.md), as this documentation does not cover all required migration steps.

## Installation

Install the package using Meteor's package management system:

```sh
meteor add fourseven:scss
```

If you are using this plugin in a Meteor package, add it in the `onUse` block of your package's control file:

```javascript
Package.onUse(function (api) {
  ...
  api.use('fourseven:scss');
  ...
});
```

## Compatibility

| Meteor Version | Recommended `fourseven:scss` version |
| ---------------| ------------------------------------ |
| 1.0 - 1.1      | 3.2.0                                |
| 1.2 - 1.3.1    | 3.4.2                                |
| 1.3.2+         | 3.8.0_1                              |
| 1.4.0          | 3.8.1                                |
| 1.4.1+         | 4.5.4                                |
| 1.6+           | 4.12.0                               |
| 2.10+          | 5.0.0                                |

## Usage

After installation, this package automatically finds all `.scss` and `.sass` files in your project,
compiles them with [Dart Sass](https://www.npmjs.com/package/sass),
and includes the resulting CSS in the application's client bundle.
These files can be located anywhere in your project.

### File types

There are two main types of Sass files handled by this package:

- **Sass source files**: These are the `*.scss` and `*.sass` files that are not imports.
- **Sass imports/partials**: These are files prefixed with an underscore (`_`) or explicitly marked as `isImport: true` in the package's `package.js` file, like this:

```javascript
api.addFiles('x.scss', 'client', { isImport: true });
```

Each compiled source file generates a separate CSS file, which is then merged into one by the `standard-minifiers` package.

### Importing

You can import styles from various locations, including other packages, your own app, and npm modules.

#### Importing from another package

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

#### Importing from your app

```scss
@use "{}/client/styles/imports/colors.scss" as *;

.my-nav {
  background-color: @primary-branding-color; // Use a color from the app's style palette
}
```

#### Importing from npm modules

```scss
@use "~module-name/stylesheet"; // Imports a module from node_modules
```

> [!TIP]
> If the target is a directory, it will search for an `index.scss`, `_index.scss`, `index.sass`, or `_index.sass` file inside that directory.

### Custom configuration

You can customize the behavior of the Sass compiler by creating a `.scss.config.json` file in the root of your project.

#### Sass compile options

Currently, this file supports only the `quietDeps` and `verbose` options, but more options will be available in the future.
Learn more about the available options in the [Sass compiler options](https://sass-lang.com/documentation/js-api/interfaces/options).

#### Global include path

When working with long and complex project structures,
specifying the full path to a (e.g. mixin) library in every SCSS file can be cumbersome.
Configuring a global `includePaths` option allows you to define shorter, more maintainable import paths.

Additionally, if the directory structure of the libraries changes,
you only need to update the configuration file once instead of modifying multiple SCSS files.

```scss
@use "mixins"; // This file is actually placed in another directory!
```

> [!NOTE]
> This plugin does not use [Sass’s built-in loadPaths](https://sass-lang.com/documentation/js-api/interfaces/options/#loadPaths) option.Instead, it implements a custom method tailored to the Meteor environment, ensuring seamless integration with Meteor’s build system and package resolution.

#### Example configuration

```json
{
  "quietDeps": true,
  "verbose": false,
  "includePaths": [
    "{}/some/very/long/path/for/holding/my/mixins/folder"
  ]
}
```

### Source maps

These are on by default.

### Autoprefixer

In Meteor projects compatible with version `5.0.0` of this plugin (see [compatibility table](#compatibility)), prefixes are supported via PostCSS.

```sh
meteor add standard-minifier-css
meteor npm install --save-dev autoprefixer postcss postcss-load-config
```

Learn more about [`standard-minifier-css` setup in its docs](https://docs.meteor.com/packages/standard-minifier-css.html)

## Debug mode for Development and Troubleshooting

You can enable debug mode by setting the `DEBUG_PACKAGE_SASS` environment variable to any value other than `false` or `0`.
In debug mode, additional error messages and stack traces will be logged to the console, providing more detailed error logs.

```sh
export DEBUG_PACKAGE_SASS=true
```

This can be helpful for troubleshooting issues during compilation.
