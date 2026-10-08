cmake_minimum_required(VERSION 3.21)

project(DragAndDrop VERSION 0.1.99 LANGUAGES CXX)
set(_requested_label "${DRAGANDDROP_VERSION_LABEL}")
set(DRAGANDDROP_VERSION_LABEL "alpha" CACHE STRING "Candidate version suffix" FORCE)
if(NOT _requested_label STREQUAL "" AND NOT _requested_label STREQUAL DRAGANDDROP_VERSION_LABEL)
    message(FATAL_ERROR "Change the version label in CMakeLists.txt, not a cache override, so package identity stays synchronized")
endif()

set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# Keep the SDK and plugin on the same C++23 and SE/AE ABI profile.
if(MSVC)
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("/std:c++23" DRAGANDDROP_CXX23)
    check_cxx_compiler_flag("/std:c++23preview" DRAGANDDROP_CXX23_PREVIEW)
    if(DRAGANDDROP_CXX23)
        set(CMAKE_CXX23_STANDARD_COMPILE_OPTION "/std:c++23")
    elseif(DRAGANDDROP_CXX23_PREVIEW)
        set(CMAKE_CXX23_STANDARD_COMPILE_OPTION "/std:c++23preview")
    endif()
    set(CMAKE_CXX23_EXTENSION_COMPILE_OPTION "${CMAKE_CXX23_STANDARD_COMPILE_OPTION}")
endif()
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)
set(BUILD_TESTS OFF CACHE BOOL "CommonLib tests" FORCE)
set(ENABLE_SKYRIM_SE ON CACHE BOOL "SE support" FORCE)
set(ENABLE_SKYRIM_AE ON CACHE BOOL "AE support" FORCE)
set(ENABLE_SKYRIM_VR OFF CACHE BOOL "VR support" FORCE)
set(COMMONLIB_PREBUILT OFF CACHE BOOL "Source-build CommonLib" FORCE)
set(COMMONLIB_PREBUILT_DIR "" CACHE PATH "No prebuilt override" FORCE)
set(_minll_commit "550cc4fb9114649dcf526d1f3d73d710c5d7003b")
FetchContent_Declare(CommonLibSSE
    GIT_REPOSITORY https://github.com/MinLL/CommonLibVR.git
    GIT_TAG ${_minll_commit}
    GIT_SUBMODULES extern/openvr)
FetchContent_MakeAvailable(CommonLibSSE)

get_target_property(_imported CommonLibSSE IMPORTED)
if(_imported)
    message(FATAL_ERROR "Migration requires source-built CommonLib")
endif()
get_target_property(_minll_source CommonLibSSE SOURCE_DIR)
find_package(Git REQUIRED)
execute_process(COMMAND "${GIT_EXECUTABLE}" -C "${_minll_source}" rev-parse HEAD
    RESULT_VARIABLE _rev_result OUTPUT_VARIABLE _rev OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND "${GIT_EXECUTABLE}" -C "${_minll_source}" status --porcelain --untracked-files=all
    RESULT_VARIABLE _status_result OUTPUT_VARIABLE _status OUTPUT_STRIP_TRAILING_WHITESPACE)
if(NOT _rev_result EQUAL 0 OR NOT _rev STREQUAL _minll_commit OR
   NOT _status_result EQUAL 0 OR NOT _status STREQUAL "")
    message(FATAL_ERROR "CommonLib must be the clean pinned checkout: ${_rev} ${_status}")
endif()
file(STRINGS "${_minll_source}/LICENSE" _license LIMIT_COUNT 1)
if(NOT _license STREQUAL "MIT License")
    message(FATAL_ERROR "Unexpected CommonLib license")
endif()
message(STATUS "CommonLib source: ${_minll_source} @ ${_rev}")

# This pin has one extra non-VR virtual; correct SDK and plugin together.
set(_abi_include "${CMAKE_CURRENT_BINARY_DIR}/commonlib-abi/include")
file(READ "${_minll_source}/include/RE/T/TESObjectREFR.h" _header)
string(REPLACE "\r\n" "\n" _header "${_header}")
string(SHA256 _original_sha "${_header}")
if(NOT _original_sha STREQUAL "98a8a1cb24f51b893b4343c4c552e363d9bc90ee7a611c05f7719c2479bec2b6")
    message(FATAL_ERROR "Header changed; re-audit the non-VR ABI correction")
endif()
string(REGEX REPLACE "\n[ \t]*SKYRIM_REL_VR_VIRTUAL void[ \t]+Unk_8C\\(void\\);[^\n]*" "" _fixed "${_header}")
string(SHA256 _fixed_sha "${_fixed}")
if(NOT _fixed_sha STREQUAL "6ff59cd37693e301cc3282099a11b785f4a386334f0947adc37511afeabf690f")
    message(FATAL_ERROR "Unexpected corrected header; re-audit the non-VR ABI correction")
endif()
file(MAKE_DIRECTORY "${_abi_include}/RE/T")
file(GENERATE OUTPUT "${_abi_include}/RE/T/TESObjectREFR.h" CONTENT "${_fixed}")
target_include_directories(CommonLibSSE BEFORE PUBLIC "$<BUILD_INTERFACE:${_abi_include}>")
include("${_minll_source}/cmake/CommonLibSSE.cmake")

add_commonlibsse_plugin(${PROJECT_NAME} USE_ADDRESS_LIBRARY
    SOURCES src/main.cpp src/DragHandler.cpp src/Hooks.cpp)
target_include_directories(${PROJECT_NAME} BEFORE PRIVATE "${_abi_include}")
target_compile_definitions(${PROJECT_NAME} PRIVATE
    DRAGANDDROP_VERSION="${PROJECT_VERSION}-${DRAGANDDROP_VERSION_LABEL}")
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_23)
target_precompile_headers(${PROJECT_NAME} PRIVATE src/PCH.h)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)
target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)

