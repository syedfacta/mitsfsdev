# FACTA EXAMPLE


This repository is a reference implementation and start state for a modern WordPress workflow utilizing [Composer](https://getcomposer.org/), Continuous Integration (CI), Automated Testing, and Pantheon. Even though this is a good starting point, you will need to customize and maintain the CI/testing set up for your projects.

This repository is meant to be copied one-time by the the [Terminus Build Tools Plugin](https://github.com/pantheon-systems/terminus-build-tools-plugin) but can also be used as a template. It should not be cloned or forked directly.

The Terminus Build Tools plugin will scaffold a new project, including:

* A Git repository
* A free Pantheon sandbox site
* Continuous Integration configuration/credential set up

For more details and instructions on creating a new project, see the [Terminus Build Tools Plugin](https://github.com/pantheon-systems/terminus-build-tools-plugin/).

## Important files and directories

### `/web`

Pantheon will serve the site from the `/web` subdirectory due to the configuration in `pantheon.yml`. This is necessary for a Composer based workflow. Having your website in this subdirectory also allows for tests, scripts, and other files related to your project to be stored in your repo without polluting your web document root or being web accessible from Pantheon. They may still be accessible from your version control project if it is public. See [the `pantheon.yml`](https://pantheon.io/docs/pantheon-yml/#nested-docroot) documentation for details.

### `/web/wp`

Even within the `/web` directory you may notice that other directories and files are in different places compared to a default WordPress installation. [WordPress allows installing WordPress core in its own directory](https://codex.wordpress.org/Giving_WordPress_Its_Own_Directory), which is necessary when installing WordPress with Composer.

See `/web/wp-config.php` for key settings, such as `WP_SITEURL`, which must be updated so that WordPress core functions properly in the relocated `/web/wp` directory. The overall layout of directories in the repo is inspired by, but doesn't exactly mirror, [Bedrock](https://github.com/roots/bedrock).

### `composer.json`
This project uses Composer to manage third-party PHP dependencies.

The `require` section of `composer.json` should be used for any dependencies your web project needs, even those that might only be used on non-Live environments. All dependencies in `require` will be pushed to Pantheon. 

The `require-dev` section should be used for dependencies that are not a part of the web application but are necesarry to build or test the project. Some example are `php_codesniffer` and `phpunit`. Dev dependencies will not be deployed to Pantheon.

Third party WordPress dependencies, such as plugins and themes, are added to the project via `composer.json`. The `composer.lock` file keeps track of the exact version of dependency. [Composer `installer-paths`](https://getcomposer.org/doc/faqs/how-do-i-install-a-package-to-a-custom-path-for-my-framework.md#how-do-i-install-a-package-to-a-custom-path-for-my-framework-) are used to ensure the WordPress dependencies are downloaded into the appropriate directory.

Non-WordPress dependencies are downloaded to the `/vendor` directory.

### `.ci`
This `.ci` directory is where all of the scripts that run on Continuous Integration are stored. 

The scripts are organized into subdirectories of `.ci` according to their function: `build`, `deploy`, or `test`.

#### Build Scripts `.ci/build`
Steps for building an artifact suitable for deployment. Feel free to add other build scripts here, such as installing Node dependencies, depending on your needs.

- `.ci/build/php` installs PHP dependencies with Composer

#### Build Scripts `.ci/deploy`
Scripts for facilitating code deployment to Pantheon.

- `.ci/deploy/pantheon/create-multidev` creates a new [Pantheon multidev environment](https://pantheon.io/docs/multidev/) for branches Master, Staging or any other branch prefixing with CI-
  - Note that not all users have multidev access. Please consult [the multidev FAQ doc](https://pantheon.io/docs/multidev-faq/) for details.

## Working locally with Lando
To get started using Lando to develop locally complete these one-time steps. Please note than Lando is an independent product and is not supported by Pantheon. For further assistance please refer to the [Lando documentation](https://docs.devwithlando.io/).

* Clone this  project repository from BitBucket locally.
* Run `lando start` to start Lando.
    - Save the local site URL. It should be similar to `https://<PROJECT_NAME>.lndo.site`.
* Run `lando composer install --no-ansi --no-interaction --optimize-autoloader --no-progress` to download dependencies
* Run `lando pull --code=none` to download the media files and database from Pantheon.
* Visit the local site URL saved from above.

You should now be able to edit your site locally. The steps above do not need to be completed on subsequent starts. You can stop Lando with `lando stop` and start it again with `lando start`.

**Warning:** do NOT push/pull code between Lando and Pantheon directly. All code should be pushed to GitHub and deployed to Pantheon through a continuous integration service, such as CircleCI.

Composer, Terminus and wp-cli commands should be run in Lando rather than on the host machine. This is done by prefixing the desired command with `lando`. For example, after a change to `composer.json` run `lando composer update` rather than `composer update`.
