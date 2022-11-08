# Issue with test case

Sometime we can't reproduce an customer's problem based only on the Issue description.
In this case the preffered way is to make a PR with a reproducible Test Case.

## Cycle ORM

Examples of other test cases you can find in the
[Cycle reository](https://github.com/cycle/orm/tree/2.x/tests/ORM/Functional/Driver/Common/Integration).

To implement custom test case you need to do few actions.

### Prepare a case template

0. Fork and clone `cycle/orm` repository. Install `composer` dependencies.
1. Run `php tests/generate-case.php`. \
   You will have the new test case in the `tests/ORM/Functional/Driver/Common/Integration` directory.

### Implement your case

In the generated Case directory you'll find:

- Prepared entities in the `Entity` dir.
- Related ORM Schema preset in the `schema.php` file.
- Test class `CaseTest.php` with one example test method.

Feel free to change all of them to reproduce your case.

> **Note**
> If the issue doesn't depend on any database driver you can use only SQLite driver to test it
> locally with better performance (the `pdo_sqlite` extention required).

To run test case `Case42` with the SQLite driver just execute:

```bash
vendor/bin/phpunit --group driver-sqlite --filter Case42
```

### Make a PR

Push your changes in a new branch and go to [Cycle ORM repository](https://github.com/cycle/orm/pulls).\
You will see the suggestion with button `Compare & pull request`. \
Press it and fill the form.
