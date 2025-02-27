# Dart Sass for Meteor.js

## Important Notice

This is a major change—from LibSass to Dart Sass. Please read the [Migration Guide](MIGRATION-GUIDE.md) for detailed instructions before proceeding.

This is a build plugin for Meteor.js that compiles Sass files using Dart Sass.

## Installation

Install the package using Meteor's package management system:

```bash
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

After installation, this package automatically finds all `.scss` and `.sass` files in your project, compiles them with [Dart Sass](https://www.npmjs.com/package/sass), and includes the resulting CSS in the application's client bundle. These files can be located anywhere in your project.

### File Types

There are two main types of Sass files handled by this package:

- **Sass source files**: These are the `*.scss` and `*.sass` files that are not imports.
- **Sass imports/partials**: These are files prefixed with an underscore (`_`) or explicitly marked as `isImport: true` in the package's `package.js` file, like this:

```javascript
api.addFiles('x.scss', 'client', { isImport: true });
```

Each compiled source file generates a separate CSS file, which is then merged into one by the `standard-minifiers` package.

### Importing

You can import styles from various locations, including other packages, your own app, and npm modules.

- **Importing from another package**:

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

- **Importing from your app**:

```scss
@use "{}/client/styles/imports/colors.scss" as *;

.my-nav {
  background-color: @primary-branding-color; // Use a color from the app's style palette
}
```

- **Importing from npm modules**:

```scss
@use "~module-name/stylesheet"; // Imports a module from node_modules
```

If the target is a directory, it will search for an `index.scss`, `_index.scss`, `index.sass`, or `_index.sass` file inside that directory.

### Global Include Path

At the moment, there is no support for a global include path. If there is significant demand, this feature could be added in future updates.

### Source Maps

Source maps are enabled by default, helping with debugging by mapping the compiled CSS back to the original Sass files.

### Autoprefixer

To use Autoprefixer, follow the official [PostCSS with Meteor](https://docs.meteor.com/packages/standard-minifier-css.html#standard-minifier-css) documentation.

## Custom Configuration

You can customize the behavior of the Sass compiler by creating a `.scss.config.json` or `scss-config.json` file in the root of your project. This file supports options like `quietDeps`, `verbose`, and others that control Sass compilation behavior.

Example configuration:

```json
{
  "quietDeps": true,
  "verbose": false,
  "includePaths": [
    "{}/some/very/long/path/for/holding/my/mixins/folder"
  ]
}
```

## Additional Features

- **Caching**: The plugin utilizes a caching system to speed up the compilation of Sass files. The default cache size is set to 10 MB.
- **Import Handling**: When importing Sass files, it attempts to resolve various file extensions (`.scss`, `.sass`, `.css`) and also supports imports from `node_modules` using the `~` syntax.
- **Partial File Recognition**: Files with an underscore (`_`) prefix are treated as partials and are not compiled into standalone CSS files unless explicitly requested.

### Debug Mode

You can enable debug mode by setting the `DEBUG_PACKAGE_SASS` environment variable to any value other than `false` or `0`. In debug mode, additional error messages and stack traces will be logged to the console.

```bash
export DEBUG_PACKAGE_SASS=true
```

This can be helpful for troubleshooting issues during compilation.
