# == DO NEVER EDIT THE NEXT LINES for Raspberry Pi Pico VS Code Extension to work ==
if(WIN32)
   set(USERHOME $ENV{USERPROFILE})
else()
    set(USERHOME $ENV{HOME})
endif()
set(PICO_SDK_PATH ${USERHOME}/.pico-sdk/sdk/1.5.1)
set(PICO_EXTRAS_PATH ${USERHOME}/.pico-extras)
set(PICO_TOOLCHAIN_PATH ${USERHOME}/.pico-sdk/toolchain/13_2_Rel1)
if(WIN32)
    set(pico-sdk-tools_DIR ${USERHOME}/.pico-sdk/tools/1.5.1)
    include(${pico-sdk-tools_DIR}/pico-sdk-tools-config.cmake)
    include(${pico-sdk-tools_DIR}/pico-sdk-tools-config-version.cmake)
endif()
# ====================================================================================
set(PICO_BOARD pico_w CACHE STRING "Board type")
set(DISPLAY_PATH display/cosmic/cosmic_unicorn.cmake)
cmake_minimum_required(VERSION 3.12)

set(NAME blunicorn)

include(pico_sdk_import.cmake)
include(pico_extras_import.cmake)

project(${NAME} C CXX ASM)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

pico_sdk_init()

include(bluetooth/bluetooth.cmake)
include(effect/rainbow_fft.cmake)
include(effect/classic_fft.cmake)

if(DISPLAY_PATH AND EXISTS ${CMAKE_CURRENT_LIST_DIR}/${DISPLAY_PATH})
include(${CMAKE_CURRENT_LIST_DIR}/${DISPLAY_PATH})
else()
message(WARNING "Using default display (display/galactic/galactic_unicorn.cmake)...")
include(display/galactic/galactic_unicorn.cmake)
endif()

add_executable(${NAME}
    ${CMAKE_CURRENT_LIST_DIR}/src/main.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/a2dp_sink.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/btstack_audio_pico.cpp
)

target_include_directories(${NAME} PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
    ${CMAKE_CURRENT_LIST_DIR}/effect
)

target_link_libraries(${NAME}
    picow_bt_example_common
    pico_audio_i2s
    pico_multicore
    display
    rainbow_fft
    classic_fft
)

message(WARNING "Display: ${DISPLAY_NAME}")

target_compile_definitions(${NAME} PRIVATE
    PICO_AUDIO_I2S_DATA_PIN=9
    PICO_AUDIO_I2S_CLOCK_PIN_BASE=10
    BLUETOOTH_DEVICE_NAME="${DISPLAY_NAME}"
)

pico_enable_stdio_usb(${NAME} 1)
pico_add_extra_outputs(${NAME})

# Set up files for the release packages
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${NAME}.uf2
    ${CMAKE_CURRENT_LIST_DIR}/README.md
    DESTINATION .
)

set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_GENERATOR "ZIP" "TGZ")
include(CPack)