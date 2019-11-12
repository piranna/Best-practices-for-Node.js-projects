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
- `.npmignore`: files to not being added to the npm package. It's not needed at
  all, but improve package quality and reduce bandwidth usage when installing.
  It should be set to ignore tests files, and source files when using some
  transpiler tool like Babel.
- `.eslintignore`: not an usual one, but sometimes useful. Define files to be
  ignored when running the `eslint` linter tool, it's good to ignore coverage
  report files and documentation helper files.

### No `node_modules` folder

Some years ago, it was recommended to include the `node_modules` folder in
git repositories to have a exact copy of what dependencies whare being used,
but after release of `npm` 3 dependencies control got to be more stable and
not needed anymore, and since version 5 with its support for maximally plain
dependencies, this is totally discouraged (including `bundleDependencies`
feature). `node_modules` folder must not be included in git repositories
anymore, and if a change on one of them is needed, it's better to do a fork
of the original repository and use this one as dependency.

## `package.json`

Main manifest and description file for the npm package. There are some fields
that properly configured improve the quality and flexibility of the package:

- `description`: define the purposse of the package in a single sentence for
  quick reference. Try to fit it in 50 or 60 characters.
- `version`: version of the package. This is usually misconfigured although is
  very important. Keep it updated each time a new release or milestone is done,
  both for reference and to have dependencies working correctly. Ideally, you
  should follow [Semantic Versioning](https://semver.org/) rules.
- `author` and `contributors`: provide some info about the developer of the
  module. When it's a single coder, only `author` is used for the original
  author and `contributors` is left for third-party coders, but when working on
  an organization, `author` is used to define the organization name and URL and
  `contributors` is used to define the coders of the package, being the first
  one in the list the original author and/or main maintainer of the package.
- `dependencies` and `devDependencies`: libraries and packages needed by the
  package.

  These are the **most important fields** of all the file, and you should pay
  attention to have them updated all the time, both by using tools like
  [npm-check-updates](https://github.com/tjunnone/npm-check-updates) and/or
  external services like [Greenkeeper](https://greenkeeper.io/) or
  [Dependabot](https://dependabot.com/), this last one being now part of Github.
  Be sure to have a good tests battery to detect if a dependencies upgrade break
  the compatibility and fix it before blindly doing it.

  Other usual problem is to mix packages defined in `dependencies` and
  `devDependencies` fields. First one must define all the dependencies needed
  to use the package in production, including needed dependencies to build it in
  the target machine, while the latest one must include *only* the dependencies
  needed during its development, like testing tools or transpilers. If a package
  includes a command that requires some dependencies like an arguments parser or
  terminal coloring, they can be added in the `dependencies` field too, but
  usually it's better to move that command to an external package (usually named
  with a `-cli` suffix), lefting the original one as a `require()` importable
  library-only package.
- `bin`: this field describe one or several executable files that will be used
  as CLI commands, and will be available in the system path when running them
  using `npm`. When having a single CLI command, it's good to point it to the
  standard `server.js` file, and when having several ones, it's best to have
  them stored in the `bin/` folder and add entries to all of them in this field.

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

## Main file

By convention, there are several standard main entry files in the modules
depending of their purposse. In order of importance, they are:

- `server.js`: default script executed when running `npm start`. This should be
  used only for the main command being offered to the user, both being the
  single one or not, and later import all the module functionality from the
  `app.js` file or the module itself to make use of it. Needs to be an
  executable with execution bit permission enabled, and in case of Node.js
  scripts, a shebang on top of it.
- `app.js`: standard convention for `express` applications. This one should
  define and export the express `App` instance (or a factory initializing it),
  so later can be `require()`d and used from `server.js`. This file is also used
  for the main `App` component when working with React or ReactNative.
- `index.js`: default entrypoint exported by the `main` field of `package.json`
  file. In fact, when requiring a folder with Node.js, it will import the
  `index.js` file inside it, so this replicate that behaviour. If your module
  can be defined in a single file (not counting the CLI command file), then you
  should define it here.
- `lib/`: convention entrypoint folder when a project makes use of several
  files, instead of `index.js` in the root folder. Use this ONLY when it's
  actual code that doesn't needs to be transpiled with Babel or any other tool.
  If that's the case, then use `src/` folder and store the transpiled files in
  a clean `lib/` folder (never mix actual sources with generated code), or if
  it makes sense like when it's a collection of components that could be used
  independently, then build them to the project root.
