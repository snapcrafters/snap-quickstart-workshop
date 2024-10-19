# Snap Quickstart Workshop

Welcome to the Snap Quickstart workshop! For an introduction, take a look at [the introductory presentation](https://docs.google.com/presentation/d/1RF5gkTIRI0UiP0Dwusphb-n-Ld1UwqtKZY7LR50k8CU/edit?usp=sharing).

## Getting started

Download the virtual machine for your hypervisor of choice:

* VirtualBox: https://cloud.ilabt.imec.be/index.php/s/4AYaQSJeYDEdYxL
* VMWare: https://cloud.ilabt.imec.be/index.php/s/BS4n37bsJTsGERi

Run the VM and check if you have network connectivity. If you do, run the following commands in a termninal.

```bash
git clone https://github.com/snapcrafters/snap-quickstart-workshop.git
code snap-quickstart-workshop
```

This will open Visual Studio Code in this repository. Now you're ready to get started!

## Snapping a hello world GTK application

Snaps are a container technology similar to Docker containers, but they have a number of advantages for desktop Linux applications.

* Extensive support for GUI applications.
* Thorough sandbox that asks users if an app is allowed to access the camera, microphone and other sensitive features.
* Seamless integration in the Ubuntu App Store.

So let's build your first snap package! We'll try this out with `hello-world-gtk`, a very simple GTK application. You can find the source code for this application in the file [`hello-world-gtk/src/hello-world-gtk.c`](./hello-world-gtk/src/hello-world-gtk.c).

The first step is to add a `snapcraft.yaml` file in the `hello-world-gtk` folder. `snapcraft.yaml` is a file that explains how to compile your application, how to package it as a snap, and what permissions it needs.

The first part of this file contains user-facing metadata of your application.

```yaml
name: hello-world-gtk
version: '0.1'
summary: Gtk Hello World example
description: A simple Gtk example
```

After that, we choose the `base` to use for this snap. This defines two things:

* The Ubuntu version that is used **inside of your application container**. Your application inside of the container will think it is running on that version of Ubuntu, regardless of the host operating system. `core24` is based on Ubuntu 24.04, `core22` is based on Ubuntu 22.04, and so forth. If your app is only compatible with an older version of Ubuntu, you can use that base, and still run your app on newer Ubuntu versions.
* The Ubuntu version used for **building your app**. The build tool starts a container or virtual machine with that version of Ubuntu, regardless of what version of Ubuntu you're currently running. Newer versions of Ubuntu will use newer versions of compilers and build tools. Use the latest version (`core22`), to get the latest build tools.

```yaml
base: core24
```

Next, we define the confinement type. For most applications, this should always be `strict`. This means that your application runs in a full sandbox, and only has access to the host system if you explicitly grant it. (more on this below)

```yaml
confinement: strict
```

> Some special applications like an IDE can use a second type of confinement: `classic`. This means your application has full access to the host system. This is not recommended for most apps, however, because it is much harder to make your application work like that, and it is much less secure.

The next step is to define how to build your application. You can split this up into multiple "parts", if your application is composed out of multiple components that need to be built separately. For example, if you have an application with a Java and a Python component, you will need to use at least two parts.

For this application, however, we need only a single part.

```yaml
parts:
  hello-world-gtk:
    plugin: dump
    source: .
    override-build: |
      set -eux
      cd src
      gcc $(pkg-config --cflags gtk4) -o hello-world-gtk hello-world-gtk.c $(pkg-config --libs gtk4)
      cd ..
      craftctl default
    build-packages:
      - pkgconf
      - libgtk-4-dev
    stage-packages:
      - libgtk-4-1

```

* `plugin` specifies what build tool to use. Take a look at the list of [supported plugins](https://snapcraft.io/docs/supported-plugins) to see what languages are supported.
* `source` explains where to find the source code for this part. This can be a local directory, or it can be remote, such as a GitHub repository, or a zip file to download. We use `.` to specify the sources are in the current directory.
* `stage-packages` describes what additional packages from [the Ubuntu archive](https://packages.ubuntu.com/jammy/allpackages) your application needs. These are the applications you normally install using `apt` in order to get your app to work on Ubuntu.

Finally, we define how to start your application. One snap can have multiple apps inside of it. In this case, we only have a single command: the webserver.

```yaml
apps:
  hello-world-gtk:
    command: src/hello-world-gtk
    plugs:
      - x11
      - wayland
```

* `command` specifies how to start your application. This is relative from the root of your snap.
* `plugs` specifies what permissions your application wants. `network-bind` means that your application has access to the network and can listen (bind) to a port. Take a look at all the [permissions snap supports](https://snapcraft.io/docs/supported-interfaces).

Note that there is a difference between the permissions that an application _requests_ and the permissions that it actually gets. Getting access is called "connecting an interface to a snap". There are three ways that snaps get connected to interfaces:

* Manual connections: users and device manufacturers can [manually connect and disconnect interfaces](https://snapcraft.io/docs/interface-management#heading--manual-connections) after installation of a snap.
* Global auto-connects: some interfaces, such as `network-bind` and `audio-playback`, are automatically connected to all snaps. These interfaces are marked with ["auto-connect" in the documentation](https://snapcraft.io/docs/supported-interfaces). When a snap is installed, it will be automatically connected to any such interfaces it requests.
* Reviewed auto-connects: app developers can request additional automatic permissions for their application [using the permission request process](https://snapcraft.io/docs/permission-requests). The Snap Store team will review the security implications of automatically connecting this interface to all users of the snap.

Since the `network-bind` interface is globally auto-connected, installing this snap will immediately give it the permissions it needs.

## Building the snap

Now that all these things are in place, we can build the snap using [`snapcraft`](https://snapcraft.io/docs/snapcraft-overview). Simply open a terminal, go to this directory, and run the following command:

```shell
snapcraft --verbose
```

This will

* spin up a container or VM with the correct ubuntu version for the chosen `base`,
* download any external sources,
* install any dependencies,
* build your application,
* and package your application into a snap together with the dependencies.

The end result is the file `hello-world-gtk_0.1_amd64.snap`. Congratulations, you just built your first snap! You can install it using

```shell
sudo snap install ./hello-world-gtk_0.1_amd64.snap --dangerous
```

The `--dangerous` flag is to denote that this snap has not been signed, so you need to trust the source of the snap.

You can then run the command by running

```shell
hello-world-gtk
```
