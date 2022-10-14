# Contributing

Feel free to contribute to the development of the Cycle ORM or its components. Please make sure that the following
requirements are satisfied before submitting your pull request:

* KISS
* PSR-12
* `declare(strict_types=1);` is mandatory
* Your code must include tests

> **Note**
> Use our discord server to check for the advice or suggestion [https://discord.gg/kNUhSev](https://discord.gg/kNUhSev)

## Testing Cycle

To test ORM engine locally, download the `cycle/orm` repository and start docker containers inside the tests folder:

```bash
cd tests
docker-composer up
```

To run full test suite:

```bash
./vendor/bin/phpunit
```

To run quick test suite:

```bash
./vendor/bin/phpunit tests/ORM/Functional/Driver/SQLite
```

## Help Needed In

If you want to help but don't know where to start:

* TODOs
* Quality recommendations and improvements
* Check [Open Issues](https://github.com/orgs/cycle/projects/1/views/8?filterQuery=status%3ATodo)
* More tests are always welcome
* Typos

Feel free to propose any ideas related to architecture, docs (___docs are never complete___), adaptation or community.

> **Note**
> Original guide author is not a native English speaker, feel free to create PR for any text corrections.

## Questions

If you have a question you can ask it in one of our official channels:

1. [GitHub discussions](https://github.com/cycle/orm/discussions)
2. [Discord Server](https://discord.gg/rPneHh7z6Y)

## Issues

### Common issues

If you found obvious bug or misbehavior feel fre to open a new issue in related repository on
[GitHub](https://github.com/cycle). \
It will be great if you create a [PR with a test case](issue-test-case.md) to reproduce the problem.

Dont be shy to create a feature request [in a separated issue](https://github.com/cycle/orm/issues/new/choose).

### Critical/Security Issues

If you found something which shouldn't be there or a bug which opens a security hole please let me know immediately by
email [team@spiralscout.com](mailto:team@spiralscout.com)

## Official Support

Cycle ORM and all related components are maintained by [Spiral Scout](https://spiralscout.com/).

For commercial support please contact team@spiralscout.com.

## Licensing

Cycle ORM and its components will remain under [MIT license](license.md) indefinitely.
