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



要結合 `CMakePresets.json` 和 GoogleTest，你可以藉由在 CMake 項目中使用 `CMakePresets.json` 來簡化建構過程，同時透過 `FetchContent` 或 `add_subdirectory` 等方式引入 GoogleTest。以下是一個完整的範例，展示如何整合這兩者。

---

## ✅ 專案結構範例

```
my_project/
├── CMakeLists.txt
├── CMakePresets.json
├── src/
│   └── mylib.cpp
│   └── mylib.h
├── tests/
│   └── test_mylib.cpp
```

---

## 📄 `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.23)
project(MyProject LANGUAGES CXX)

# 如果是測試 preset，就啟用測試
include(CTest)
enable_testing()

# 加入你的原始碼
add_library(mylib src/mylib.cpp)
target_include_directories(mylib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)

# GoogleTest：用 FetchContent 自動抓取
include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/refs/heads/main.zip
)
FetchContent_MakeAvailable(googletest)

# 測試程式
add_executable(tests tests/test_mylib.cpp)
target_link_libraries(tests gtest_main mylib)

include(GoogleTest)
gtest_discover_tests(tests)
```

---

## 📄 `CMakePresets.json`

```json
{
  "version": 3,
  "cmakeMinimumRequired": {
    "major": 3,
    "minor": 23,
    "patch": 0
  },
  "configurePresets": [
    {
      "name": "default",
      "hidden": true,
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_EXPORT_COMPILE_COMMANDS": "YES",
        "CMAKE_CXX_STANDARD": "17"
      }
    },
    {
      "name": "dev",
      "inherits": "default",
      "description": "開發模式"
    },
    {
      "name": "with-tests",
      "inherits": "default",
      "description": "啟用測試",
      "cacheVariables": {
        "BUILD_TESTING": "ON"
      }
    }
  ]
}
```

---

## 📄 測試檔 `tests/test_mylib.cpp`

```cpp
#include <gtest/gtest.h>
#include "mylib.h"

TEST(MyLibTest, AddTest) {
    EXPECT_EQ(add(2, 3), 5);
}
```

---

## ✅ 使用方式

1. 設定並建構：

   ```bash
   cmake --preset=with-tests
   cmake --build --preset=with-tests
   ```

2. 執行測試：

   ```bash
   ctest --preset=with-tests
   ```

---

如果你使用的是 Visual Studio Code 的 CMake Tools 插件，它會自動讀取 `CMakePresets.json`，讓你可以直接從 UI 選擇 preset 建構和執行測試。

---

是否需要我幫你根據你的實際專案目錄結構來調整？你也可以告訴我你是使用 `add_subdirectory()` 還是 `FetchContent` 來加入 GoogleTest。
