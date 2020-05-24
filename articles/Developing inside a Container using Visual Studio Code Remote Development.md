Developing inside a Container using Visual Studio Code Remote Development

[(L)](https://github.com/Microsoft/vscode-docs/blob/master/docs/remote/containers.md)

# Developing inside a Container

❗ **Note:** The **[Remote Development extensions](https://aka.ms/vscode-remote/download)** require **[Visual Studio Code Insiders](https://code.visualstudio.com/insiders)**.

* * *

The **Visual Studio Code Remote - Containers** extension lets you use a [Docker container](https://docker.com/) as a full-featured development environment. It allows you to open any folder inside (or mounted into) a container and take advantage of VS Code's full feature set. A [`devcontainer.json` file](https://code.visualstudio.com/docs/remote/containers#_creating-a-devcontainerjson-file) in your project tells VS Code how to access (or create) a **development container** with a well-defined tool and runtime stack. This container can be used to run an application or to sandbox tools, libraries, or runtimes needed for working with a codebase.

Workspace files are mounted from the local file system or copied or cloned into the container. Extensions are installed and run inside the container, where they have full access to the tools, platform, and file system. This means that you can seamlessly switch your entire development environment just by connecting to a different container.

![](../_resources/8d8a6f5ba07276e7fcb2705ef6fe4f05.png)

This lets VS Code provide a **local-quality development experience** — including full IntelliSense (completions), code navigation, and debugging — **regardless of where your tools (or code) is located**.

## Getting started

### Installation

To get started, follow these steps:

1. Install and configure [Docker](https://www.docker.com/get-started) for your operating system.

**Windows / macOS**:

    1. Install [Docker Desktop for Windows/Mac](https://www.docker.com/products/docker-desktop). (Docker Toolbox is not currently supported.)

    2. Right-click on the Docker taskbar item and update **Settings / Preferences > Shared Drives / File Sharing** with any source code locations you want to open in a container. If you run into trouble, see [Docker Desktop for Windows tips](https://code.visualstudio.com/docs/remote/troubleshooting#_docker-desktop-for-windows-tips) on avoiding common problems with sharing.

    3. **Windows**: Consider adding a `.gitattributes` file or disabling automatic line ending conversion for Git on the **Windows side** by using a command prompt to run: `git config --global core.autocrlf false` If left enabled, this setting can cause files that you have not edited to appear modified due to line ending differences. See [tips and tricks](https://code.visualstudio.com/docs/remote/troubleshooting#_resolving-git-line-ending-issues-in-containers-resulting-in-many-modified-files) for details.

**Linux**:

    1. Follow the [install instructions for your Linux distribution](https://docs.docker.com/install/#supported-platforms). **Note**: The Ubuntu Snap package is not supported.

    2. Add your user to the `docker` group by using a terminal to run: `sudo usermod -aG docker $USER`

    3. Sign out and back in again so your changes take effect.

2. Install [Visual Studio Code Insiders](https://code.visualstudio.com/insiders/).

3. Install the [Remote Development](https://aka.ms/vscode-remote/download/extension) extension pack.

The Remote - Containers extension supports two primary operating models:

- You can use a container as your [full-time development environment](https://code.visualstudio.com/docs/remote/containers#_creating-a-devcontainerjson-file).
- You can [attach to a running container](https://code.visualstudio.com/docs/remote/containers#_attaching-to-running-containers) to inspect it.

We will cover how to use a container as your full-time development environment first.

### Quick start: Try a dev container

Let's start out by using a sample project to try things out.
1. Clone one of the sample repositories below.

	git clone https://github.com/Microsoft/vscode-remote-try-node
	git clone https://github.com/Microsoft/vscode-remote-try-python
	git clone https://github.com/Microsoft/vscode-remote-try-go
	git clone https://github.com/Microsoft/vscode-remote-try-java
	git clone https://github.com/Microsoft/vscode-remote-try-dotnetcore
	git clone https://github.com/Microsoft/vscode-remote-try-php
	git clone https://github.com/Microsoft/vscode-remote-try-rust
	git clone https://github.com/Microsoft/vscode-remote-try-cpp

2. Start VS Code and click on the quick actions Status Bar item in the lower left corner of the window.

![](../_resources/66c802c9bfdc64f97f116da2b6e4e1c6.png)

3. Select **Remote-Containers: Open Folder in Container...** from the command list that appears, and open the root folder of the project you just cloned.

4. The window will then reload, but since the container does not exist yet, VS Code will create one. This may take some time, and a progress notification will provide status updates. Fortunately, this step isn't necessary the next time you open the folder since the container will already exist.

![](../_resources/9539b345e955cd1c2e430f4e5a73d79c.png)

5. After the container is built, VS Code automatically connects to it and maps the project folder from your local file system into the container. Check out the **Things to try** section of `README.md` in the repository you cloned to see what to do next.

### Quick start: Open a folder in a container

Next we will cover how to set up a dev container for an existing project to use as your full-time development environment.

The steps are similar to those above:

1. Start VS Code, run the **Remote-Containers: Open Folder in Container...** command from the Command Palette, and select the project folder you'd like to set up the container for.

2. Now pick a starting point for your dev container. You can either select a base **dev container definition** from a filterable list, or use an existing [Dockerfile](https://docs.docker.com/engine/reference/builder/) or [Docker Compose file](https://docs.docker.com/compose/compose-file/#compose-file-structure-and-examples) if one exists in the folder you selected.

**> Note:**>  Alpine Linux and Windows based containers are not currently supported.

![](../_resources/3365fd91335f8710ce64a5317dd3e961.png)

Note the dev container definitions displayed come from the [vscode-dev-containers repository](https://aka.ms/vscode-dev-containers). You can browse the `containers` folder of that repository to see the contents of each definition.

3. After picking the starting point for your container, VS Code will add the dev container configuration files to your project (`.devcontainer/devcontainer.json`). The VS Code window will reload and start building the dev container. A progress notification provides you status updates. Note that you only have to build a dev container the first time you open it; opening the folder after the first successful build will be much quicker.

![](../_resources/9539b345e955cd1c2e430f4e5a73d79c.png)

4. After building completes, VS Code will automatically connect to the container. You can now interact with your project in VS Code just as you could when opening the project locally. From now on, when you open the project folder, VS Code will automatically pick up and reuse your dev container configuration.

## Creating a devcontainer.json file

VS Code's container configuration is stored in a [`devcontainer.json`](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference) file. This file is similar to the `launch.json` file for debugging configurations, but is used for launching (or attaching to) your development container instead. You can also specify any extensions to install once the container is running or post-create commands to prepare the environment. The dev container configuration is either located under `.devcontainer/devcontainer.json` or stored as a `.devcontainer.json` file (note the dot-prefix) in the root of your project.

The **Remote-Containers: Create Container Configuration File...** command adds the file to your project, where you can further customize for your needs.

For example, through a `devcontainer.json` file, you can:

- Spin up a [stand-alone "sandbox" container](https://code.visualstudio.com/docs/remote/containers#_working-with-a-developer-sandbox).
- Work inside a dev container defined by an [image](https://code.visualstudio.com/docs/remote/containers#_using-an-existing-container-image), [Dockerfile](https://code.visualstudio.com/docs/remote/containers#_using-a-dockerfile), or [docker-compose.yml](https://code.visualstudio.com/docs/remote/containers#_using-docker-compose).
- [Use Docker or Kubernetes](https://code.visualstudio.com/docs/remote/containers-advanced#_using-docker-or-kubernetes-from-a-container) from inside a dev container to build and deploy your app.
- [Attach to an already running container](https://code.visualstudio.com/docs/remote/containers#_attaching-to-running-containers).

The [vscode-dev-containers repository](https://aka.ms/vscode-dev-containers) has examples of `devcontainer.json` for different scenarios. You can [alter your configuration](https://code.visualstudio.com/docs/remote/containers#_indepth-setting-up-a-folder-to-run-in-a-container) to:

- Install additional tools such as Git in the container.
- Automatically install extensions.
- Expose additional ports.
- Set runtime arguments.
- Reuse or [extend your existing Docker Compose setup](https://aka.ms/vscode-remote/containers/docker-compose/extend).

### Configuration edit loop

Editing your container configuration is easy. Since rebuilding a container will "reset" the container to its starting contents (with the exception of your local source code), VS Code does not automatically rebuild if you edit a container configuration file (`devcontainer.json`, `Dockerfile`, `docker-compose.yml`). Instead, there are several commands that can be used to make editing your configuration easier.

Here is the typical edit loop using these commands:

1. Start with F1 > **Remote-Containers: Create Container Configuration File...**

2. Edit the contents of the `.devcontainer` folder as required.
3. Try it with F1 > **Remote-Containers: Reopen Folder in Container**.
4. On failure:

    1. F1 > **Remote-Containers: Reopen Folder Locally**, which will open a new local window.

    2. In this local window: Edit the contents of the `.devcontainer` folder as required.

    3. Try it again: Go back to the container window, F1 > **Developer: Reload Window**.

    4. Repeat as needed.
5. If the build was successful, but you want to make more changes:

    1. Edit the contents of the `.devcontainer` folder as required when connected to the container.

    2. F1 > **Remote-Containers: Rebuild Container**.
    3. On failure: Follow the same workflow above.

### Adding configuration files to public or private repositories

You can easily share a customized dev container definition for your project by adding `devcontainer.json` files to source control. By including these files in your repository, anyone that opens a local copy of your repo in VS Code will be automatically prompted to reopen the folder in a container, provided they have the Remote - Containers extension installed.

![](../_resources/12538623a88b30d76f8cb25461f8e28d.png)

Beyond the advantages of having your team use a consistent environment and tool-chain, this also makes it easier for new contributors or team members to be productive quickly. First-time contributors will require less guidance and hit fewer issues related to environment setup.

## Attaching to running containers

While using VS Code to spin up a new container can be useful in many situations, it may not match your workflow and you may prefer to "attach" VS Code to an already running container.

**> Note:**>  Alpine Linux and Windows based containers are not currently supported.

Once you have a container up and running, you can connect by either:

### Option 1: Use the Attach to Running Container command

Run **Remote-Containers: Attach to Running Container...** command from the Command Palette (F1) and selecting a container.

### Option 2: Use the Docker extension

1. Install the [Docker extension](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker) from the Extensions view (search for "docker") if it is not already installed.

2. Go to the Docker view and expand the **Containers** node in the explorer.
3. Right-click and select **Attach Visual Studio Code**.
![](../_resources/4c183a6832de4789eb21ca3cb310e617.png)

After a brief moment, a new window will appear and you'll be connected to the running container.

## Managing containers

By default, the Remote - Containers extension automatically starts the containers mentioned in the `devcontainer.json` when you open the folder. When you close VS Code, the extension automatically shuts down the containers you've connected to. You can change this behavior by adding `"shutdownAction": "none"` to the `devcontainer.json`.

You can also manually manage your containers using one of the following options:

### Option 1: Use the Docker extension

1. Install the [Docker extension](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker) from the Extensions view, if not already installed.

**> Note:**>  Some Docker commands invoked from the Docker extension can fail when invoked from a VS Code window opened in a container. Most containers do not have the Docker command line installed. Therefore commands invoked from the Docker extension that rely on the Docker command line, for example **> Docker: Show Logs**> , fail. If you need to execute these commands, open a new local VS Code window and use the Docker extension from this window or > [> set up Docker inside your container](https://aka.ms/vscode-remote/samples/docker-in-docker)> .

2. You can then go to the Docker view and expand the **Containers** node to see what containers are running. Right-click and select **Stop Container** to shut one down.

![](../_resources/843038ed3caf403226fefc4b93c1ccbb.png)

### Option 2: Use the Docker CLI

1. Open a **local** terminal/command prompt (or use a local window in VS Code).

2. Type `docker ps` to see running containers. Use `docker ps -a` to also see any stopped containers.

3. Type `docker stop <Container ID>` from this list to stop a container.

4. If you would like to delete a container, type `docker rm <Container ID>` to remove it.

### Option 3: Use Docker Compose

1. Open a **local** terminal/command prompt (or use a local window in VS Code).
2. Go to the directory with your `docker-compose.yml` file.
3. Type `docker-compose top` to see running processes.

4. Type `docker-compose stop` to stop the containers. If you have more than one Docker Compose file, you can specify additional Docker Compose files with the `-f` argument.

5. If you would like to delete the containers, type `docker-compose down` to both stop and delete them.

If you want to clean out images or mass-delete containers, [see here](https://code.visualstudio.com/docs/remote/troubleshooting#_cleaning-out-unused-containers-and-images) for different options.

## Managing extensions

VS Code runs extensions in one of two places: locally on the UI / client side, or in the container. While extensions that affect the VS Code UI, like themes and snippets, are installed locally, most extensions will reside inside a particular container. This allows you to install only the extensions you need for a given task in a container and seamlessly switch your entire tool-chain just by connecting to a new container.

If you install an extension from the Extensions view, it will automatically be installed in the correct location. You can tell where an extension is installed based on the category grouping. There will be a **Local - Installed** category and also one for your container.

![](../_resources/b021ba2c60348d573728ba5515dfce9a.png)
![](../_resources/cb7451059b188c540c34beebf6d9f0cd.png)

**> Note:**>  If you are an extension author and your extension is not working properly or installs in the wrong place, see > [> Supporting Remote Development](https://code.visualstudio.com/api/advanced-topics/remote-extensions)>  for details.

Local extensions that actually need to run remotely will appear **Disabled** in the **Local - Installed** category. You can click the **Install** button to install an extension on your remote host.

![](../_resources/39dbcd63895bd9d68cc14168c000dff4.png)

### "Always installed" extensions

If there are extensions that you would like always installed in any container, you can update the `remote.containers.defaultExtensions` User [setting](https://code.visualstudio.com/docs/getstarted/settings). For example, if you wanted to install the [GitLens](https://marketplace.visualstudio.com/itemdetails?itemName=eamodio.gitlens) and [Resource Monitor](https://marketplace.visualstudio.com/itemdetails?itemName=mutantdino.resourcemonitor) extensions, you would specify their extension IDs as follows:

	"remote.containers.defaultExtensions": [
	    "eamodio.gitlens",
	    "mutantdino.resourcemonitor"
	]

### Advanced: Forcing an extension to run locally / remotely

Extensions are typically designed and tested to either run locally or remotely, not both. However, if an extension supports it, you can force it to run in a particular location in your `settings.json` file.

For example, the setting below will force the Docker and Debugger for Chrome extensions to run remotely instead of their local defaults:

	"remote.extensionKind": {
	    "msjsdiag.debugger-for-chrome": "workspace",
	    "peterjausovec.vscode-docker": "workspace"
	}

A value of `"ui"` instead of `"workspace"` will force the extension to run on the local UI/client side instead. Typically, this should only be used for testing unless otherwise noted in the extension's documentation since it **can break extensions**. See the article on [Supporting Remote Development](https://code.visualstudio.com/api/advanced-topics/remote-extensions) for details.

## Forwarding a port

Containers are isolated environments, so if you want to access a server, service, or other resource inside your container, you will need to "forward" the port to your host. You can either configure your container to always expose these ports or just forward them temporarily.

### Temporarily forwarding a port

Sometimes when developing you may need to access a port in your container that you didn't add to `devcontainer.json`, your `Dockerfile`, or Docker Compose file. If you want to **temporarily forward** a new port for the duration of the session, run the **Remote-Containers: Forward Port from Container...** command from the Command Palette (F1).

After selecting a port, a notification will tell you the localhost port you should use to access the port in the container. For example, if you forwarded an HTTP server listening on port 3000, the notification may tell you that it was mapped to port 4123 on localhost. You can then connect to this remote HTTP server using `http://localhost:4123`.

### Always forwarding / exposing / publishing a port

If you have ports you always want use from your host, you can set them up so they are always available. Specifically you can:

1. **Use the appPort property:** If you reference an `image` or `Dockerfile` in `devcontainer.json`, you can use the `appPort` property to publish ports to the host.

	"appPort": [3000, "8921:5000"]

2. **Use the Docker Compose ports mapping:** The [`ports` mapping](https://docs.docker.com/compose/compose-file#ports) can easily be added your `docker-compose.yml` file to publish additional ports.

	ports:

	- "3000"
	- "8921:5000"

In each case, you'll need to rebuild your container for the setting to take effect. You can do this by running the **Remote-Containers: Rebuild Container** command in the Command Palette (F1) when you are connected to the container.

## Opening a terminal

Opening a terminal in a container from VS Code is simple. Once you've opened a folder in a container, **any terminal window** you open in VS Code (**Terminal > New Terminal**) will automatically run in the container rather than locally.

You can also use the `code-insiders` command line from this same terminal window to perform a number of operations such as opening a new file or folder in the container. Type `code-insiders --help` to learn what options are available from the command line.

![](../_resources/03aabc785c641dff6510b81f29549818.png)

## Debugging in a container

Once you've opened a folder in a container, you can use VS Code's debugger in the same way you would when running the application locally. For example, if you select a launch configuration in `launch.json` and start debugging (F5), the application will start on the remote host and attach the debugger to it.

See the [debugging](https://code.visualstudio.com/docs/editor/debugging) documentation for details on configuring VS Code's debugging features in `.vscode/launch.json`.

## Container specific settings

VS Code's local user settings are also reused when you are connected to a dev container. While this keeps your user experience consistent, you may want to vary some of these settings between your local machine and each container. Fortunately, once you have connected to a container, you can also set container-specific settings by running the **Preferences: Open Remote Settings** command from the Command Palette (F1) or by selecting the **Remote** tab in the settings editor. These will override any local settings you have in place whenever you connect to the container.

### Default container specific settings

You can also add a default container settings file into your container image. This can be useful for settings like absolute paths that you know will always differ from local ones.

For example, consider this `.devcontainer/settings.vscode.json` file that sets the Java home path:

	{
	    "java.home": "/docker-java-home"
	}

If you have an existing `.devcontainer/Dockerfile`, you can just add a `COPY` statement that puts the settings file in the right location:

	# Copy endpoint specific user settings into container to specify Java path
	COPY settings.vscode.json /root/.vscode-remote/data/Machine/settings.json

## In-depth: Setting up a folder to run in a container

There are a few different ways VS Code Remote - Containers can be used to develop an application inside a fully containerized environment. In general, there are two primary scenarios that drive interest in this development style:

- [Stand-Alone Dev Sandboxes](https://code.visualstudio.com/docs/remote/containers#_working-with-a-developer-sandbox): You may not be deploying your application into a containerized environment but still want to isolate your build and runtime environment from your local OS or develop in an environment that is more representative of production. For example, you may be running code on your local macOS or Windows machine that will ultimately be deployed to a Linux VM or server. You can reference an existing [image](https://code.visualstudio.com/docs/remote/containers#_using-an-existing-container-image) or a [Dockerfile](https://code.visualstudio.com/docs/remote/containers#_using-a-dockerfile) for this purpose.
- **Container Deployed Applications**: You deploy your application into one or more containers and would like to work locally in the containerized environment. VS Code currently supports working with container-based applications defined in a number of ways:
    - [Dockerfile](https://code.visualstudio.com/docs/remote/containers#_using-a-dockerfile): You are working on a single container / service that is described using a single `Dockerfile`.
    - [Docker Compose](https://code.visualstudio.com/docs/remote/containers#_using-docker-compose): You are working with multiple orchestrated services that are described using a `docker-compose.yml` file.
    - [Attach](https://code.visualstudio.com/docs/remote/containers#_attaching-to-running-containers): You can use an alternate workflow and attach to an already running container.
    - In each case, you may also need to [build container images and deploy to Docker or Kubernetes](https://code.visualstudio.com/docs/remote/containers#_using-docker-or-kubernetes-from-a-container) from inside your container.

This section will walk you through configuring your project for each of these situations. The [vscode-dev-containers GitHub repository](https://aka.ms/vscode-dev-containers) also contains dev container definitions to get you up and running quickly.

### Working with a developer sandbox

You can create a dev sandbox by selecting a base container image from a source like [DockerHub](https://hub.docker.com/) and then manually installing additional software, such as Git, which may be missing.

You can use the **Remote-Containers: Create Container Configuration File** command in the Command Palette (F1) to select from a base image to get you started and customize from there.

**> Note:**>  Alpine Linux and Windows based containers are not currently supported.

If you are not able to find an image that meets your needs or just want to automate the installation of additional software, you can also [create a custom image using a `Dockerfile`](https://code.visualstudio.com/docs/remote/containers#_using-a-dockerfile).

### Using an existing container image

You can use the `image` property in a `.devcontainer/devcontainer.json` in your project root to configure VS Code for use with an existing container image. The image should reside in a container registry ([DockerHub](https://hub.docker.com/), [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)) that VS Code can use to create the dev container.

For example:

	{
	    "name": "My Project",
	    "image": "microsoft/dotnet:sdk",
	    "appPort": 8090,
	    "extensions": [
	        "ms-vscode.csharp"
	    ]
	}

See the [devcontainer.json reference](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference) for information on other available properties such as the `appPort`, `runArgs`, and `extensions` list.

To open the folder in the container, run the **Remote-Containers: Open Folder in Container** or **Remote: Reopen Folder in Container** command from the Command Palette (F1).

Once the container has been created, the **local filesystem will be automatically mapped** into the container and you can start working with it from VS Code. You can also add additional local mount points to give your container access to other locations. The example below mounts your home / user profile folder into the container using the `runArgs` property and local environment variables:

	{
	    "name": "My Project",
	    "image": "microsoft/dotnet:sdk",
	    "appPort": 8090,
	    "extensions": [
	        "ms-vscode.csharp"
	    ],
	    "runArgs": [
	        "-v", "${env:HOME}${env:USERPROFILE}:/host-home-folder"
	    ]
	}

After making edits, run the **Remote-Containers: Rebuild Container** command for the updated settings to take effect.

### Installing additional software in the sandbox

Once VS Code is connected to the container, you can open a VS Code terminal and execute any command against the OS inside the container. This allows you to install new command-line utilities and spin up databases or application services from inside the Linux container.

Most container images are based on Debian or Ubuntu, where the `apt-get` command is used to install new packages.

For example:

	apt-get update # Critical step - you won't be able to install software before you do this
	apt-get install <package>

**> Note:**>  GUI based tools do not typically work inside of containers.

Documentation for the software you want to install will usually provide specific instructions, but note that you typically do **not need to prefix commands with `sudo`** given you are likely running as root in the container. If you are not already root, read the directions for the image you've selected to learn how to install additional software. If you would **prefer not to run as root**, see the [tips and tricks](https://code.visualstudio.com/docs/remote/containers-advanced#_adding-a-nonroot-user-to-your-dev-container) article for how to set up a separate user.

However, note that if you **rebuild** the container, you will have to **re-install** anything you've installed manually. To avoid this problem, you can use a `Dockerfile` to create a custom image with additional software pre-installed. We'll cover this scenario next.

### Using a Dockerfile

To create a customized sandbox or application in a single container, you can use (or reuse) a `Dockerfile` to define your dev container. If you have an existing `Dockerfile` you want to use, you can use the **Remote-Containers: Create Container Configuration File** command in the Command Palette (F1) where you'll be asked to pick which Dockerfile you want to use. You can then customize from there.

**> Note:**>  Alpine Linux and Windows based containers are not currently supported.

You may want to install other tools such as Git inside the container, which you can easily [do manually](https://code.visualstudio.com/docs/remote/containers#_installing-additional-software-in-the-sandbox). However, you can also create a custom Dockerfile specifically for development that includes these dependencies. The [vscode-dev-containers repository](https://github.com/Microsoft/vscode-dev-containers) contains examples you can use as a starting point.

You can use the `dockerFile` property in `.devcontainer/devcontainer.json` to specify the path to a custom `Dockerfile` and use the same properties available in the `image` case.

For example:

	{
	    "name": "My Node.js App",
	    "dockerFile": "Dockerfile",
	    "appPort": 3000,
	    "extensions": [
	        "dbaeumer.vscode-eslint"
	    ],
	    "postCreateCommand": "npm install"
	}

See the [devcontainer.json reference](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference) for information on other available properties such as `appPort`, `runArgs`, the `extensions` list, and `postCreateCommand`.

The example below uses `runArgs` to change the security policy to enable the ptrace system call for a Go development container:

	{
	    "name": "My Go App",
	    "dockerFile": "Dockerfile",
	    "extensions": [
	        "ms-vscode.go"
	    ],
	    "runArgs": [
	        "--cap-add=SYS_PTRACE",
	        "--security-opt",
	        "seccomp=unconfined" ]
	}

After making edits, you can run the **Remote-Containers: Reopen Folder in Container** or **Remote-Containers: Rebuild Container** commands to try things out. Once the container is created, the local filesystem is automatically mapped into the container and you can start working with it from VS Code.

### Using Docker Compose

In some cases, a single container environment isn't sufficient. Fortunately, VS Code Remote also works with multi-container configurations that are managed by `docker-compose.yml`.

You can either:
1. Reuse an existing `docker-compose.yml` unmodified.

2. Make a copy of your exiting `docker-compose.yml` that you use for development.

3. [Extend your existing Docker Compose configuration](https://code.visualstudio.com/docs/remote/containers#_extending-your-docker-compose-file-for-development) for development.

4. Use the command line (for example `docker-compose up`) and [attach to an already running container](https://code.visualstudio.com/docs/remote/containers#_attaching-to-running-containers).

**> Note:**>  Alpine Linux and Windows based containers are not currently supported.

VS Code can be configured to **automatically start any needed containers** for a particular service in a Docker Compose file (if they are not already running). This gives your multi-container workflow the same quick setup advantages described for the Docker image and Dockerfile flows above.

To reuse a `docker-compose.yml` unmodified, you can use the `dockerComposeFile` and `service` properties in a `.devcontainer/devcontainer.json`.

For example:

	{
	    "name": "[Optional] Your project name here",
	    "dockerComposeFile": "../docker-compose.yml",
	    "service": "the-name-of-the-service-you-want-to-work-with-in-vscode",
	    "workspaceFolder": "/default/workspace/path/in/container/to/open",
	    "shutdownAction": "stopCompose"
	}

If the containers are not already running, VS Code will call `docker-compose -f ../docker-compose.yml up` in this example. Note that the `service` property indicates which service in your Docker Compose file VS Code should connect to, not which service should be started.

See the [devcontainer.json reference](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference) for information other available properties such as the `workspaceFolder` and `shutdownAction`.

You can also create a development copy of your Docker Compose file. For example, if you had `.devcontainer/docker-compose.devcontainer.yml`, you would just change the following line in `devcontainer.json`:

	"dockerComposeFile": "docker-compose.devcontainer.yml"

You can also avoid making a copy of your Docker Compose file by extending it with another one. We'll cover this topic in the [next section](https://code.visualstudio.com/docs/remote/containers#_extending-your-docker-compose-file-for-development).

Note that, if you use Git, you may want to include a volume mount to your local `.gitconfig` folder in your Docker Compose file so you don't have to set up Git again inside of the container.

	volumes:
	  # This lets you avoid setting up Git again in the container

	  - ~/.gitconfig:/root/.gitconfig

If your application was built using C++, Go, or Rust, or another language that uses a ptrace-based debugger, you will also need to add the following settings to your Docker Compose file:

	# Required for ptrace-based debuggers like C++, Go, and Rust
	cap_add:

	- SYS_PTRACE

	security_opt:

	- seccomp:unconfined

After making edits, you can test by running the **Remote-Containers: Reopen Folder in Container** or **Remote-Containers: Rebuild Container** commands. Once the container is been created, the local filesystem is automatically mapped into the container and you can start working with it from VS Code.

### Extending your Docker Compose file for development

Referencing an existing deployment / non-development focused `docker-compose.yml` has some potential downsides.

For example:

- Docker Compose will shut down a container if its entry point shuts down. This is problematic for situations where you are debugging and need to restart your app on a repeated basis.
- You also may not be mapping the local filesystem into the container or exposing ports to other resources like databases you want to access.
- You may not want to add a `.gitconfig` mount or the ptrace options [described above](https://code.visualstudio.com/docs/remote/containers#_using-docker-compose) into your existing Docker Compose file.
- You may be using an [Alpine Linux](https://alpinelinux.org/) based image in your production configuration. (VS Code Remote - Containers does not currently support Alpine Linux).

You can solve these and other issues like them by extending your entire Docker Compose configuration with [multiple `docker-compose.yml` files](https://docs.docker.com/compose/extends/#multiple-compose-files) that override or supplement your primary one.

For example, consider this additional `.devcontainer/docker-compose.extend.yml` file:

	version: '3'
	  services:
	    your-service-name-here:
	      volumes:
	        # Mounts the project folder to '/workspace'. The target path inside the container
	        # should match what your application expects. In this case, the compose file is
	        # in a sub-folder, so we will mount '..'. We'll then reference this as the
	        # workspaceFolder in '.devcontainer/devcontainer.json' so VS Code starts here.

	        - ..:/workspace

	        # This lets you avoid setting up Git again in the container

	        - ~/.gitconfig:/root/.gitconfig

	      # [Optional] Required for ptrace-based debuggers like C++, Go, and Rust
	      cap_add:

	        - SYS_PTRACE

	      security_opt:

	        - seccomp:unconfined

	      # Overrides default command so things don't shut down after the process ends.
	      command: sleep infinity

This same file can provide additional settings, such as port mappings, as needed. To use it, reference your original `docker-compose.yml` file in addition to this one in `.devcontainer/devcontainer.extend.json` as follows:

	{
	    "name": "[Optional] Your project name here",
	    "dockerComposeFile": [
	        "../docker-compose.yml",
	        "docker-compose.extend.yml"
	    ],
	    "service": "your-service-name-here",
	    "workspaceFolder": "/workspace",
	    "shutdownAction": "stopCompose"
	}

VS Code will then **automatically use both files** when starting up any containers or you can start them yourself from the command line as follows:

	docker-compose up -f docker-compose.yml -f .devcontainer/docker-compose.extend.yml

### Using an updated Dockerfile to automatically install more tools

You may want to install other tools such as Git inside the container for the service you've specified. You can easily [do this manually](https://code.visualstudio.com/docs/remote/containers#_installing-additional-software-in-the-sandbox). However, you can also create a custom `Dockerfile` specifically for development that includes these dependencies. The [vscode-dev-containers repository](https://github.com/Microsoft/vscode-dev-containers) contains examples you can use to augment a copy of a Dockerfile or when creating a new one.

Assuming you put this file under `.devcontainer/Dockerfile`, the `.devcontainer/docker-compose.yml` above would be modified as follows:

	version: '3'
	  services:
	    your-service-name-here:
	    build:
	      context: .
	      # Location is relative to folder containing this compose file
	      dockerfile: Dockerfile
	    ports:

	      - 3000:3000

	    volumes:

	      - ..:/workspace

	    command: sleep infinity

### Docker Compose dev container definitions

The following are dev container definitions that use Docker Compose:

- [Existing Docker Compose](https://aka.ms/vscode-remote/samples/existing-docker-compose) - Includes a set of files that you can drop into an existing project that will reuse a `docker-compose.yml` file in the root of your project.
- [Node.js & MongoDB](https://aka.ms/vscode-remote/samples/node-mongo) - A Node.js container that connects to a Mongo DB in a different container.
- [Python & PostgreSQL](https://aka.ms/vscode-remote/samples/python-postgresl) - A Python container that connects to PostGreSQL in a different container.
- [Docker-in-Docker Compose](https://aka.ms/vscode-remote/samples/docker-in-docker-compose) - Includes the Docker CLI and illustrates how you can use it to access your local Docker install from inside a dev container by volume mounting the Docker Unix socket.

### Advanced container configuration

See the [Advanced Container Configuration](https://code.visualstudio.com/docs/remote/containers-advanced) article for information on the following topics:

- [Adding another volume mount](https://code.visualstudio.com/docs/remote/containers-advanced#_adding-another-volume-mount)
- [Adding a non-root user to your dev container](https://code.visualstudio.com/docs/remote/containers-advanced#_adding-a-nonroot-user-to-your-dev-container)
- [Using Docker or Kubernetes from inside a container](https://code.visualstudio.com/docs/remote/containers-advanced#_using-docker-or-kubernetes-from-a-container)
- [Connecting to multiple containers at once](https://code.visualstudio.com/docs/remote/containers-advanced#_connecting-to-multiple-containers-at-once)
- [Using SSH to connect to a remote Docker host](https://code.visualstudio.com/docs/remote/containers-advanced#_using-ssh-to-connect-to-a-remote-docker-host)
- [Reducing Dockerfile build warnings](https://code.visualstudio.com/docs/remote/containers-advanced#_reducing-dockerfile-build-warnings)

## devcontainer.json reference

Property
Type
Description
**Dockerfile or image**

[object Object]
string

**Required** when [using an image](https://code.visualstudio.com/docs/remote/containers#_using-an-existing-container-image). The name of an image in a container registry ([DockerHub](https://hub.docker.com/), [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)) that VS Code should use to create the dev container.

[object Object]
string

**Required** when [using a Dockerfile](https://code.visualstudio.com/docs/remote/containers#_using-a-dockerfile). The location of a [Dockerfile](https://docs.docker.com/engine/reference/builder/) that defines the contents of the container. The path is relative to the [object Object] file. You can find a number of sample Dockerfiles for different runtimes [in this repository](https://github.com/Microsoft/vscode-dev-containers/tree/master/containers).

[object Object]
string

Path that the Docker build should be run from relative to [object Object]. For example, a value of [object Object] would allow you to reference content in sibling directories. Defaults to [object Object].

[object Object]
integer, string, or array

A port or array of ports that should be made available locally when the container is running. Defaults to [object Object].

[object Object]
array

An array of [Docker CLI arguments](https://docs.docker.com/engine/reference/commandline/run/) that should be used when running the container. Defaults to [object Object]. A run argument can refer to environment variables using the following format [object Object]

[object Object]
boolean

Tells VS Code whether it should run [object Object] when starting the container instead of the default command to prevent the container from immediately shutting down if the default command fails. Defaults to [object Object].

[object Object]
enum: [object Object], [object Object]

Indicates whether VS Code should stop the container when the VS Code window is closed / shut down. Defaults to [object Object].

**Docker Compose**

[object Object]
string or array

**Required.** Path or an ordered list of paths to Docker Compose files relative to the [object Object] file.

[object Object]
string
**Required.** The name of the service VS Code should connect to once running.
[object Object]
string

Sets the default path that VS Code should open when connecting to the container (which is often the path to a volume mount where the source code can be found in the container). Defaults to [object Object].

[object Object]
enum: [object Object], [object Object]

Indicates whether VS Code should stop the containers when the VS Code window is closed / shut down. Defaults to [object Object].

**General**

[object Object]
string
A display name for the container.
[object Object]
array

An array of extension IDs that specify the extensions that should be installed inside the container when it is created. Defaults to [object Object].

[object Object]
string or array

A command or list of commands to run after the container is created (for example, [object Object]). Defaults to none.

[object Object]
integer

Allows you to force a specific port that the VS Code Server should use in the container. Defaults to a random, available port.

## Known limitations

### Remote - Containers limitations

- Alpine Linux or Windows container images are not yet supported. Most images come with a Debian or Ubuntu based flavor you can use instead. (Typically Alpine variations end in `-alpine`).
- All roots/folders in a multi-root workspace will be opened in the same container, regardless of whether there are configuration files at lower levels.
- The unofficial Ubuntu Docker **snap** package for Linux is **not** supported. Follow the [official Docker install instructions for your distribution](https://docs.docker.com/install/#supported-platforms).
- Docker Toolbox is not currently supported.
- Docker variants or alternate containerization tool kits like [podman.io](https://podman.io/) are not supported.
- When installing an extension pack in a container, extensions may install locally instead of inside the container. Click the **Install** button for each extension in the Local section of the extension panel to work around the issue. See [Microsoft/vscode-remote-release#11](https://github.com/Microsoft/vscode-remote-release/issues/11) for details.
- If you clone a Git repository using SSH and your SSH key has a passphrase, VS Code's pull and sync features may hang when running remotely. Either use an SSH key without a passphrase, clone using HTTPS, or run `git push` from the command line to work around the issue.
- Local proxy settings are not reused inside the container, which can prevent extensions from working unless the appropriate proxy information is configured (for example global `HTTP_PROXY` or `HTTPS_PROXY` environment variables with the appropriate proxy information).

See [here for a list of active issues](https://aka.ms/vscode-remote/containers/issues) related to Containers.

### Docker limitations

- First time installs of Docker Desktop for Windows will require an additional "sharing" step to give your container access to local source code. However, this step may not work with certain AAD (email based) identities. See [Docker Desktop for Windows tips](https://code.visualstudio.com/docs/remote/troubleshooting#_docker-desktop-for-windows-tips) and [Enabling file sharing in Docker Desktop](https://code.visualstudio.com/docs/remote/troubleshooting#_enabling-file-sharing-in-docker-desktop) for details and workarounds.
- You may see errors if you sign into Docker with your email address instead of your Docker ID. This is a known issue and can be resolved by signing in with your Docker ID instead. See Docker issue [#935](https://github.com/docker/hub-feedback/issues/935#issuecomment-300361781) for details.
- If you see high CPU spikes for `com.docker.hyperkit` on macOS, this may be due to a [known issue with Docker for Mac](https://github.com/docker/for-mac/issues/1759). See the Docker issue for details.
- If you see either of these messages building a Dockerfile, you may be hitting a known Docker issue with Debian 8 (Jessie):

	W: Failed to fetch http://deb.debian.org/debian/dists/jessie-updates/InRelease
	E: Some index files failed to download. They have been ignored, or old ones used instead

See [here for a workaround](https://code.visualstudio.com/docs/remote/troubleshooting#_resolving-dockerfile-build-failures-for-images-using-debian-8).

See the Docker troubleshooting guide for [Windows](https://docs.docker.com/docker-for-windows/troubleshoot) or [Mac](https://docs.docker.com/docker-for-mac/troubleshoot), consult [Docker Support Resources](https://success.docker.com/article/best-support-resources) for more information.

### Docker Extension limitations

Some Docker commands invoked from the Docker extension can fail when invoked from a VS Code window opened in a container. Most containers do not have the Docker command line installed. Therefore commands invoked from the Docker extension that rely on the Docker command line, for example **Docker: Show Logs**, fail. If you need to execute these commands, open a new local VS Code window and use the Docker extension from this window or [set up Docker inside your container](https://aka.ms/vscode-remote/samples/docker-in-docker).

### Extension limitations

Many extensions will work inside dev containers without modification. However, in some cases, certain features may require changes. If you run into an extension issue, [see here for a summary of common problems and solutions](https://code.visualstudio.com/docs/remote/troubleshooting#_extension-tips) that you can mention to the extension author when reporting the issue.

## Common questions

### I am seeing errors when trying to mount the local filesystem into a container

Right-click on the Docker task bar item. On Windows, go to the **Settings > Shared Drives** tab and check the drive(s) where your source code is located. On macOS, go the **Preferences > File Sharing** tab and make sure the folder containing your source code is under a file path specified in the list.

See [Docker Desktop for Windows tips](https://code.visualstudio.com/docs/remote/troubleshooting#_docker-desktop-for-windows-tips) for information on workarounds to common Docker for Windows issues.

### I am seeing "W: Failed to fetch ..." when building a Dockerfile

If you see "W: Failed to fetch http://deb.debian.org/debian/dists/jessie-updates/InRelease" when building a Dockerfile, you may be hitting a known Docker issue with Debian 8 (Jessie). See [here for a workaround](https://code.visualstudio.com/docs/remote/troubleshooting#_resolving-dockerfile-build-failures-for-images-using-debian-8).

### I'm seeing an error about a missing library or dependency

Some extensions rely on libraries not found in the certain Docker images. See [above](https://code.visualstudio.com/docs/remote/containers#_installing-additional-software-in-the-sandbox) for help resolving the problem.

### Can I connect to multiple containers at once?

A VS Code window can only connect to one window currently, but you can open a new window and [attach](https://code.visualstudio.com/docs/remote/containers#_attaching-to-running-containers) to an already running container or [use a common Docker Compose file with multiple `devcontainer.json` files](https://code.visualstudio.com/docs/remote/containers-advanced#_connecting-to-multiple-containers-at-once) to automate the process a bit more.

### Can I work with containers on a remote host?

With some manual configuration, yes. You can attach to a container running on a remote host or create specialized `devcontainer.json` and `Docker Compose` files designed to work with your remote host. See [Using SSH to connect to a remote Docker host](https://code.visualstudio.com/docs/remote/containers-advanced#_using-ssh-to-connect-to-a-remote-docker-host) for details.

### How can I build or deploy container images into my local Docker / Kubernetes install when working inside a container?

You can build images and deploy containers by forwarding the Docker socket and installing the Docker CLI (and kubectl for Kubernetes) in the container. See the [Docker-in-Docker](https://aka.ms/vscode-remote/samples/docker-in-docker), [Docker-in-Docker Compose](https://aka.ms/vscode-remote/samples/docker-in-docker-compose), and [Kubernetes-Helm](https://aka.ms/vscode-remote/samples/kubernetes-helm) dev container definitions for details.

### What are the connectivity requirements for the VS Code Server when it is running in a container?

The VS Code Server requires outbound HTTPS (port 443) connectivity to `update.code.visualstudio.com` and `marketplace.visualstudio.com`. All other communication between the server and the VS Code client is accomplished through an authenticated, random, TCP port automatically exposed via the Docker CLI.

### As an extension author, what do I need to do to make sure my extension works?

The VS Code extension API hides most of the implementation details of running remotely so many extensions will just work inside dev containers without any modification. However, we recommend that you test your extension in a dev container to be sure that all of its functionality works as expected. See the article on [Supporting Remote Development](https://code.visualstudio.com/api/advanced-topics/remote-extensions) for details.

### What other resources are there that can may be able to answer my question?

The following articles may help answer your question:

- [Advanced Container Configuration](https://code.visualstudio.com/docs/remote/containers-advanced) or [Tips and Tricks](https://code.visualstudio.com/docs/remote/troubleshooting#_containers-tips)
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Docker Desktop for Windows troubleshooting guide](https://docs.docker.com/docker-for-windows/troubleshoot) and [FAQ](https://docs.docker.com/docker-for-windows/faqs/)
- [Docker Desktop for Mac troubleshooting guide](https://docs.docker.com/docker-for-mac/troubleshoot) and [FAQ](https://docs.docker.com/docker-for-mac/faqs/)
- [Docker Support Resources](https://success.docker.com/article/best-support-resources)

## Questions or feedback

- See [Tips and Tricks](https://code.visualstudio.com/docs/remote/troubleshooting#_containers-tips) or the [FAQ](https://code.visualstudio.com/docs/remote/faq).
- Search on [Stack Overflow](https://stackoverflow.com/questions/tagged/vscode-remote).
- Add a [feature request](https://aka.ms/vscode-remote/feature-requests) or [report a problem](https://aka.ms/vscode-remote/issues/new).
- Create a [development container definition](https://aka.ms/vscode-dev-containers) for others to use.
- Contribute to [our documentation](https://github.com/Microsoft/vscode-docs) or [VS Code itself](https://github.com/Microsoft/vscode).
- See our [CONTRIBUTING](https://aka.ms/vscode-remote/contributing) guide for details.

### Was this documentation helpful?

5/15/2019