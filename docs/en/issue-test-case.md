# Issue with test case

Sometime we can't reproduce an customer's problem based only on the Issue description.
In this case the preffered way is to make a PR with a reproducible Test Case.

## Cycle ORM

Examples of other test cases you can find in the
[Cycle reository](https://github.com/cycle/orm/tree/2.x/tests/ORM/Functional/Driver/Common/Integration).

To implement custom test case you need to do few actions.

### Prepare a case template

0. Fork and clone `cycle/orm` repository. Install `composer` dependencies.
1. Go to the dir `tests/ORM/Functional/Driver/Common/Integration`.
2. Make a copy of the `CaseTemplate` catalog to the same place with your case name. At can be
   `Case№` with next case number.
3. Replace `CaseTemplate` with your case name in namespaces of all case files.
4. Run `php tests/generate.php` to generate child classes for all supported databases.

### Implement your case

In the copied directory you'll find:

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
