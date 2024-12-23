---
title: "Create a Hello World module"
linkTitle: "Hello World module"
type: "docs"
weight: 24
images: ["/registry/module-puzzle-piece.svg"]
icon: true
tags: ["modular resources", "components", "services", "registry"]
description: "Get started writing your own modular resources by creating a Hello World module."
languages: ["python", "go"]
viamresources: ["components"]
platformarea: ["registry"]
level: "Beginner"
date: "2024-10-22"
# updated: ""  # When the tutorial was last entirely checked
cost: "0"
---

This guide will walk you through creating a {{< glossary_tooltip term_id="modular-resource" text="modular" >}} camera component that responds to API calls by returning a configured image.
This guide also includes optional steps to create a modular sensor that returns random numbers, to demonstrate how you can include two modular resources within one {{< glossary_tooltip term_id="module" text="module" >}}.
By the end of this guide, you will be able to create your own modular resources and package them into modules so you can use them on your machines.

{{% alert title="In this page" color="tip" %}}

1. [Create a test script](#create-a-test-script)
1. [Choose an API](#choose-an-api-to-implement)
1. [Generate code stub files](#generate-stub-files)
1. [Implement the API methods](#implement-the-api-methods)
1. [Test your module](#test-your-module)
1. [Package and upload the module](#package-and-upload-the-module)

{{% /alert %}}

## Prerequisites

{{< expand "Install the Viam CLI and authenticate" >}}
Install the Viam CLI and authenticate to Viam, from the same machine that you intend to upload your module from.

{{< readfile "/static/include/how-to/install-cli.md" >}}

Authenticate your CLI session with Viam using one of the following options:

{{< readfile "/static/include/how-to/auth-cli.md" >}}
{{< /expand >}}

{{% expand "Install viam-server on your computer and connect to the Viam app" %}}

{{% snippet "setup.md" %}}

{{% /expand%}}

## Create a test script

The point of creating a module is to add functionality to your machine.
For the purposes of this guide, you're going to make a module that does two things: It opens an image file from a configured path on your machine, and it returns a random number.

1.  Find an image you'd like to display when your program runs.
    We used [this image of a computer with "hello world" on the screen](https://unsplash.com/photos/a-laptop-computer-sitting-on-top-of-a-wooden-desk-8q6e5hu3Ilc).
    Save the image to your computer.

1.  Create a test script on your computer and copy the following code into it:

    {{< tabs >}}
    {{% tab name="Python" %}}

```python {class="line-numbers linkable-line-numbers"}
# test.py opens an image and prints a random number
from PIL import Image
import random

# TODO: Replace path with path to where you saved your photo
photo = Image.open("/Users/jessamyt/Downloads/hello-world.jpg")

photo.show()

number = random.random()

print("Hello, World! The latest random number is ", number, ".")
```

{{% /tab %}}

<!-- TODO add back in when generator supports Go

```go {class="line-numbers linkable-line-numbers"}
// test.go opens an image and prints a random number
package main

import "fmt"
import "math/rand"

func main() {

  number := rand.Float64()
  fmt.Println("Hello, World! The latest random number is ", number, ".")

  // TODO
}
```

-->

{{< /tabs >}}

1.  Replace the path in the script above with the path to where you saved your photo.
    Save the script.

1.  Run the test script in your terminal:

    {{< tabs >}}
    {{% tab name="Python" %}}

It's best practice to use a virtual environment for running Python scripts.
You'll also need to install the dependency Pillow in the virtual environment before running the test script.

```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
python3 -m venv .venv
source .venv/bin/activate
pip install Pillow
python3 test.py
```

{{% /tab %}}
{{< /tabs >}}

    The image you saved should open on your screen, and a random number should print to your terminal.

1.  In later steps, the module generator will create a new virtual environment with required dependencies, so you can deactivate the one you just ran the test script in:

    ```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
    deactivate
    ```

## Choose an API to implement

Now it's time to decide which Viam [APIs](/appendix/apis/#component-apis) make sense for your module.
You need a way to return an image, and you need a way to return a number.

If you look at the [camera API](/appendix/apis/components/camera/), you can see the `GetImage` method, which returns an image.
That will work for the image.
None of the camera API methods return a number though.

Look at the [sensor API](/appendix/apis/components/sensor/), which includes the `GetReadings` method.
You can return a number with that, but the sensor API can't return an image.

Your module can contain multiple modular resources, so let's make two modular resources: a camera to return the image, and a sensor to return a random number.

{{% alert title="Note" color="note" %}}

For a quicker hello world experience, you can skip the sensor and only create a camera modular resource.
If you prefer the simpler path, skip the sensor sections in the steps below.

{{% /alert %}}

## Generate stub files

The easiest way to generate the files for your module is to use the [Viam CLI](/cli/).

### Generate the camera files

The CLI module generator generates the files for one modular resource at a time.
First let's generate the camera component files, and we'll add the sensor code later.

1.  Run the `module generate` command in your terminal:

    ```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
    viam module generate
    ```

1.  Follow the prompts, selecting the following options:

    - Module name: `hello-world`
    - Language: Your choice
    - Visibility: `Private`
    - Namespace/Organization ID:
      - In the [Viam app](https://app.viam.com), navigate to your organization settings through the menu in upper right corner of the page.
        Find the **Public namespace** and copy that string.
        In the example snippets below, the namespace is `jessamy`.
    - Resource to add to the module (API): `Camera Component`.
      We will add the sensor later.
    - Model name: `hello-camera`
    - Enable cloud build: `No`
    - Register module: `No`

1.  Hit your Enter key and the generator will generate a folder called <file>hello-world</file> containing stub files for your modular camera component.

### Generate the sensor code

{{< expand "Click if you are also creating a sensor component" >}}

Some of the code you just generated is shared across the module no matter how many modular resource models it supports.
Some of the code you generated is camera-specific.
You need to add some sensor-specific code to support the sensor component.

1.  Instead of writing the code manually, use the module generator again.

    ```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
    viam module generate
    ```

1.  You're going to delete this module after copy-pasting the sensor-specific code from it.
    The only things that matter are the API and the model name.

    - Module name: `temporary`
    - Language: Your choice
    - Visibility: `Private`
    - Namespace/Organization ID: Same as you used before.
    - Resource to add to the module (API): `Sensor Component`.
    - Model name: `hello-sensor`
    - Enable cloud build: `No`
    - Register module: `No`

1.  Open <file>temporary/src/main.py</file>.
    Copy the sensor class definition, from `class HelloSensor(Sensor, EasyResource)` through the `get_readings()` function definition (lines 15-65).

    Open the <file>hello-world/src/main.py</file> file you generated earlier, and paste the sensor class definition in after the camera class definition, above `if __name__ == "__main__":`.

1.  Change `temporary` to `hello-world` in the ModelFamily line, so you have, for example:

    ```python {class="line-numbers linkable-line-numbers" data-start="94" }
    MODEL: ClassVar[Model] = Model(ModelFamily("jessamy", "hello-world"), "hello-sensor")
    ```

1.  Add the imports that are unique to the sensor file:

    ```python {class="line-numbers linkable-line-numbers"}
    from viam.components.sensor import *
    from viam.utils import SensorReading
    ```

    Save the <file>hello-world/src/main.py</file> file.

1.  Open <file>temporary/meta.json</file> and copy the model information.
    For example:

    ```json {class="line-numbers linkable-line-numbers" data-start="8"}
    {
      "api": "rdk:component:sensor",
      "model": "jessamy:temporary:hello-sensor"
    }
    ```

1.  Open <file>hello-world/meta.json</file> and paste the sensor model into the model list.

    Edit the `description` to accurately include both models.

    Change `temporary` to `hello-world`.

    The file should now resemble the following:

    ```json {class="line-numbers linkable-line-numbers" data-line="6-16"}
    {
      "$schema": "https://dl.viam.dev/module.schema.json",
      "module_id": "jessamy:hello-world",
      "visibility": "private",
      "url": "",
      "description": "Example camera and sensor components: hello-camera and hello-sensor",
      "models": [
        {
          "api": "rdk:component:camera",
          "model": "jessamy:hello-world:hello-camera"
        },
        {
          "api": "rdk:component:sensor",
          "model": "jessamy:hello-world:hello-sensor"
        }
      ],
      "entrypoint": "./run.sh",
      "first_run": ""
    }
    ```

1.  You can now delete the <file>temporary</file> module directory and all its contents.

{{< /expand >}}

## Implement the API methods

Edit the stub files to add the logic from your test script in a way that works with the camera and sensor APIs:

{{< tabs >}}
{{% tab name="Python" %}}

### Implement the camera API

First, implement the camera API methods by editing the camera class definition:

1. Add the following to the list of imports at the top of <file>hello-world/src/main.py</file>:

   ```python {class="line-numbers linkable-line-numbers"}
   from viam.media.utils.pil import pil_to_viam_image
   from viam.media.video import CameraMimeType
   from viam.utils import struct_to_dict
   from PIL import Image
   ```

1. In the test script you hard-coded the path to the image.
   For the module, let's make the path a configurable attribute so you or other users of the module can set the path from which to get the image.
   Add the following lines to the camera's `reconfigure()` function definition.
   These lines set the `image_path` based on the configuration when the resource is configured or reconfigured.

   ```python {class="line-numbers" data-start="68"}
   attrs = struct_to_dict(config.attributes)
   self.image_path = str(attrs.get("image_path"))
   ```

1. We are not providing a default image but rely on the end user to supply a valid path to an image when configuring the resource.
   This means `image_path` is a required attribute.
   Add the following code to the `validate()` function to throw an error if `image_path` isn't configured:

   ```python {class="line-numbers linkable-line-numbers" data-start="57"}
   # Check that a path to get an image was configured
   fields = config.attributes.fields
   if not "image_path" in fields:
       raise Exception("Missing image_path attribute.")
   elif not fields["image_path"].HasField("string_value"):
       raise Exception("image_path must be a string.")
   ```

1. The module generator created a stub for the `get_image()` function we want to implement:

   ```python {class="line-numbers linkable-line-numbers" data-start="79" }
    async def get_image(
        self,
        mime_type: str = "",
        *,
        extra: Optional[Dict[str, Any]] = None,
        timeout: Optional[float] = None,
        **kwargs
    ) -> ViamImage:
        raise NotImplementedError()
   ```

   You need to replace `raise NotImplementedError()` with code to actually implement the method:

   ```python {class="line-numbers linkable-line-numbers" data-start="86" }
   ) -> ViamImage:
       img = Image.open(self.image_path)
       return pil_to_viam_image(img, CameraMimeType.JPEG)
   ```

   You can leave the rest of the functions not implemented, because this module is not meant to return a point cloud (`get_point_cloud()`), and does not need to return multiple images simultaneously (`get_images()`).

   Save the file.

1. Open <file>requirements.txt</file>.
   Add the following line:

   ```text
   Pillow
   ```

### Implement the sensor API

{{< expand "Click if you are also creating a sensor component" >}}

Now edit the sensor class definition to implement the sensor API.
You don't need to edit any of the validate or configuration methods because you're not adding any configurable attributes for the sensor model.

1. Add `random` to the list of imports in <file>main.py</file> for the random number generation:

   ```python {class="line-numbers linkable-line-numbers"}
   import random
   ```

1. The sensor API only has one resource-specific method, `get_readings()`:

   ```python {class="line-numbers linkable-line-numbers" data-start="156" }
    async def get_readings(
        self,
        *,
        extra: Optional[Mapping[str, Any]] = None,
        timeout: Optional[float] = None,
        **kwargs
    ) -> Mapping[str, SensorReading]:
        raise NotImplementedError()
   ```

   Replace `raise NotImplementedError()` with the following code:

   ```python {class="line-numbers linkable-line-numbers" data-start="162" }
    ) -> Mapping[str, SensorReading]:
        number = random.random()
        return {
            "random_number": number
        }
   ```

   Save the file.

{{< /expand >}}

{{% /tab %}}
{{< /tabs >}}

## Test your module

With the implementation written, it's time to test your module locally:

1. Create a virtual Python environment with the necessary packages by running the setup file from within the <file>hello-world</file> directory:

   ```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
   sh setup.sh
   ```

   This environment is where the local module will run.
   `viam-server` does not need to run inside this environment.

1. Make sure your machine's instance of `viam-server` is live and connected to the [Viam app](https://app.viam.com).

1. In the Viam app, navigate to your machine's **CONFIGURE** page.

1. Click the **+** button, select **Local module**, then again select **Local module**.

1. Enter the path to the automatically-generated <file>run.sh</file> file, for example, `/Users/jessamyt/myCode/hello-world/run.sh`.
   Click **Create**.

1. Now add the modular camera resource provided by the module:

   Click **+**, click **Local module**, then click **Local component**.

   For the {{< glossary_tooltip term_id="model-namespace-triplet" text="model namespace triplet" >}}, enter `<namespace>:hello-world:hello-camera`, replacing `<namespace>` with the organization namespace you used when generating the stub files.
   For example, `jessamy:hello-world:hello-camera`.

   For type, enter `camera`.

   For name, you can use the automatic `camera-1`.

1. Configure the image path attribute by pasting the following in place of the `{}` brackets:

   ```json {class="line-numbers linkable-line-numbers"}
   {
     "image_path": "<replace with the path to your image>"
   }
   ```

   Replace the path with the path to your image, for example `"/Users/jessamyt/Downloads/hello-world.jpg"`.

1. Save the config, then click the **TEST** section of the camera's configuration card.

   ![The Viam app configuration interface with the Test section of the camera card open, showing a hello world image.](/how-tos/hello-camera.png)

   You should see your image displayed.
   If not, check the **LOGS** tab for errors.

{{< expand "Click if you also created a sensor component" >}}

9. Add the modular sensor:

   Click **+**, click **Local module**, then click **Local component**.

   For the {{< glossary_tooltip term_id="model-namespace-triplet" text="model namespace triplet" >}}, enter `<namespace>:hello-world:hello-sensor`, replacing `<namespace>` with the organization namespace you used when generating the stub files.
   For example, `jessamy:hello-world:hello-sensor`.

   For type, enter `sensor`.

   For name, you can use the automatic `sensor-1`.

10. Save the config, then click **TEST** to see a random number generated every second.

    ![The sensor card test section open.](/how-tos/hello-sensor.png)

{{< /expand >}}

## Package and upload the module

You now have a working local module.
To make it available to deploy on more machines, you can package it and upload it to the [Viam Registry](https://app.viam.com/registry).

The hello world module you created is for learning purposes, not to provide any meaningful utility, so we recommend making it available only to machines within your {{< glossary_tooltip term_id="organization" text="organization" >}} instead of making it publicly available.

{{< expand "Click to see what you would do differently if this wasn't just a hello world module" >}}

1. Create a GitHub repo with all the source code for your module.
   Add the link to that repo as the `url` in the <file>meta.json</file> file.
1. Create a README to document what your module does and how to configure it.
1. If you wanted to share the module outside of your organization, you'd set `"visibility": "public"` in the <file>meta.json</file> file.

{{< /expand >}}

To package and upload your module and make it available to configure on machines in your organization:

1. Package the module as an archive, run the following command from inside the <file>hello-world</file> directory:

   ```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
   tar -czf module.tar.gz run.sh setup.sh requirements.txt src
   ```

   This creates a tarball called <file>module.tar.gz</file>.

1. Run the `viam module upload` CLI command to upload the module to the registry:

   ```sh {id="terminal-prompt" class="command-line" data-prompt="$"}
   viam module upload --version 1.0.0 --platform any module.tar.gz
   ```

1. Now, if you look at the [Viam Registry page](https://app.viam.com/registry) while logged into your account, you'll be able to find your private module listed.
   You can configure the hello-sensor and hello-camera on your machines just as you would configure other components and services; there's no more need for local module configuration.

   ![The create a component menu open, searching for hello. The hello-camera and hello-sensor components are shown in the search results.](/how-tos/hello-config.png)

For more information about uploading modules, see [Upload a module](/how-tos/upload-module/).

## Next steps

For a guide that walks you through creating different sensor models, for example to get weather data from an online source, see [Create a sensor module with Python](/how-tos/sensor-module/).

For more module creation information with more programming language options, see the [Create a module](/how-tos/create-module/) guide.

To update or delete a module, see [Update and manage modules](/how-tos/manage-modules/).
