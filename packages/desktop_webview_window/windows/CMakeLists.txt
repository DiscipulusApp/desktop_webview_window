cmake_minimum_required(VERSION 3.14)
set(PROJECT_NAME "desktop_webview_window")
project(${PROJECT_NAME} LANGUAGES CXX)

# This value is used when generating builds using this plugin, so it must
# not be changed
set(PLUGIN_NAME "desktop_webview_window_plugin")

add_library(${PLUGIN_NAME} SHARED
        "desktop_webview_window_plugin.cpp"
        "webview_window.h"
        "webview_window.cc"
        "strconv.h"
        "utils.h"
        "utils.cc"
        "flutter_view.h"
        "flutter_view.cc"
        "web_view.h"
        "web_view.cc"
        "message_channel_plugin.h"
        "message_channel_plugin.cc"
        "web_view_window_plugin.h"
        "web_view_window_plugin.cc"
        )

apply_standard_settings(${PLUGIN_NAME})
set_target_properties(${PLUGIN_NAME} PROPERTIES
        CXX_VISIBILITY_PRESET hidden)
target_compile_definitions(${PLUGIN_NAME} PRIVATE FLUTTER_PLUGIN_IMPL)
target_include_directories(${PLUGIN_NAME} INTERFACE
        "${CMAKE_CURRENT_SOURCE_DIR}/include")
target_link_libraries(${PLUGIN_NAME} PRIVATE flutter flutter_wrapper_plugin flutter_wrapper_app)

# Determine target architecture (arm64, x64, or x86)
string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR} ${CMAKE_GENERATOR_PLATFORM} ${FLUTTER_TARGET_PLATFORM}" ARCH_STRING)
if(ARCH_STRING MATCHES "arm64|aarch64")
    set(WEBVIEW2_ARCH "arm64")
elseif(ARCH_STRING MATCHES "x86|win32|i686|i386")
    set(WEBVIEW2_ARCH "x86")
else()
    set(WEBVIEW2_ARCH "x64")
endif()

message(STATUS "desktop_webview_window: Using WebView2 architecture: ${WEBVIEW2_ARCH}")

add_library(Webview2 SHARED IMPORTED GLOBAL)
set_property(TARGET Webview2 PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/libs/${WEBVIEW2_ARCH}/WebView2Loader.dll")
set_property(TARGET Webview2 PROPERTY IMPORTED_IMPLIB "${CMAKE_CURRENT_SOURCE_DIR}/libs/${WEBVIEW2_ARCH}/WebView2Loader.dll.lib")

target_link_libraries(${PLUGIN_NAME} PRIVATE Webview2 uxtheme)

# List of absolute paths to libraries that should be bundled with the plugin
set(desktop_webview_window_bundled_libraries
        "$<TARGET_FILE:Webview2>"
        PARENT_SCOPE
        )
