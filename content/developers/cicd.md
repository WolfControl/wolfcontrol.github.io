+++
title = "Continuous Integration/Deployment"
type = "docs"
weight = 2
+++

## Overview

As seen in [System Architecture](/developers/architecture), the WolfControl system is composed of various components. To ensure seamless integration and functionality, we employ continuous integration/continuous deployment (CI/CD) pipelines. These pipelines automate the building, testing, and deployment processes, ensuring that all components are up-to-date and operational.

## Firmware Libraries

The WolfControl system uses several shared firmware libraries across all devices. These libraries are updated frequently to add features, fix bugs, and improve performance. Our CI/CD pipeline ensures that all devices use the latest library versions by automatically updating firmware repositories when a new release is created.

The current process is as follows:

1. **Release Creation:** A new release of a library is manually created. (We aim to automate this in the future.)
2. **Notification:** All firmware repositories using the library are notified of the new release.
3. **Update Commit:** An automated commit updates the library version in the `platformio.ini` file of each firmware repository.
4. **Build Trigger:** The standard firmware build process is triggered, generating a new firmware binary with the updated library version.
5. **Release Creation:** Upon successful build, a new firmware release is created, versioned according to the library changes (major, minor, or patch).

## Firmware

Firmware repository changes are infrequent, as most functionality resides within shared libraries. Changes typically involve PlatformIO configurations or hardware-specific code. Currently, new builds are triggered on every commit to the main branch. We plan to transition to a development branch and pull requests for a more structured process.

## Backend Libraries

Backend services rely on shared libraries for database access, logging, and MQTT communication. While these libraries are not frequently updated, a CI/CD pipeline ensures all services use the latest versions.

The update process for backend libraries is similar to firmware libraries:

1. **Release Creation:** A new release of a library is manually created.
2. **Notification:** All backend service repositories using the library are notified of the new release.
3. **Update Commit:** An automated commit updates the library version in the `go.mod` file of each service repository.
4. **Build Trigger:** The standard build process is initiated.

## Backend

The backend build system is under development. Currently, changes are committed directly to the main branch of each service repository, triggering the following build process:

1. **Build:** The Go binary is built using the latest commit from the main branch.
2. **Recompile:** If successful, the binary is recompiled within a lean Docker container.
3. **Versioning:** The version is determined based on the latest commit message.
4. **Push to DockerHub:** The container is pushed to DockerHub.
5. **Release Creation:** A new release is created for the service repository.

## Documentation

Code documentation is built using Hugo, Docsy, and Dart Sass. API documentation is generated with Redoc, and all documentation is hosted on GitHub Pages.

Documentation updates are triggered on push to the `main` branch of the documentation repository, following these steps in a cloud-hosted runner:

1. **Checkout:** Checkout the documentation repository.
2. **Install Dependencies:** Install Node, Redoc CLI, Hugo, and Dart Sass.
3. **Generate API Documentation:** Generate API documentation.
4. **Configure:** Configure GitHub Pages.
5. **Build:** Build the site.
6. **Deploy:** Deploy the site to GitHub Pages.
