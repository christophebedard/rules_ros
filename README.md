# ROS2 Rules for Bazel

## Overview

This repository contains all the setup, rules and build configuration to use
[Bazel](http://bazel.build) with ROS2. As reccomended by Bazel, all ROS2 packages
are built from source. They are loaded as needed, so no a priori loading or manual version
management of ROS2 repos is required.

The neccessary BUILD files for ROS2 repos are injected while loading. In case bazel will
gain some traction within the ROS community, Bazel BUILD files can also be provided directly
by the ROS2 repositories.

Specific rules for message generation and packaging are provided in this repository (see 
[rosidl/defs.bzl](rosidl/defs.bzl) and [pkg/defs.bzl](pkg/defs.bzl)).

## ! Current Restrictions !

This is still work in progress. Some essential parts are not complete yet. 
Here is a short list of major restrictions:
* Only tested on Ubunut 20.04 linux (x86_64). Other linux distributions may work. Windows
  will not be supported in the foreseeable future.
* Only ROS2 humble supported. Other distros may work after extending
  `@rules_ros//repos/config/defs.bzl` accordingly.
* Message generation is still incomplete. Therefore even the simples examples will not compile
  fully yet.
* Not all packages have been bazelized yet. Main focus currently lies on generating an
  install space and providing message generation support.
* The addition of custom packages into the repo setup is not yet available. Same applies to
  adding additional python packages. This will be added soon.

## Getting started

### Workspace setup

To import rules_ros in your project, add the following code snippet to your WORKSPACE file:

```python
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_ROS_VERSION = "xxx" # TODO: where to find the right version
RUIES_ROS_SHA = "xxx"

https_archive(
    name = "rules_ros",
    sha256 = RUIES_ROS_SHA,
    strip_prefix = "xxx",
    url = "https://github.com/ApexAI/rules_ros/archive/{}.zip".format(RULES_ROS_VERSION),
)

load("@rules_ros//repos/config:defs.bzl", "configure_ros2")
configure_ros2(distro = "humble") # currently only humble is supported

load("@ros2_config//:setup.bzl", "setup")
setup()

load("@rules_ros//thirdparty:setup_01.bzl", "setup_01")
setup_01()

load("@rules_ros//thirdparty:setup_02.bzl", "setup_02")
setup_02()

load("@rules_ros//thirdparty:setup_03.bzl", "setup_03")
setup_03()

load("@rules_ros//thirdparty:setup_04.bzl", "setup_04")
setup_04()
```

### Python Interpreter

In this setup a hermetic python interpreter is included. The version is specified in 
`thirdparty/python/repositories.bzl`. Python packages specified in
`thirdparty/python/requirements_lock.in` are available for use. If you need to add a package,
you must run 
```console
bazel run @rules_ros//thirdparty/python:requirements_lock.update
```
to pin specific versions of the dependencies. In the future there will be a possibility to
inject any customization in the WORKSPACE file.

### ROS2 Repositories

ROS2 repositories are made available as external dependencies with the name specified in
the `ros2.repos` file known from the original ros2 repository. E. g. the repo `ros2/rclcpp`
will be available as `@ros2.rclcpp` with all targets specified in the BUILD files in
`repos/config/ros2.rclcpp.BUILD`. The precise location where the build files will be injected
into the rclcpp repo is specified in the index file `repos/config/bazel.repos`.

So, if you need to depend on `rclcpp` in you code, you would add `"@ros2.rclcpp//rclcpp"`
as a dependency in your `cc_binary` or `cc_library` target.

The exact version of a ros2 repository is pinned in the file `repos/config/ros2_<distro>.lock`.
This file can be updated for the configured distro by running:
```console
bazel run @rules_ros//repos/config:repos_lock.update
```
 In the future there will be a possibility to inject any customization in the WORKSPACE file.



