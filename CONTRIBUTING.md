Contributing
============

Thanks for contributing!

## Development

We primarily develop in `yarn`, but use `npm` in our benchmarks. Please make sure to have something like:

* node: `8+`
* yarn: (anything modern)
* npm:  `5.7.0+`

available. Also note that for the time being, our development tools assume a Unix-like environment, which means Mac, Linux, or something like the Windows Subsystem for Linux.

Our development revolves around various fixture packages we have in `test`. Get set up with:

```sh
$ yarn
$ yarn benchmark:install
```

to install the root project and a lot of fixture packages. (This is **meant** to take a while as we install a lot of dependencies to give us sizable app simulations to work with...) You will need to re-run `benchmark:install` whenever you update dependencies inside `test/` packages.

Our present fixture setup is:

```
$ tree test/packages/ -L 2
test/packages/
├── huge
│   ├── npm
│   └── yarn
├── individually
│   ├── npm
│   └── yarn
├── simple
│   ├── npm
│   └── yarn
└── webpack
    ├── npm
    └── yarn
```

**Note**: Only **some** of the scenarios contribute to the timed benchmark results as some scenarios don't actually use either built-in Serverless or Jetpack packaging.

For ease of development, we want to do `yarn benchmark:install` and install the respective yarn/npm packages **once**. However, this means we keep duplicates of source code / package.json files across the `npm`/`yarn` variant directories. To keep things in sync, we designate the `yarn` directory as "the source of truth" for everything except for `SCENARIO/npm/package-lock.json` and copy files across scenarios with:

```sh
$ yarn benchmark:build
```

From there you can run various packaging configurations and perform benchmarks.

```sh
$ TEST_MODE=yarn TEST_SCENARIO=simple yarn benchmark
$ TEST_MODE=yarn TEST_SCENARIO=simple,huge yarn benchmark
$ TEST_MODE=yarn,npm TEST_SCENARIO=simple yarn benchmark

# Faster, because scenarios run in parallel, but less reliable results because
# of impact on your machine. Use this for faster development, but do a normal
# serial benchmark for pasting results.
$ TEST_PARALLEL=true yarn benchmark
```

After this, we can run benchmark specific QA stuff:

```sh
$ yarn benchmark:lint
$ yarn benchmark:test
```

(The `lint` needs the individual installs and `test` needs file list output from a full benchmark).

### QA

```sh
$ yarn lint
$ yarn test

# ... or all together ...
$ yarn run check
```

## Before submitting a PR...

Before you go ahead and submit a PR, make sure that you have done the following:

```sh
# Run lint and unit tests
$ yarn run check
$ yarn benchmark:lint

# Make sure all fixtures are updated and valid
$ yarn benchmark:install
$ yarn benchmark:build

# Run a benchmark.
# First, check that `npm` versions is 5.7.0+ (which has `npm ci`).
$ npm --version
# Then, actually generate the benchmark.
# _Note_: Unfortunately, this takes some **time**. Grab a ☕
$ yarn benchmark
# Now, test the benchmark for correctness.
$ yarn benchmark:test

# Generate the usage message.
$ yarn usage

# If the timed benchmark stats and/or the usage message are notably different
# than what's in README.md, update relevant sections and commit your changes.
$ vim README.md

# Run all final checks.
$ yarn run check
```

## Releasing a new version to NPM

_Only for project administrators_.

1. Update `CHANGELOG.md`, following format for previous versions
2. Commit as "Changes for version VERSION"
3. Run `npm version patch` (or `minor|major|VERSION`) to run tests and lint,
   build published directories, then update `package.json` + add a git tag.
4. Run `npm publish` and publish to NPM if all is well.
5. Run `git push && git push --tags`