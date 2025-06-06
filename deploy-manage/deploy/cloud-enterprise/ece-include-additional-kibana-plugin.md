---
navigation_title: Include additional {{kib}} plugins
mapped_pages:
  - https://www.elastic.co/guide/en/cloud-enterprise/current/ece-include-additional-kibana-plugin.html
applies_to:
  deployment:
    ece: all
products:
  - id: cloud-enterprise
---

# Include additional {{kib}} plugins [ece-include-additional-kibana-plugin]

In certain cases you may choose to expand the {{kib}} Docker image included in an {{stack}} pack to include one or more additional plugins that are not bundled in the image by default. Plugins can extend the features included in {{kib}}, for example to include specialized visualizations. Adding plugins allows you to tailor your ECE deployments that include {{kib}} to suit your specific use cases.

The process involves two main steps:

1. [Extend an existing {{kib}} Docker image to include the additional plugins.](#ece-create-modified-docker-image)
2. [Update the {{stack}} pack included in your ECE installation to point to your modified Docker image.](#ece-modify-stack-pack)


## Before you begin [ece_before_you_begin_5]

Note the following restrictions:

* These instructions have been tested for {{stack}} versions starting with 6.7.0 and may not work for earlier versions.
* Plugins that you bundle yourself to be included in the {{stack}} are not covered by Elastic Customer Support and include no guarantee from Elastic.
* After uploading a modified version of an {{stack}} pack, if you reapply the original stack the changes will be lost and new {{kib}} instances will use the original Docker image provided by Elastic.
* The Dockerfile used in this example includes an optimization process that is relatively expensive and may require a machine with several GB of RAM to run successfully.


## Extend a {{kib}} Docker image to include additional plugins [ece-create-modified-docker-image]

This example runs a Dockerfile to install the [analyze_api_ui plugin](https://github.com/johtani/analyze-api-ui-plugin) or [kibana-enhanced-table](https://github.com/fbaligand/kibana-enhanced-table) into different versions of {{kib}} Docker image. The contents of the Dockerfile varies depending on the version of the {{stack}} pack that you want to modify.

1. Choose a directory on your ECE installation and save the Dockerfile code for your {{stack}} version as a file named  `Dockerfile`.

    ```sh
    FROM docker.elastic.co/cloud-release/kibana-cloud:8.13.1
    MAINTAINER Cloud Developers <cloud-pioneer@elastic.co>

    RUN /usr/share/kibana/bin/kibana-plugin install https://github.com/fbaligand/kibana-enhanced-table/releases/download/v1.14.0/enhanced-table-1.14.0_8.13.1.zip
    ```

2. Update the Dockerfile for your specific use case by changing these settings:

    * The maintainer
    * The version of the image
    * The plugin name and version number

        ::::{important}
        When you modify a {{kib}} Docker image, make sure you maintain the original image structure and only add the additional plugins.
        ::::

3. Build the modified Docker image, specifying an image name and version number. If you are using your own Docker repository, the `docker.elastic.co/cloud-assets` section must match your specific configuration. The image build process can take several minutes.

    ```sh
    docker build . -t docker.elastic.co/cloud-assets/kibana-with-plugin:8.13.1
    ```

4. If you have your own Docker repository, you can [push the modified Docker image to your repository](ece-install-offline-no-registry.md). Otherwise, run the following commands to compress and load the image into Docker:

    1. Create a .tar file of the Docker image, specifying the image name and version number:

        ```sh
        docker save -o kibana.8.13.1.tar docker.elastic.co/cloud-assets/kibana-with-plugin:8.13.1
        ```

    2. Copy the .tar file to a location on your network where it is available to each ECE host. Alternatively, you can copy the .tar file to each host directly. A third option is to run the previous steps on each host to create the modified Docker image and .tar file.
    3. On each host, load the image into Docker, where `FILE_PATH` is the location of your modified Docker image:

        ```sh
        docker load < FILE_PATH/kibana.8.13.1.tar
        ```



## Modify the {{stack}} pack to point to your modified image [ece-modify-stack-pack]

Follow these steps to update the {{stack}} pack zip files in your ECE setup to point to your modified Docker image:

1. Download to a local directory the [{{stack}} pack](manage-elastic-stack-versions.md) that you want to modify.
2. Save the following bash script with the name `change-kibana-image.sh`:

    ```sh
    #!/usr/bin/env bash

    set -eo pipefail

    # Repack a stackpack to modify the Kibana image it points to

    NO_COLOR='\033[0m'
    ERROR_COLOR='\033[1;31m'
    WARN_COLOR='\033[0;33m'
    ERROR="${ERROR_COLOR}[ERROR]${NO_COLOR}"
    WARNING="${WARN_COLOR}[WARNING]${NO_COLOR}"

    if [[ -z "$1" ]]; then
        echo -e "$ERROR Missing required stackpack argument"
        exit 1
    fi

    if [[ -z "$2" ]]; then
        echo -e "$ERROR Missing required kibana docker image argument"
        exit 1
    fi

    STACKPACK=$1
    KIBANA_IMAGE=$2

    if [[ ! -s "$STACKPACK" ]]; then
        echo -e "$ERROR $STACKPACK: No such stackpack"
        exit 1
    fi

    TMP_DIR=$(mktemp -d)

    TMP_FILE="$TMP_DIR/$(basename "$STACKPACK")"
    cp "$STACKPACK" "$TMP_FILE"

    pushd "$TMP_DIR" > /dev/null

    MANIFEST=$(zipinfo -1 "$TMP_FILE" | grep -E "stack.*\.json")
    unzip "$TMP_FILE" "$MANIFEST" > /dev/null

    jq ".kibana.docker_image |= \"${KIBANA_IMAGE}\"" "$MANIFEST" > "${MANIFEST}.updated"
    mv "${MANIFEST}.updated" "$MANIFEST"
    zip "$TMP_FILE" "$MANIFEST" > /dev/null

    popd > /dev/null

    cp "$TMP_FILE" "$STACKPACK"

    rm -rf "$TMP_DIR"
    ```

3. Modify the script permissions so that you can run it:

    ```sh
    sudo chmod 755 change-kibana-image.sh
    ```

4. Run the script to update the {{stack}} pack, where `FILE_PATH` is the location where you downloaded the {{stack}} pack zip file:

    ```sh
    ./change-kibana-image.sh FILE_PATH/8.13.1.zip docker.elastic.co/cloud-assets/kibana-with-plugin:8.13.1
    ```

5. Upload the modified {{stack}} pack to your ECE installation:

    1. [Log into the Cloud UI](log-into-cloud-ui.md).
    2. Go to **Platform** and then **{{stack}}**.
    3. Select **Upload {{stack}} pack** to add the new {{stack}} pack or replace an existing one. You can create a new deployment using the new or updated {{stack}} pack. When you launch {{kib}} the additional plugin is available.



## Common causes of problems [ece-custom-plugin-problems]

1. If the custom Docker image is not available, make sure that the image has been uploaded to your Docker repository or loaded locally onto each ECE allocator.
2. If the container takes a long time to start, the problem might be that the `reoptimize` step in the Dockerfile did not complete successfully.

