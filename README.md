<div align="center">
	<p>
		<img alt="CircleCI Logo" src="https://raw.github.com/CircleCI-Public/cimg-php/master/img/circle-circleci.svg?sanitize=true" width="75" />
		<img alt="Docker Logo" src="https://raw.github.com/CircleCI-Public/cimg-php/master/img/circle-docker.svg?sanitize=true" width="75" />
		<img alt="PHP Logo" src="https://raw.github.com/CircleCI-Public/cimg-php/master/img/circle-php.svg?sanitize=true" width="75" />
	</p>
	<h1>CircleCI Convenience Images => PHP</h1>
	<h3>A Continuous Integration focused PHP Docker image built to run on CircleCI</h3>
</div>

[![CircleCI Build Status](https://circleci.com/gh/CircleCI-Public/cimg-php.svg?style=shield)](https://circleci.com/gh/CircleCI-Public/cimg-php) [![Software License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/CircleCI-Public/cimg-php/master/LICENSE) [![Docker Pulls](https://img.shields.io/docker/pulls/cimg/php)](https://hub.docker.com/r/cimg/php) [![CircleCI Community](https://img.shields.io/badge/community-CircleCI%20Discuss-343434.svg)](https://discuss.circleci.com/c/ecosystem/circleci-images)

***This image is designed to supercede the legacy CircleCI PHP image, `circleci/php`.***

`cimg/php` is a Docker image created by CircleCI with continuous integration builds in mind.
Each tag contains a complete PHP version and Composer, everything required for builds to complete successfully in a CircleCI environment.


## Table of Contents

- [Getting Started](#getting-started)
- [How This Image Works](#how-this-image-works)
- [Development](#development)
- [Contributing](#contributing)
- [Additional Resources](#additional-resources)
- [License](#license)


## Getting Started

This image can be used with the CircleCI `docker` executor.
For example:

```yaml
jobs:
  build:
    docker:
      - image: cimg/php:7.4.26
    steps:
      - checkout
      - run: php --version
```

In the above example, the CircleCI PHP Docker image is used for the primary container.
More specifically, the tag `7.4.26` is used meaning the version of PHP will be PHP v7.4.26.
You can now use PHP within the steps for this job.


## How This Image Works

This image contains the PHP programming language as well as Composer and a few very popular PHP extensions.

### PHP Extensions

This image contains [PEAR](https://pear.php.net/) and [PECL](https://pecl.php.net/).
These tools can be used to install various PHP extensions and utilities.
For example, to install pcov, the following command can be run:

```bash
sudo pecl install pcov
```

Older versions of this image suggested installing extensions via `apt-get`.
This is no longer the suggested route unless you are providing your own source of packages and know exactly what you are doing.

### Composer v2

This image includes Composer v2 for tags built on November 2020 and after.
For anyone who can't yet use this and needs to use Composer v1 instead, you can run the following at the beginning of your build:

```bash
- run : sudo composer self-update --1
```

### Variants

Variant images typically contain the same base software, but with a few additional modifications.

#### Node.js

The Node.js variant is the same PHP image but with Node.js also installed.
The Node.js variant can be used by appending `-node` to the end of an existing `cimg/php` tag.

```yaml
jobs:
  build:
    docker:
      - image: cimg/php:7.4.26-node
    steps:
      - checkout
      - run: php --version
      - run: node --version
```

#### Browsers

The browsers variant is the same PHP image but with Node.js, Java, Selenium, and browser dependencies pre-installed via apt.
The browsers variant can be used by appending `-browser` to the end of an existing `cimg/php` tag.
The browsers variant is designed to work in conjunction with the [CircleCI Browser Tools orb](https://circleci.com/developer/orbs/orb/circleci/browser-tools).
You can use the orb to install a version of Google Chrome and/or Firefox into your build. The image contains all of the supporting tools needed to use both the browser and its driver.

```yaml
orbs:
  browser-tools: circleci/browser-tools@1.1
jobs:
  build:
    docker:
      - image: cimg/php:7.4.26-browsers
    steps:
      - browser-tools/install-browser-tools
      - checkout
      - run: |
          php --version
          node --version
          java --version
          google-chrome --version
```

### Tagging Scheme

This image has the following tagging scheme:

```
cimg/php:<php-version>[-variant]
```

`<php-version>` - The version of PHP to use.
This can be a full SemVer point release (such as `7.4.25`) or just the minor release (such as `7.4`).
If you use the minor release tag, it will automatically point to future patch updates as they are released by the PHP Team.
For example, the tag `7.4` points to PHP v7.4.25 now, but when the next release comes out, it will point to PHP v7.4.26.

`[-variant]` - Variant tags, if available, can optionally be used.
For example, the Node.js variant could be used like this: `cimg/php:7.4-node`.


## Development

Images can be built and run locally with this repository.
This has the following requirements:

- local machine of Linux (Ubuntu tested) or macOS
- modern version of Bash (v4+)
- modern version of Docker Engine (v19.03+)

### Cloning For Community Users (no write access to this repository)

Fork this repository on GitHub.
When you get your clone URL, you'll want to add `--recurse-submodules` to the clone command in order to populate the Git submodule contained in this repo.
It would look something like this:

```bash
git clone --recurse-submodules <my-clone-url>
```

If you missed this step and already cloned, you can just run `git submodule update --recursive` to populate the submodule.
Then you can optionally add this repo as an upstream to your own:

```bash
git remote add upstream https://github.com/CircleCI-Public/cimg-php.git
```

### Cloning For Maintainers ( you have write access to this repository)

Clone the project with the following command so that you populate the submodule:

```bash
git clone --recurse-submodules git@github.com:CircleCI-Public/cimg-php.git
```

### Generating Dockerfiles

Dockerfiles can be generated for a specific PHP version using the `gen-dockerfiles.sh` script.
For example, to generate the Dockerfile for PHP v7.4.26, you would run the following from the root of the repo:

```bash
./shared/gen-dockerfiles.sh 7.4.26
```

The generated Dockerfile will be located at `./7.3/Dockefile`.
To build this image locally and try it out, you can run the following:

```bash
cd 7.3
docker build -t test/php:7.3.11 .
docker run -it test/go:7.3.11 bash
```

### Building the Dockerfiles

To build the Docker images locally as this repository does, you'll want to run the `build-images.sh` script:

```bash
./build-images.sh
```

This would need to be run after generating the Dockerfiles first.
When releasing proper images for CircleCI, this script is run from a CircleCI pipeline and not locally.

### Publishing Official Images (for Maintainers only)

The individual scripts (above) can be used to create the correct files for an image, and then added to a new git branch, committed, etc.
A release script is included to make this process easier.
To make a proper release for this image, let's use the fake PHP version of PHP v9.99, you would run the following from the repo root:

```bash
./shared/release.sh 9.99
```

This will automatically create a new Git branch, generate the Dockerfile(s), stage the changes, commit them, and push them to GitHub.
The commit message will end with the string `[release]`.
This string is used by CircleCI to know when to push images to Docker Hub.
All that would need to be done after that is:

- wait for build to pass on CircleCI
- review the PR
- merge the PR

The master branch build will then publish a release.

### Incorporating Changes

How changes are incorporated into this image depends on where they come from.

**build scripts** - Changes within the `./shared` submodule happen in its [own repository](https://github.com/CircleCI-Public/cimg-shared).
For those changes to affect this image, the submodule needs to be updated.
Typically like this:

```bash
cd shared
git pull
cd ..
git add shared
git commit -m "Updating submodule for foo."
```

**parent image** - By design, when changes happen to a parent image, they don't appear in existing PHP images.
This is to aid in "determinism" and prevent breaking customer builds.
New PHP images will automatically pick up the changes.

If you *really* want to publish changes from a parent image into the PHP image, you have to build a specific image version as if it was a new image.
This will create a new Dockerfile and once published, a new image.

**PHP specific changes** - Editing the `Dockerfile.template` file in this repo is how to modify the PHP image specifically.
Don't forget that to see any of these changes locally, the `gen-dockerfiles.sh` script will need to be run again (see above).


## Contributing

We encourage [issues](https://github.com/CircleCI-Public/cimg-php/issues) to and [pull requests](https://github.com/CircleCI-Public/cimg-php/pulls) against this repository however, in order to value your time, here are some things to consider:

1. We won't include just anything in this image. In order for us to add a tool within the PHP image, it has to be something that is maintained and useful to a large number of PHP users. Every tool added makes the image larger and slower for all users so being thorough on what goes in the image will benefit everyone.
1. PRs are welcome. If you have a PR that will potentially take a large amount of time to make, it will be better to open an issue to discuss it first to make sure it's something worth investing the time in.
1. Issues should be to report bugs or request additional/removal of tools in this image. For help with images, please visit [CircleCI Discuss](https://discuss.circleci.com/c/ecosystem/circleci-images).


## Additional Resources

[CircleCI Docs](https://circleci.com/docs/) - The official CircleCI Documentation website.
[CircleCI Configuration Reference](https://circleci.com/docs/2.0/configuration-reference/#section=configuration) - From CircleCI Docs, the configuration reference page is one of the most useful pages we have.
It will list all of the keys and values supported in `.circleci/config.yml`.
[Docker Docs](https://docs.docker.com/) - For simple projects this won't be needed but if you want to dive deeper into learning Docker, this is a great resource.


## License

This repository is licensed under the MIT license.
The license can be found [here](./LICENSE).
