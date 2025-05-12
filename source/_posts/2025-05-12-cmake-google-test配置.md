---
title: cmake google test配置
date: 2025-05-12 17:30:22
categories:
tags:
---
範例
```cmake
find_package(Threads REQUIRED)


include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        52eb8108c5bdec04579160ae17225d66034bd723 # release-1.17.0
)
FetchContent_MakeAvailable(googletest)
enable_testing()

# Create a libgmock target to be used as a dependency by test programs

add_subdirectory(testplate)

```

```cmake
include(GoogleTest)
file(GLOB SRCS *.cpp)
add_executable(testplate ${SRCS})


target_link_libraries(testplate
    gtest_main
    libtestplate
)

gtest_add_tests(TARGET      testplate
                TEST_SUFFIX .noArgs
                TEST_LIST   noArgsTests
)

#gtest_add_tests(TARGET      testplate
#                EXTRA_ARGS  --someArg someValue
#                TEST_SUFFIX .withArgs
#                TEST_LIST   withArgsTests
#)

set_tests_properties(${noArgsTests}   PROPERTIES TIMEOUT 10)
#set_tests_properties(${withArgsTests} PROPERTIES TIMEOUT 20)
```

參考:
https://cmake.org/cmake/help/latest/module/FetchContent.html#integrating-with-find-package

https://cmake.org/cmake/help/v3.31/module/GoogleTest.html#command:gtest_add_tests