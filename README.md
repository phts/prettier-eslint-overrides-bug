# prettier-eslint-overrides-bug

- eslint 6.0.1 (on 5.16.0 bug is **not** reproduced)
- prettier-eslint-cli 5.0.0

## Run with `overrides[0].files = "src/myFile.js"`

**Missing** `parserOptions` and `env` in the computed config:

```
prettier-eslint [DEBUG]: inferred options: Object {
  "eslintConfig": Object {
    "baseConfig": Object {
      "settings": Object {},
    },
    "env": Object {},
    "fix": true,
    "globals": Array [],
    "parser": null,
    "parserOptions": Object {},
    "plugins": Array [],
    "rules": Object {
      "dot-notation": Array [
        "error",
      ],
    },
    ...

1 file was unchanged
```

This means `prettier-eslint` creates wrong eslint config so eslint cannot parse `myFile.js` because it has ES6 syntax and ES modules. So `eslint --fix` is not executed on this file.

After removing [lines from prettier-eslint/src/index.js](https://github.com/prettier/prettier-eslint/blob/c87b69b769c59bcea0d96c87dddb6a4925085d8a/src/index.js#L226):

```js
if (filePath) {
  eslintOptions.cwd = path.dirname(filePath);
}
```

this example will work.

## Run with `overrides[0].files = "**/src/myFile.js"`

It works! `parserOptions` and `env` **are presented** in the computed config:

```
prettier-eslint [DEBUG]: inferred options: Object {
  "eslintConfig": Object {
    "baseConfig": Object {
      "settings": Object {},
    },
    "env": Object {
      "es6": true,
    },
    "fix": true,
    "globals": Array [],
    "parser": null,
    "parserOptions": Object {
      "ecmaVersion": 2018,
      "sourceType": "module",
    },
    "plugins": Array [],
    "rules": Object {
      "dot-notation": Array [
        "error",
      ],
    },
    ...

success formatting 1 file with prettier-eslint
```

Now `prettier-eslint` creates correct eslint config and `eslint --fix` is executed.

Also any combinations with `**` in the path also work, e.g. `"src/**/myFile.js"`.
