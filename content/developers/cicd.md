+++
title = "Continuous Integration/Deployment"
type = "docs"
weight = 10
+++

## Overview

As we saw in [System Architecture](/developers/architecture), the WolfControl system is made up of many different components. To ensure that all these components work together seamlessly, we use a system of continuous integration/continuous deployment (CI/CD) pipelines. This automates the process of building, testing, and deploying changes to the system, ensuring that everything is always up-to-date and working as expected.

## Firmware Libraries

A number of firmware libraries are shared across all devices in the WolfControl system. These libraries are updated frequently to add new features, fix bugs, and improve performance. To ensure that all devices are using the latest versions of these libraries, we use a CI/CD pipeline to automatically update the firmware repositories whenever a new release is created.

Currently, changes are made directly in the main branch of each library. Development firmware is built using the latest commit from each library, while production firmware is built using the latest release.

The process works as follows:

1. A new release of a library is created. (Releases are manually created right now, but will be moved towards automation in the future.)
2. All repositories for firmware using that library are notified of the new release.
3. A new commit is automatically made to each firmware repository, updating the library version in the `platformio.ini` file.
4. The standard firmware build process is triggered, creating a new firmware binary with the updated library version.
5. Upon successful build, a new release is created for the firmware repository, versioned based on the changes in the library being major, minor, or patch.

## Firmware

Changes to the firmware repositories are infrequent, as most functionality is encapsulated within the shared libraries. When changes are made, they are typically limited to modifications in the PlatformIO configurations or hardware-specific code. Currently new builds are triggered on every commit to the main branch, but this will be replaced with a proper process utilizing a development branch and pull requests in the near future.

## Backend Libraries

Similar to the firmware libraries, the backend services rely on a number of shared libraries for things like database access, logging, and MQTT communication. These libraries are updated frequently, and a CI/CD pipeline is used to ensure that all services are using the latest versions.

The process for updating backend libraries is similar to that for firmware libraries:

1. A new release of a library is created manually.
2. All repositories for backend services using that library are notified of the new release.
3. A new commit is automatically made to each service repository, updating the library version in the `go.mod` file.
4. The standard build process is triggered.

## Backend

The build system for the backend is under development. Currently, changes are made directly in the main branch of each service repository. Commits trigger a build process outlined below:

1. The go binary is built using the latest commit from the main branch.
2. If successful, it is recompiled within a lean docker container.
3. Version is determined based on the latest commit message.
4. The container is pushed to dockerhub.
5. A new release is created for the service repository.

## Documentation

Code documentation is built using Hugo, Docsy, and Dart Sass. API documentation is generated using redoc, and all documentation is hosted on GitHub Pages.

Documentation is automatically updated on push to the `main` branch of the documentation repository. The following steps are taken in a cloud hosted runner:

1. Checkout the repository
2. Install Node and Redoc CLI
3. Generate API documentation
4. Install Hugo and Dart Sass
5. Configure Github Pages
6. Install all dependencies
7. Build the site
8. Deploy the site to Github Pages
