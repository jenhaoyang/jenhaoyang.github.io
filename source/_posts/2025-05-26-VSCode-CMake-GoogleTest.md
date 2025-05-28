---
title: VSCode_CMake_GoogleTest
date: 2025-05-26 18:15:43
categories:
tags:
---

# 重點:
下面這兩行要放在最外面的 `CMakeLists.txt` 中，這樣才能在build資料夾啟動ctest
```
include(CTest)
enable_testing()
```

# 讓VSCode的Ctest可以下中斷點
參考:
https://stackoverflow.com/a/76447033/22299707


https://github.com/microsoft/vscode-cmake-tools/blob/defc0b5369c64467c3466b1cee3faba9f9633a6a/docs/debug-launch.md#debugging-tests



launch.json
```launch.json
        {
            "name": "(ctest) Launch",
            "type": "cppdbg",
            "cwd": "${cmake.testWorkingDirectory}",
            "request": "launch",
            "program": "${cmake.testProgram}",
            "args": [ "${cmake.testArgs}" ],
            // other options...
        }
```


參考:
https://github.com/esweet431/box2d-lite/blob/vs-launch/CMakePresets.json