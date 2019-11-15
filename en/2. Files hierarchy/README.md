# 2. Files hierarchy

main -> /lib/routes
/app.js
/server.js

Files that export a class by default, named with the class name (including capital letter)
- [ ] Empty file, remove
.gitpreserve
- [ ] I would move step scripts to actual files for clarify and better
      maintenance, instead of have commands hardcoded in the file
- [ ] Using `bin/dcx` as main binary and `start` script instead of standard
      `server.js`

- [ ] Move `translator` to its own file, use `Promise.all` to wait for all
      translations, and wait until all promises are resolver before start the
      web server

## `.*ignore` files

> **TL;DR**
>
> - Remove all unneeded data, and don't include any extra one.

These files have some patterns inside, and make some tools to ignore the files
that match these patterns. The most important ones are:

- `.gitignore`: files to not being added to the git repository. There are
  several standard versions on internet, but today common bare minimum are
  `node_modules` folder, logs, coverage reports and any auto-generated file.

  ```sh
  # Does .gitignore file exists?
  test -f .gitignore

  # Has .gitignore file all needed patterns?
  grep -l -q -v -w 'bower_modules|coverage|logs|*.bkp|*.tmp' .gitignore
  ```

- `.npmignore`: files to not being added to the npm package. It's not needed at
  all, but improve package quality and reduce bandwidth usage when installing.
  It should be set to ignore tests files, and source files when using some
  transpiler tool like Babel.

  ```sh
  # Has package files config or .npmignore file?
  npx package-config-checker --files

  # Does .npmignore file exists?
  test -f .npmignore

  # Has .gitignnpmignoreore file all needed patterns?
  grep -l -q -v -w 'coverage|logs|src|__tests__|*.bkp|*.tmp' .npmignore
  ```

### No `node_modules` folder

Some years ago, it was recommended to include the `node_modules` folder in git
repositories to have a exact copy of what dependencies whare being used, but
after release of `npm` 3 dependencies control got to be more stable and not
needed anymore, and since version 5 with its support for maximally plain
dependencies, this is totally discouraged (including `bundleDependencies`
feature). `node_modules` folder must not be included in git repositories
anymore, and if a change on one of them is needed, it's better to do a fork of
the original repository and use this one as dependency.

```sh
# Is npm version recent enought?
npm -v  # > 5.0.0

# Does .gitignore file exists?
test -f .gitignore

# Has .gitignore file a pattern for the node_modules folder?
grep -l -q -v -w 'node_modules' .gitignore
```

## `README.md`

> **TL;DR**
>
> - Make it clear, it's the project first documentation users and devs will see.

Include basic information of the code of the repository, and so it's the first
file everybody will read about. It should include a description of the purposse
of package itself, how it's organized, how to install and use it, and it's good
to have some info about how to test and debug it too. In any case, it SHOULD NOT
include generic information of the bigger project, of worst, left untouched the
info of the scafolding tool used to generate the package. There's no standard
format, but in https://github.com/RichardLitt/standard-readme you can find some
good guidelines about writting readme files.

```sh
# Does README.md file exists?
test -f README.md

# Has README.md file a valid markdown format?
npx markdownlint-cli .
# npx markdownlint-cli --fix .  # FIX

# Has README.md file a valid markdown format?
npx quality-docs README.md --silent
```

## Main file

By convention, there are several standard main entry files in the modules
depending of their purposse. In order of importance, they are:

- `server.js`: default script executed when running `npm start`. This should be
  used only for the main command being offered to the user, both being the
  single one or not, and later import all the module functionality from the
  `app.js` file or the module itself to make use of it. This has the added
  advantage that will be executed automatically when using `npx` to launch it,
  too. Needs to be an executable with execution bit permission enabled, and in
  case of Node.js scripts, a shebang on top of it.

  ```sh
  # Does server.js file exists?
  test -f server.js

  # Has server.js file a shebang pointing to Node.js?
  file server.js

  # Is server.js file executable? [YES]
  ```

- `app.js`: standard convention for `express` applications. This one should
  define and export the express `App` instance (or a factory initializing it),
  so later can be `require()`d and used from `server.js`. This file is also used
  for the main `App` component when working with React or ReactNative.

  ```sh
  # Is a express project?
  npm -c "require('package.json').dependencies.express != null"

  # Is a React or ReactNative project?
  npm -c "require('package.json').dependencies.react != null"

  # Does app.js file exists?
  test -f app.js

  # Is app.js file executable? [NO]
  ```

- `index.js`: default entrypoint exported by the `main` field of `package.json`
  file. In fact, when requiring a folder with Node.js, it will import the
  `index.js` file inside it, so this replicate that behaviour. If your module
  can be defined in a single file (not counting the CLI command file), then you
  should define it here.

  ```sh
  # Does index.js file exists?
  test -f index.js

  # Is package.json file main field using default index.js value?
  npm -c "require('package.json').main === undefined"
  ```

- `lib/`: convention entrypoint folder when a project makes use of several
  files, instead of `index.js` in the root folder. Use this ONLY when it's
  actual code that doesn't needs to be transpiled with Babel or any other tool.

  ```sh
  # Does lib/ folder exists?
  test -d lib

  # Is package.json file main field pointing to lib/ folder?
  npm -c "require('package.json').main === 'lib'"
  ```
