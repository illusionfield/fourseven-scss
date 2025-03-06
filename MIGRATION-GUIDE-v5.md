# Meteor SCSS Migration Guide

The latest version of the `fourseven:scss` package includes significant changes compared to the previous node-sass implementation.
It now uses Dart Sass for compilation, which is modern, faster, and officially supported by the Sass community.
This guide provides a step-by-step process to help developers transition smoothly to the new version.

## Why is this change happening?

- The Node.js-based `node-sass` relies on the outdated **LibSass** compiler, which is no longer maintained.
- Dart Sass is the official Sass implementation that supports the latest standards and features.
- Meteor 3 compatibility requires modern packages.
- This transition ensures **better compatibility and long-term maintainability**.

## Table of contents

- Step 1: [Update the fourseven:scss package](#update-the-foursevenscss-package)
- Step 3: [Install and use NPM packages for autoprefixing](#installing-and-configuring-new-autoprefixer)
- Step 4: [Change @imports to @use and @forward](#change-imports-to-use-and-forward)
- Step 6: [Update configuration file](#update-configuration-file)

## Migration steps

### Update the `fourseven:scss` package

> [!WARNING]
> Critical: Remove outdated packages. Failure to remove `seba:minifiers-autoprefixer` may lead to errors or incorrect CSS processing by the Meteor build system!

```sh
meteor remove seba:minifiers-autoprefixer
meteor add standard-minifier-css
meteor update fourseven:scss
```

This automatically switches to the Dart Sass-based compiler.

### Installing and configuring new autoprefixer

In Meteor projects compatible with version `5.0.0` of this plugin, prefixes are supported via PostCSS.
Ensure to add the following package:

```bash
meteor add standard-minifier-css
meteor npm install --save-dev autoprefixer postcss postcss-load-config
```

Learn more about [`standard-minifier-css` setup in its docs](https://docs.meteor.com/packages/standard-minifier-css.html)

## Change @imports to @use and @forward

Dart Sass does not support the `@import` directive. Replace it with `@use` and `@forward`.

### Old `@import` Syntax

```scss
@import "variables";
@import "mixins";
```

### New Dart Sass Syntax

```scss
@use "variables";
@use "mixins";
```

To make imported files globally available, use **`@forward`** instead of `@use`.

### Comparison: `@import` vs. `@use`

| Legacy `@import`                      | Modern `@use`                    |
| ------------------------------------- | -------------------------------- |
| `@import "file";`                     | `@use "file";`                   |
| `@import "file";` (global namespace)  | `@use "file" as *;`              |
| `@import "file";` (to forward styles) | `@forward "file";`               |

For details, see the [Sass Documentation](https://sass-lang.com/documentation/at-rules/import).

> [!NOTE]
> Throughout the rest of this document, we will refer to `@use` and `@forward` instead of `@import`, as `@import` is no longer supported in Dart Sass. For more information, see: [`SASS Lang: @import` is Deprecated](https://sass-lang.com/blog/import-is-deprecated)

### Error: "Undefined variable" (`@use` issue)

Dart Sass requires explicit prefixes for imported variables and mixins.

❌ **Incorrect (`@use` does not expose variables directly):**

```scss
@use "variables";

body {
  background-color: $primary-color; // ❌ Error!
}
```

✅ **Solution (`as *` or adding a prefix):**

```scss
@use "variables" as v;

body {
  background-color: v.$primary-color;
}
```

Or, to expose everything globally:

```scss
@use "variables" as *;
```

## Check file paths

Previously, the `includePaths` option was necessary for proper resolution of relative imports. With Dart Sass, this is handled natively.

### Importing from Node Modules

Old:

```scss
@import "{}/node_modules/bootstrap/scss/bootstrap";
```

New ([recommended by Dart Sass official documentation](https://sass-lang.com/documentation/js-api/interfaces/fileimporter/)):

```scss
@use "~bootstrap/scss/bootstrap";
```

### Importing from Meteor Packages

> [!IMPORTANT]
> Importing styles from a different package has changed!

Old:

```scss
@import "{my-package:styles}/main";
```

New:

```scss
@use "meteor:{my-package:styles}/main";
```

The `meteor:` prefix is now required due to the Dart Sass importer strategy.

### Index files

If `@use` or `@forward` targets a directory, Dart Sass searches for one of the following files:

- `index.scss`
- `_index.scss`
- `index.sass`
- `_index.sass`

Example:

```scss
// Loads "index.scss" or "_index.scss" if found in the "styles" directory.
@use "styles";
```

For more details, see [Sass Documentation on Index Files](https://sass-lang.com/documentation/at-rules/import/#index-files).

## Additional changes

### Indented syntax support

Previously, due to a limitation in LibSass, indented syntax (`.sass`) files could only import at the top level.
This limitation has been removed in Dart Sass, allowing imports at any level.

### Update configuration file

The configuration file has been renamed from `scss-config.json` to `.scss.config.json` to support a wider range of Dart Sass-specific settings.

> [!WARNING]
> In the future, `scss-config.json` will no longer be supported. Please make sure to rename it in time.

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

### Debug mode for Development and Troubleshooting

You can enable debug mode by setting the `DEBUG_PACKAGE_SASS` environment variable to any value other than `false` or `0`.
In debug mode, additional error messages and stack traces will be logged to the console, providing more detailed error logs.

```sh
export DEBUG_PACKAGE_SASS=true
```

This can be helpful for troubleshooting issues during compilation.

## Summary

| **Change**             | **Old (`node-sass`)**                   | **New (`Dart Sass`)**                    |
| ---------------------- | --------------------------------------- | ---------------------------------------- |
| Sass Compiler          | `node-sass` (LibSass)                   | `dart-sass`                              |
| Minimum Meteor Version | 1.6+                                    | 2.10+ (Full Meteor 3 support)            |
| Import Syntax          | `@import`                               | `@use` and `@forward`                    |
| Module Import          | `@import "{}/node_modules/module-name"` | `@use "~module-name"`                    |
| Meteor Package Import  | `@import "{package}/styles"`            | `@use "meteor:{package}/styles"`         |
| Variable & Mixin Usage | Direct variable access                  | Prefix required (using `as *` or `as v`) |
| CSS Minification       | `seba:minifiers-autoprefixer`           | `standard-minifier-css`                  |
| Indented Syntax        | No import support                       | Fully supported                          |
| Config File Name       | `scss-config.json`                      | `.scss.config.json`                      |
| Debug Mode             | Not available                           | Enabled via environment variable         |

This guide helps developers transition to the Dart Sass-based `fourseven:scss` package.

In the future updates, primarily focuses on ensuring compatibility with Meteor 3 and modern Sass features.
However, we aim to support `Meteor 2` as long as feasible. This version is compatible with `Meteor 2.10` and later releases.

## Further reading

### Official documentation

- [Sass Documentation](https://sass-lang.com/documentation)
- [SASS Lang: @import is Deprecated](https://sass-lang.com/blog/import-is-deprecated)
- [Meteor `standard-minifier-css` package](https://docs.meteor.com/packages/standard-minifier-css.html)

### Related packages

- [`fourseven:scss`](https://atmospherejs.com/fourseven/scss)
- [`standard-minifier-css`](https://atmospherejs.com/meteor/standard-minifier-css)

### Additional learning resources

- [Dart Sass Official Docs](https://sass-lang.com/dart-sass)
- [PostCSS Documentation](https://postcss.org/)
- [Meteor Package Docs](https://docs.meteor.com/)
