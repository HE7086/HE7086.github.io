# CMakeLists.txt

## invoke
```shell
cmake -B build -G "Ninja Multi-Config"
cmake --build build --config Release
cmake --install build --config Release --prefix .
```

## normal
```cmake
cmake_minimum_required(VERSION 3.20)
project(example)

if (NOT UNIX)
    message(FATAL_ERROR "Unsupported OS")
endif()

# Common Options
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# save the executables/libraries according to GNU standard
include(GNUInstallDirs)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

# enable warnings by default for all targets
add_compile_options(-Wall -Wextra -Wpedantic -Werror)

# enable sanitizers for debug build
add_compile_options("$<$<CONFIG:DEBUG>:-fsanitize=address;-fsanitize=undefined;-fno-omit-frame-pointer>")

# enable lto for release build
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE true)
# force fat lto objects
set(CMAKE_CXX_COMPILE_OPTIONS_IPO -flto=auto -ffat-lto-objects)
# enable strip for release build
add_link_options("$<$<CONFIG:RELEASE>:-s>")
# -fPIC
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
# build native binaries
add_compile_options("$<$<CONFIG:RELEASE>:-march=native>")

# use pthread
find_package(Threads REQUIRED)
set(THREADS_PREFER_PTHREAD_FLAG ON)

# static library
add_library(lib1 STATIC lib1.cpp)

# shared library
add_library(lib2 SHARED lib2.cpp)
install(TARGETS lib1 lib2 DESTINATION "${CMAKE_INSTALL_LIBDIR}")

# executable
add_executable(exe1 main.cpp)
add_dependencies(exe1 lib1 lib2)

target_link_libraries(exe1 PRIVATE lib1 lib2 Threads::Threads)
target_link_directories(exe1 PRIVATE "/opt/cuda/lib64")

target_include_directories(exe1 PRIVATE ${CMAKE_CURRENT_BINARY_DIR} "/opt/cuda/include")

target_compile_options(exe1 PRIVATE $<$<COMPILE_LANGUAGE:CXX>:$<$<CONFIG:DEBUG>:-g3;-Og>>)

target_compile_definitions(exe1 PRIVATE
    MAJOR_VERSION=0
    MINOR_VERSION=0
    PATCH_VERSION=1
    BUILD_DEBUG
    )
install(TARGETS exe1 DESTINATION "${CMAKE_INSTALL_BINDIR}")

# child targets, should contain CMakeLists.txt
add_subdirectory(examples)
```

## cuda
```cmake
set(CMAKE_CUDA_STANDARD 20)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_ARCHITECTURES native)
enable_language(CUDA)
find_package(CUDAToolkit REQUIRED)

add_compile_options("$<$<COMPILE_LANGUAGE:CUDA>:-Werror=all-warnings>")
add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:-Wall;-Wextra;-Wpedantic;-Werror>")

add_executable(examle_cuda
    worker.cu
    main.cpp
)
target_include_directories(example_cuda PRIVATE ${CUDAToolkit_INCLUDE_DIRS})
target_link_libraries(example_cuda PRIVATE
    CUDA::nvrtc # -lnvrtc
    CUDA::cuda_driver # -lcuda
)
```

## nasm
```cmake
enable_language(ASM_NASM)
set(CMAKE_ASM_NASM_COMPILE_OBJECT "<CMAKE_ASM_NASM_COMPILER> <INCLUDES> <FLAGS> -o <OBJECT> <SOURCE>")
set(CMAKE_ASM_NASM_FLAGS "-g -felf64")
set(CMAKE_C_FLAGS "-g -nostdlib -fPIE -fno-stack-protector")
add_executable(exe_asm a.asm b.c)
set_source_files_properties(a.asm PROPERTIES LANGUAGE ASM_NASM)
```

## flex & bison
```cmake
find_package(FLEX REQUIRED)
find_package(BISON REQUIRED)
FLEX_TARGET(SCANNER scannr.l ${CMAKE_CURRENT_BINARY_DIR}/scanner.cpp)
BISON_TARGET(PARSER parser.y ${CMAKE_CURRENT_BINARY_DIR}/parser.cpp)
add_flex_bison_dependency(SCAN PARSE)

add_executable(exe1 main.cpp ${FLEX_SCAN_OUTPUTS} ${BISON_PARSE_OUTPUTS})
target_link_libraries(exe1 PRIVATE ${FLEX_LIBRARIES})
```
