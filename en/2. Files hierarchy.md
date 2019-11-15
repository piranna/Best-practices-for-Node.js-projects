# Files hierarchy

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

- `.eslintignore`: not an usual one, but sometimes useful. Define files to be
  ignored when running the `eslint` linter tool, it's good to ignore coverage
  report files and documentation helper files.

  ```sh
  # Does .eslintignore file exists?
  test -f .eslintignore

  # Has .eslintignore file all needed patterns?
  grep -l -q -v -w 'bower_modules|coverage|node_modules|vendor|*.bkp|*.tmp' \
    .eslintignore
  ```

### No `node_modules` folder

Some years ago, it was recommended to include the `node_modules` folder in
git repositories to have a exact copy of what dependencies whare being used,
but after release of `npm` 3 dependencies control got to be more stable and
not needed anymore, and since version 5 with its support for maximally plain
dependencies, this is totally discouraged (including `bundleDependencies`
feature). `node_modules` folder must not be included in git repositories
anymore, and if a change on one of them is needed, it's better to do a fork
of the original repository and use this one as dependency.

```sh
# Is npm version recent enought?
npm -v  # > 5.0.0

# Does .gitignore file exists?
test -f .gitignore

# Has .gitignore file a pattern for the node_modules folder?
grep -l -q -v -w 'node_modules' .gitignore
```

## `package.json`

```sh
# Has package.json file all the required fields?
npx package-json-validator
```

Main manifest and description file for the npm package. There are some fields
that properly configured improve the quality and flexibility of the package:

- `description`: define the purpose of the package in a single sentence for
  quick reference. Try to fit it in 50 or 60 characters.

  ```sh
  # Is description field long enought?
  npm -c "require('package.json').description.length > 10"

  # Can description field be shown in a single line?
  npm -c "require('package.json').description.length < 60"
  ```

- `version`: version of the package. This is usually misconfigured although is
  very important. Keep it updated each time a new release or milestone is done,
  both for reference and to have dependencies working correctly. Ideally, you
  should follow [Semantic Versioning](https://semver.org/) rules.

  ```sh
  # Has version field the default value?
  npm -c "require('package.json').version !== '1.0.0'"

  # Has version field a valid semver string?
  npx semver `npm -c "require('package.json').version"`
  ```

- `author` and `contributors`: provide some info about the developer of the
  module. When it's a single coder, only `author` is used for the original
  author and `contributors` is left for third-party coders, but when working on
  an organization, `author` is used to define the organization name and URL and
  `contributors` is used to define the coders of the package, being the first
  one in the list the original author and/or main maintainer of the package.

  ```sh
  # Is author field defined?
  npm -c "require('package.json').author.length"

  # Is contributors field defined?
  npm -c "require('package.json').contributors.length"
  ```

- `license`: license of the package. By default `npm` set this field to the
  permissive open source [ISC](https://opensource.org/licenses/ISC) license,
  that allow to do anything with the code as far as a reference to the original
  author is included. This license easily fits in the way the Node.js ecosystem
  works, but could lead to legal issues to corporate and privative code. If this
  license doesn't fit the actual status of your code, you should fix it soon.

  ```sh
  # Has license field the default value?
  npm -c "require('package.json').license !== 'ISC'"
  ```

- `private`: indicate if the module should not be published to public package
  registries, like [Github registry](https://github.com/features/packages) or
  [npm registry](https://www.npmjs.com).

  ```sh
  # Is project private?
  npm -c "require('package.json').private === true"
  ```

- `dependencies` and `devDependencies`: libraries and packages needed by the
  package.

  These are the **most important fields** of all the file, and you should pay
  attention to have them updated all the time, both by using tools like
  [npm-check-updates](https://github.com/tjunnone/npm-check-updates) and/or
  external services like [Greenkeeper](https://greenkeeper.io/) or
  [Dependabot](https://dependabot.com/), this last one being now part of Github.
  Be sure to have a good tests battery to detect if a dependencies upgrade break
  the compatibility and fix it before blindly doing it.

  ```sh
  # Is Dependabot configured?
  test -d .dependabot

  # Are dependencies updated?
  npx npm-check-updates --semverLevel major
  # npx npm-check-updates --semverLevel major -u  # FIX
  ```

  Other usual problem is to mix packages defined in `dependencies` and
  `devDependencies` fields. First one must define all the dependencies needed
  to use the package in production, including needed dependencies to build it in
  the target machine, while the latest one must include *only* the dependencies
  needed during its development, like testing tools or transpilers.

  ```sh
  # Are all runtime project dependencies defined?
  npx eslint --no-eslintrc \
    --rule '"import/no-extraneous-dependencies": "error"' .

  # Has dependencies field no development dependencies?
  npm -c "!Object.keys(require('package.json').dependencies).includes('babel-cli')"
  ```

  If a package includes an executable that requires some dependencies like an
  arguments parser or terminal coloring, they can be added in the `dependencies`
  field too, but usually the
  [recommended](https://github.com/npm/npm/issues/3739#issuecomment-22076022)
  way is to move that executable to an external package (named by convention
  with a `-cli` suffix), lefting the original one as a `require()`-able
  library-only package.

  ```sh
  # Has server.js file other dependencies than the module itself?
  grep require server.js | wc -l
  ```

- `bin`: this field describe one or several executable files that will be used
  as CLI commands, and will be available in the system path when running them
  using `npm`. When having a single CLI command, it's good to point it to the
  standard `server.js` file, and when having several ones, it's best to have
  them stored in the `bin/` folder and add entries to all of them in this field.

  ```sh
  # Is server.js the package main binary?
  npm -c "require('package.json').bin === 'server.js'"

  # Does bin/ folder exists?
  test -d bin
  ```

- `scripts`: this field store shell commands that can be executed by using the
  `npm` CLI tool, using `npm run <script name>`, with the addition of having the
  executable files from `bin` field already in the system path. The most
  important ones are `start` (that's the standard way to run Node.js
  applications, and by default it makes use of the `server.js` file) and `test`
  (that's the standard way to run tests). You can add your own scripts here too,
  so in most cases there's no need to use external tasks runners like `gulp` or
  `grunt`, or have ad-hoc scripts in the repo. In case you need them, or the
  commands to be used are a bit complex, store them in shell script files or
  javascript executable files, store them in a `scripts` folder, and just set
  the `scripts` field to exec them.

  ```sh
  # Has project tests?
  npm -c "require('package.json').scripts.test != null"

  # Is there any skipped test?
  npx eslint --no-eslintrc --rule '"jest/no-disabled-tests": "error"' .

  # Are tests passing successfully?
  npm test

  # Does tests generate coverage info?
  test -d coverage
  # jest --coverage  # FIX

  # Is tests coverage greater than 90%?

  # Is there a scripts folder?
  test -d scripts
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

## Transpilation

If you are writting your project source code not in javascript but with other
language or using non-standard features, you need transpile (*translate
compile*) with Babel or some other tool to make it executable by Node.js or by
the browser. This includes when you are building a webapp, where you need to
bundle and minify all your javascript and CSS files to get them ready to
publish. The standard way for this is to add a `build` script to `package.json`
file, so it's possible to build them (usually generating code only for
development stage) just by executing `npm run build`.

```sh
# Has package.json file a prepare script?
npm -c "require('package.json').scripts.build != null"
```

One good idea, specially for transpiled libraries, is to build them when
installing from a git repository. This way, in case you need to do some changes
from the original source code (for example, you've found a bug), you can point
your git repository as dependency and its code will be transpiled automatically.
To do so, add a new `prepare` script entry that simply call your build/transpile
script. This has the drawback that you would need to add the build dependencies
in the `dependencies` field, but alternatively you can be able to define them as
`devDependencies` field of the parent project root `package.json` file.

```sh
# Has package.json file a prepare script pointing to build script?
npm -c "require('package.json').scripts.prepare === 'npm run build'"
```

Use the convention folder `src/` to store the original source code needs to be
transpiled with Babel or any other tool. If that's the case, then config your
build scripts to store the transpiled files in a clean `lib/` folder (**never**
mix actual sources with generated code), or if it makes sense for your project
(for example, when it's a collection of components that could be used
independently), then store the transpiled files to the project root and use the
default `index.js` file instead and add the `src/` folder to the `.npmignore`
file to publishhave a clean package.

```sh
# Does code needs transpilation?
npm -c "require('package.json').devDependencies['babel-cli']"

# Does src/ folder exists?
test -d src
```
