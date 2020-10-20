## New things

### WSL

- REMOTE-WSL: New or ReOpen by WSL
- Install extensions on WSL, such as python, c/c++

### CMake

- CMake: Select a Kit -> Select the compiler you want to use
- CMake: Select Variant -> Select a variant contains instructions for how to build your project
- CMake: Configure -> configure your project after selecting kit and variant
- CMake: Build

### Tasks and Launch

- Tasks: cmake, make, build

  ```json
  {
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
      {
        "label": "cmake",
        "type": "shell",
        "command": "cmake -G 'Unix Makefiles' -DCMAKE_BUILD_TYPE=Debug ..",
        "presentation": {
          "echo": true,
          "reveal": "always",
          "focus": false,
          "panel": "shared",
          "showReuseMessage": true,
          "clear": true
        },
        "options": {
          "cwd": "${workspaceRoot}/build"
        }
      },

      {
        "label": "make",
        "type": "shell",
        "command": "make -j 6",
        "presentation": {
          "echo": true,
          "reveal": "always",
          "focus": false,
          "panel": "shared",
          "showReuseMessage": true,
          "clear": true
        },
        "options": {
          "cwd": "${workspaceRoot}/build"
        }
      },

      {
        "label": "build",
        "dependsOn": ["cmake", "make"]
      }
    ]
  }
  ```

- Launch: gdb with pre-build (preLaunchTask)

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "(gdb) Launch",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/CMakeProject1/CMakeProject1",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "build"
    }
  ]
}
```

### Tools

- clang-format: sudo apt install clang-format
  provide .clang-format in the work project root folder
- clang-tidy: sudo apt install clang-tidy
  provide .clang-tidy
- valgrind: sudo apt install valgrind

### setting.json

```json
{
  "cmake.sourceDirectory": "${workspaceFolder}/src/t7",
  //"cmake.installPrefix": "/tmp/t3/usr",
  "cmake.buildBeforeRun": true,
  "cmake.configureSettings": {
    "CMAKE_TOOLCHAIN_FILE": "/home/anoldfriend/Workspace/MyRepo/vcpkg/scripts/buildsystems/vcpkg.cmake"
  },
  // "clang-tidy.buildPath": "${workspaceFolder}/build",// wrong,clang-tidy.buildPath donot support variables like ${workspaceFolder}.
  // https://github.com/notskm/vscode-clang-tidy/issues/39
  //   it seems you can use relative paths as a workaround. So instead of ${workspaceFolder}/build you can use build.
  "clang-tidy.buildPath": "build",
  // it seems the .clang-tidy is not consumed by clang-tidy extensiosn
  "clang-tidy.checks": [
    "clang-diagnostic-*",
    "clang-analyzer-*",
    "readability-*",
    "modernize-*",
    "-modernize-deprecated-headers",
    "-modernize-use-trailing-return-type"
  ],

  //// If you not set the build path, you can also use the compilerArgs to add include dirfor clang - tidy to avoid the include error from clang - tidy
  // "clang-tidy.compilerArgs": [
  //     "-I",
  //     "/tmp/t3/usr/include/hello"
  // ],

  "C_Cpp.errorSquiggles": "Enabled",
  "C_Cpp.intelliSenseEngineFallback": "Enabled"
}
```

### c_cpp_properties.json

```json
{
  "configurations": [
    {
      "name": "Linux",
      "includePath": [],
      "defines": [],
      "compilerPath": "/usr/bin/gcc",
      "cStandard": "c11",
      "cppStandard": "gnu++14",
      "intelliSenseMode": "clang-x64",
      "configurationProvider": "ms-vscode.cmake-tools ",
      "compileCommands": "${workspaceFolder}/build/compile_commands.json"
    }
  ],
  "version": 4
}
```
