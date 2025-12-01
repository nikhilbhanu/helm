cmake_minimum_required(VERSION 3.25)

# Project configuration
set(PROJECT_NAME "Helm")
set(PRODUCT_NAME "Helm")
set(COMPANY_NAME "Matt Tytel")
set(BUNDLE_ID "org.tytel.helmplugin")
set(FORMATS AU VST3 Standalone)

# Version from helm.jucer
project(${PROJECT_NAME} VERSION 0.9.0)

# Configure for Apple Silicon
if(APPLE)
    set(CMAKE_OSX_ARCHITECTURES "arm64" CACHE STRING "Build architectures for macOS" FORCE)
    set(CMAKE_OSX_DEPLOYMENT_TARGET "15" CACHE STRING "Minimum macOS version" FORCE)
endif()

# Add submodules
add_subdirectory(third_party/JUCE)
add_subdirectory(third_party/concurrentqueue)

# Gather all source files
file(GLOB_RECURSE SourceFiles
    "${CMAKE_CURRENT_SOURCE_DIR}/src/common/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/editor_components/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/editor_sections/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/look_and_feel/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/plugin/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/synthesis/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/mopo/src/*.cpp"
)

file(GLOB_RECURSE HeaderFiles
    "${CMAKE_CURRENT_SOURCE_DIR}/src/common/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/editor_components/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/editor_sections/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/look_and_feel/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/plugin/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/synthesis/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/mopo/src/*.h"
)

# Create binary data target for resources
juce_add_binary_data(HelmBinaryData SOURCES

    # Fonts
    fonts/DroidSansMono.ttf
    fonts/Roboto-Light.ttf
    fonts/Roboto-Regular.ttf
    fonts/Roboto-Thin.ttf

    # Images
    images/helm_debian_icon.xpm
    images/helm_icon_16_1x.png
    images/helm_icon_16_2x.png
    images/helm_icon_22_1x.png
    images/helm_icon_24_1x.png
    images/helm_icon_32_1x.png
    images/helm_icon_32_2x.png
    images/helm_icon_48_1x.png
    images/helm_icon_128_1x.png
    images/helm_icon_128_2x.png
    images/helm_icon_256_1x.png
    images/helm_icon_256_2x.png
    images/helm_icon_512_1x.png
    images/helm_icon_512_2x.png
    images/modulation_selected_active_1x.png
    images/modulation_selected_active_2x.png
    images/modulation_selected_inactive_1x.png
    images/modulation_selected_inactive_2x.png
    images/modulation_unselected_active_1x.png
    images/modulation_unselected_active_2x.png
    images/modulation_unselected_inactive_1x.png
    images/modulation_unselected_inactive_2x.png
)

# Create the plugin
juce_add_plugin(${PROJECT_NAME}
    COMPANY_NAME "${COMPANY_NAME}"
    BUNDLE_ID "${BUNDLE_ID}"
    PLUGIN_MANUFACTURER_CODE Tyte
    PLUGIN_CODE Helm
    FORMATS "${FORMATS}"
    PRODUCT_NAME "${PRODUCT_NAME}"

    # Plugin characteristics
    IS_SYNTH TRUE
    NEEDS_MIDI_INPUT TRUE
    NEEDS_MIDI_OUTPUT FALSE
    IS_MIDI_EFFECT FALSE
    EDITOR_WANTS_KEYBOARD_FOCUS TRUE

    # Copy plugin after build on macOS
    COPY_PLUGIN_AFTER_BUILD TRUE

    # Icon
    ICON_BIG "${CMAKE_CURRENT_SOURCE_DIR}/images/helm_icon_512_2x.png"
)

# Add source files to the plugin target
target_sources(${PROJECT_NAME} PRIVATE ${SourceFiles} ${HeaderFiles})

# Include directories
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/src/common
    ${CMAKE_CURRENT_SOURCE_DIR}/src/editor_components
    ${CMAKE_CURRENT_SOURCE_DIR}/src/editor_sections
    ${CMAKE_CURRENT_SOURCE_DIR}/src/look_and_feel
    ${CMAKE_CURRENT_SOURCE_DIR}/src/plugin
    ${CMAKE_CURRENT_SOURCE_DIR}/src/synthesis
    ${CMAKE_CURRENT_SOURCE_DIR}/mopo/src
)

# Compile definitions
target_compile_definitions(${PROJECT_NAME} PUBLIC
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_CAN_REPLACE_VST2=0
    JUCE_USE_XRANDR=0
    JUCE_DISPLAY_SPLASH_SCREEN=0
)

# Link libraries
target_link_libraries(${PROJECT_NAME} PRIVATE
    HelmBinaryData
    concurrentqueue
    juce::juce_audio_basics
    juce::juce_audio_devices
    juce::juce_audio_formats
    juce::juce_audio_plugin_client
    juce::juce_audio_processors
    juce::juce_audio_utils
    juce::juce_core
    juce::juce_data_structures
    juce::juce_events
    juce::juce_graphics
    juce::juce_gui_basics
    juce::juce_gui_extra
    juce::juce_opengl
    juce::juce_recommended_config_flags
    juce::juce_recommended_lto_flags
    juce::juce_recommended_warning_flags
)

juce_generate_juce_header(${PROJECT_NAME})

# Enable C++23
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_23)

# Copy patches to user presets directory after build
if(APPLE)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory "/Library/Audio/Presets/Helm"
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_SOURCE_DIR}/patches"
        "/Library/Audio/Presets/Helm"
        COMMENT "Copying patches to /Library/Audio/Presets/Helm"
    )
endif()
