---
title: Continuous Integration Solutions on Pantheon
description: Run automated unit and integration tests with Terminus and Drupal SimpleTest.
tags: [platformintegrations]
categories: []
---
Continuous Integration (CI) is a method of running automated unit and integration tests to apply quality control. Pantheon doesn't provide or host tools for continuous integration, but many tools and techniques are compatible with Pantheon. If you have a particular use case or technique that you'd like to highlight, let us know by [contacting support](/docs/support).

## Terminus Command-Line Interface

[Terminus](/docs/terminus/) is a Drush-based command-line interface (CLI) in the Pantheon core API. Most operations available through the Pantheon Dashboard can be performed with Terminus, including:

- Site creation
- [Multidev environment](/docs/multidev) creation and removal
- Content cloning
- Code pushes

You can use Terminus for scripting many operations. For example, a post-commit hook can trigger Jenkins to create a Multidev environment with the latest code on master and the content from Live, then run automated browser tests using [Selenium](https://github.com/SeleniumHQ/selenium).

## Drupal SimpleTest

The drush test-run command was dropped in Drush 7 and 8.  See this GitHub issue for more details: https://github.com/drush-ops/drush/issues/1362.

[SiteTest](https://www.drupal.org/project/site_test) is a contrib module designed to allow running tests directly against your sites code instead of a base drupal clone of your site.  This module is recommended for use of SimpleTest on Pantheon.

[SimpleTest](https://drupal.org/project/simpletest) is a testing framework based on the [SimpleTest PHP library](https://github.com/simpletest/simpletest) that is included with Drupal core. If you are creating a custom web application, you should consider including SimpleTests of your module functionality.

When running tests on Pantheon you will want to enable site_test using the following command:
```
terminus drush $SITE_NAME.$ENV_NAME -- en site_test -y
```

When running tests on Pantheon you will want to clear the cache immediately before running test to avoid strange failures:
```
terminus drush $SITE_NAME.$ENV_NAME -- cc all
```

When running tests on Pantheon you will need to get the absolue path before you can run the script:
```
terminus drush $SITE_NAME.$ENV_NAME -- eval "echo DRUPAL_ROOT"
```
NOTE: In the above command you may need to strip out warnings by ending the command with `2>/dev/null`

To run tests the full command will look something like this:
```
terminus drush $SITE_NAME.$ENV_NAME -- exec php `terminus drush $SITE_NAME.$ENV_NAME -- eval "echo DRUPAL_ROOT" 2>/dev/null`/scripts/run-tests.sh --url http://$ENV_NAME-$SITE_NAME.pantheonsite.io Genomeweb
```
NOTE: In the above command the `--url` option is required to be passed as multidevs do not respond to `localhost`.

A full CircleCI command might look similar to this:
```
      - run:
          name: Test simpletest
          command: |
            if [ "${CIRCLE_BRANCH}" != "master" ]; then

              terminus drush $SITE_NAME.$ENV_NAME -- en site_test -y

              # If you don't clear the cache immediately before running tests
              # we get the html gibberish instead of a passing test.
              terminus drush $SITE_NAME.$ENV_NAME -- cc all

              # NOTE: It is a requirement to be running the latest version of
              # terminus in order to exclude the notice in shell output of the
              # embedded command to find the absolute path.
              terminus drush $SITE_NAME.$ENV_NAME -- exec php `terminus drush $SITE_NAME.$ENV_NAME -- eval "echo DRUPAL_ROOT" 2>/dev/null`/scripts/run-tests.sh --url http://$ENV_NAME-$SITE_NAME.pantheonsite.io ExampleTestGroupName

            fi
```

## Integration Bot

We recommend creating a bot user account that will handle the tasks or jobs by an external continuous integration service rather a standard user account.

- Add the bot to select projects
- Manage separate SSH Keys for CI

## Known Limitations

At this time, Pantheon does not provide or support:

  - [Webhooks](https://en.wikipedia.org/wiki/Webhook)
- [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
- Running [Jenkins](https://jenkins.io/index.html) or other Continuous Integration software on our servers. You'll need to self-host or use a hosted CI solution. [Compare solutions here](https://en.wikipedia.org/wiki/Comparison_of_continuous_integration_software).
- Shell access
- [PHPUnit](https://github.com/sebastianbergmann/phpunit/) Unit Testing PHP Framework: You can still write tests and include them in your code, but you'll need to run them on a CI server, not Pantheon.
