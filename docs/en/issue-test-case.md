# Issue Reproduction through Test Cases

Ever run into a tricky issue that words just can't seem to describe? Or found yourself stuck with a problem that’s hard
for others to reproduce? That’s where a reproducible test case comes into play. By creating and submitting a test case,
you provide the community with a notable representation of the problem, making it easier for everyone to understand and
solve.

This guide will walk you through the process of creating and submitting a test case to the Cycle ORM repository,
ensuring your issue gets the attention and resolution it deserves. **Dive in and let’s get started!**

## Cycle ORM

To get a sense of how test cases should look, check out the examples in
the [Cycle reository](https://github.com/cycle/orm/tree/2.x/tests/ORM/Functional/Driver/Common/Integration).

### Setting Up Your Test Case Template

Here's a step-by-step guide to crafting your test case:

1. Begin by forking and cloning the `cycle/orm` repository. Make sure to install the necessary composer dependencies.
2. Run `php tests/generate-case.php`.
   This will create a new test case under the `tests/ORM/Functional/Driver/Common/Integration` directory.

### Implement your test case

Within the directory of your generated test case, you'll discover:

- Set-up entities inside the `Entity` folder.
- The corresponding ORM Schema configuration in the `schema.php` file.
- A `CaseTest.php` test class, complete with a sample test method.

Modify these files as needed to mirror the issue you're experiencing.

> **Note**
> If the problem isn't tied to a specific database driver, you can stick with the SQLite driver for local testing. This
> provides faster results (though you'll need the `pdo_sqlite` extension).

To run test case `Case42` with the SQLite driver just execute:

```bash
vendor/bin/phpunit --group driver-sqlite --filter Case42
```

### Submitting Your PR

After making your changes, push them to a fresh branch. Then, head over to
the [Cycle ORM repository](https://github.com/cycle/orm/pulls). There, you'll see a prompt with the option **Compare &
pull request**. Click on it and complete the form.
