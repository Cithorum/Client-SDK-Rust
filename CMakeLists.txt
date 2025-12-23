cmake_minimum_required(VERSION 3.31.0)
project(livekit-rust-cmake LANGUAGES C CXX)

# ---- C++ standard ----
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(RUST_LIVEKIT_PROTOCOL_PATH "${CMAKE_CURRENT_LIST_DIR}/livekit-protocol/protocol")
set(RUST_GOOGLE_LIBYUV_PATH    "${CMAKE_CURRENT_LIST_DIR}/yuv-sys/libyuv")

# Fetch submodules via CMake, not .gitmodules
include(FetchContent)

FetchContent_Declare(
    livekit_protocol 
    URL https://github.com/livekit/protocol/archive/refs/heads/main.zip
  )
FetchContent_MakeAvailable(livekit_protocol)

FetchContent_Declare(
    libyuv 
    URL https://chromium.googlesource.com/libyuv/libyuv/+archive/refs/heads/main.tar.gz
  )
FetchContent_MakeAvailable(libyuv)

include(CopyRecursive.cmake)
CopyRecursive(${livekit_protocol_SOURCE_DIR} ${RUST_LIVEKIT_PROTOCOL_PATH} "*")
CopyRecursive(${libyuv_SOURCE_DIR}           ${RUST_GOOGLE_LIBYUV_PATH}    "*")

# ---- Build Rust Library ----
if(${CMAKE_BUILD_TYPE} MATCHES "Release")
    set(RUST_RELEASE_FLAG "--release")
elseif(${JAM_ENABLE_TRACY})
    set(RUST_RELEASE_FLAG "")
endif()

set(RUST_BUILD_GENERATOR "cargo build ${RUST_RELEASE_FLAG}")

if(CMAKE_SYSTEM_NAME MATCHES "Windows")
    execute_process(
        COMMAND cmd /C ${RUST_BUILD_GENERATOR}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
    )
else()
    execute_process(
        COMMAND bash ${RUST_BUILD_GENERATOR}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
    )
endif()

# No C++ code is compiled here, just external invocations to command-line tools are performed