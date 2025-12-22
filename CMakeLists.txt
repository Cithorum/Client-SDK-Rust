cmake_minimum_required(VERSION 3.31.0)
project(livekit LANGUAGES C CXX)

# ---- C++ standard ----
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

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