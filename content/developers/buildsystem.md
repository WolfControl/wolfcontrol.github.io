+++
title = "Build System"
type = "docs"
weight = 10
+++

## Libraries

To ensure consistency and ease of development, all core hardware functionality is implemented into libraries shared across all devices. These libraries are:

- esp-provisioning: Handles gateway and client lifecycle management.
- esp-messaging: Manages asynchronous messaging between devices.
- esp-led: Controls power and status LEDs.
- esp-sensor: Provides a multithreading wrapper for Atlas Scientific EZO sensors.

Changes are made directly to the main branch of each library. Development firmware is built using the latest commit from each library, while production firmware is built using the latest release. When a new release is created, firmware repositories are automatically updated to use the new version, triggering a new firmware release if the build is successful.

## Firmware

The WolfControl hardware is built around the ESP32-S3 microcontroller, with firmware written in C/C++ using the ESP-IDF framework. Direct changes to the firmware repositories are infrequent, as most functionality is encapsulated within the libraries, which are updated automatically. This approach allows us to keep the core business logic proprietary while maintaining public firmware repositories. Modifications to the firmware repositories are typically limited to changes in PlatformIO configurations or hardware-specific code.

To illustrate, here is the complete main.c file for the ESPNow Gateway:

```c
#include "main.h"

/* --------------------------- Main Loop --------------------------- */

void app_main(void)
{
    static const char *TAG = "MAIN";
    esp_err_t status;

    status = setupWolfESPNowGateway();
    if (status != ESP_OK) {
        ESP_LOGE(TAG, "Failed to initialize gateway: %s", esp_err_to_name(status));
        return;
    }

    ESP_LOGI(TAG, "Initialization complete. Waiting for incoming messages...");

}
```

Upon library update dispatch, the following steps are taken:

- Get name & version of library that was updated
- Check against library version currently in use to determine if change is major, minor, or patch
- Update library in `platformio.ini` file
- Commit changes to firmware repository
- Determine new firmware version based on library changes
- Update firmware version variable in `main.h(pp)`
- Build firmware
- Create release with firmware binary

## Backend

The backend controller for the WolfControl system is a Raspberry Pi 5 running Docker. A repo containing a docker-compose file and db configs is cloned onto the Pi, and roughly 10 services are started in containers. The backend is written in Go, and the frontend is vanilla JS.

Pushes to the main branch of the wolfcontroller or app-web respositories trigger a build pipeline. The following steps are taken:

- Determine version from commit message (Must contain (MAJOR), (MINOR), or (PATCH))
- Setup Go environment (Controller only)
- Build Go binary (Controller only)
- Login to Docker Hub
- Build lean Docker image
- Push Docker image to Docker Hub
- Create GitHub release

## Documentation

Code documentation is built using Hugo, Docsy, and Dart Sass. API documentation is generated using redoc, and all documentation is hosted on GitHub Pages.

Documentation is automatically updated on push to the `main` branch of the documentation repository. The following steps are taken:

- Install Node and Redocly CLI
- Build static API documentation using redoc-cli
- Install Hugo CLI and Dart Sass
- Configure github pages settings
- Install dependencies
- Build static site with Hugo
- Deploy to GitHub Pages
