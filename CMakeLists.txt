cmake_minimum_required(VERSION 3.31.0)
project(livekit-rust-cmake LANGUAGES C CXX)

# ---- C++ standard ----
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(RUST_LIVEKIT_PROTOCOL_PATH "${CMAKE_CURRENT_LIST_DIR}/livekit-protocol")
set(RUST_GOOGLE_LIBYUV_PATH    "${CMAKE_CURRENT_LIST_DIR}/yuv-sys")

# Fetch submodules via CMake, not .gitmodules
FetchContent_Declare(
    livekit_protocol 
    URL https://github.com/livekit/protocol/archive/refs/heads/main.zip
  )
FetchContent_MakeAvailable(livekit_protocol)

FetchContent_Declare(
    libyuv 
    URL https://chromium.googlesource.com/libyuv/libyuv/archive/refs/heads/main.zip
  )
FetchContent_MakeAvailable(libyuv)

# Move ${libyuv_SOURCE_DIR} to the expected directory for rust cargo
file(RENAME ${livekit_protocol_SOURCE_DIR} ${RUST_LIVEKIT_PROTOCOL_PATH})
file(RENAME ${libyuv_SOURCE_DIR}           ${RUST_GOOGLE_LIBYUV_PATH})

# ---- Build Rust Library ----
set(RUST_BUILD_GENERATOR "cargo build")
set(RUST_OUTPUT_LIBRARY  "${CMAKE_CURRENT_LIST_DIR}/lib")
if(NOT EXISTS ${RUST_OUTPUT_LIBRARY})
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
endif()

# No C++ code is compiled here, just external invocations to command-line tools are performed