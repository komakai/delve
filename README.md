
![Delve](https://raw.githubusercontent.com/go-delve/delve/master/assets/delve_horizontal.png)

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/go-delve/delve/master/LICENSE)
[![Go Reference](https://pkg.go.dev/badge/github.com/go-delve/delve.svg)](https://pkg.go.dev/github.com/go-delve/delve)
[![Build Status](https://delve.teamcity.com/app/rest/builds/buildType:(id:Delve_AggregatorBuild)/statusIcon.svg)](https://delve.teamcity.com/viewType.html?buildTypeId=Delve_AggregatorBuild&guest=1)

The GitHub issue tracker is for **bugs** only. Please use the [developer mailing list](https://groups.google.com/forum/#!forum/delve-dev) for any feature proposals and discussions.

### About this fork

This fork adds Android support

* Build command
``
CC=$ANDROID_NDK_ROOT/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android24-clang CGO_ENABLED=1 GOOS=android GOARCH=arm64 go build github.com/go-delve/delve/cmd/dlv
``

* Push dlv executable to Android device and set up port forwarding:
``
adb push ./dlv /data/local/tmp
``

* Install Visual Studio Code and the Visual Studio Code GO plugin: https://marketplace.visualstudio.com/items?itemName=golang.go

* In Visual Studio Code add tasks for building/installing/starting delve server in tasks.json like:
```
"options": {
    "env": {
        "CC": "${env:ANDROID_NDK_ROOT}/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android24-clang",
        "CGO_ENABLED": "1",
        "GOOS": "android",
        "GOARCH": "arm64"
    }
},
"tasks": [
    {
        "label": "build app",
        "command": "go",
        "args": [
            "build",
            "-gcflags",
            "all=-N -l",
            "hello-world.go"
        ],
    },
    {
        "label": "kill daemon",
        "type": "shell",
        "command": "adb shell pkill dlv || echo 0",
    },
    {
        "label": "forward ports",
        "type": "shell",
        "command": "adb -s OR65T31SM2D220900001 forward tcp:5566 tcp:5566",
        "detail": "adb forward tcp:5566 tcp:5566"
        },
     {
        "label": "install app",
        "type": "shell",
        "command": "adb push ./hello-world /data/local/tmp",
        "dependsOn": [
            "build app"
        ]
    },
    {
        "label": "Start delve server",
        "type": "shell",
        "command": "adb shell /data/local/tmp/dlv --listen=:5566 --headless=true --api-version=2 exec /data/local/tmp/hello-world",
        "dependsOn": [
            "kill daemon",
            "forward ports",
            "install app"
        ],
        "dependsOrder": "sequence"
    }
]
```
* In launch.json add a configuration like:
```
"configurations": [
{
    "name": "Connect to delve server",
    "type": "go",
    "request": "attach",
    "mode": "remote",
    "remotePath": "${workspaceFolder}",
    "port": 5566,
    "host": "127.0.0.1"
}
```
#### Debugging
Unfortunately we can't use the preLaunchTask element in the configuration because the configuration will hang waiting for the delve server to exit before it tries to attach. Instead we can run the "Start delve server" from the Terminal -> Run Task menu option. The task will start dlv in headless server mode on the device and wait for a client to attach. Then we can set break points and run the "Connect to delve server" configuration and debug in the Visual Studio Code UI

#### Evaluating expressions

In the DEBUG CONSOLE simply enter the expression you want to evaluate. This can be useful for inspecting large arrays or when casting is required. Example:
Building GO application with debug information

Check the build setting and look to see if the flags "-s" or "-w" are passed to the linker (for example via the EXTRA_LDFLAGS environment variable). If they are then remove them and rebuilt

#### Location of task.json and launch.json files

Create a .vscode directory under the project directory and place the files there. For example for the nerdctl project under $GOROOT/src/github.com/containerd/nerdctl

From Visual Code select File → Open Folder and select the project directory 

### About Delve

- [Installation](Documentation/installation)
- [Getting Started](Documentation/cli/getting_started.md)
- [Documentation](Documentation)
  - [Command line options](Documentation/usage/dlv.md)
  - [Command line client](Documentation/cli/README.md)
  - [Plugins and GUIs](Documentation/EditorIntegration.md)
  - [Frequently Asked Questions](Documentation/faq.md)
- [Contributing](CONTRIBUTING.md)
  - [Internal Documentation](Documentation/internal)
  - [API documentation](Documentation/api)
  - [How to write a Delve client](Documentation/api/ClientHowto.md)

Delve is a debugger for the Go programming language. The goal of the project is to provide a simple, full featured debugging tool for Go. Delve should be easy to invoke and easy to use. Chances are if you're using a debugger, things aren't going your way. With that in mind, Delve should stay out of your way as much as possible.

