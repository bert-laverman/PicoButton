# CMakeLists template for Raspberry Pi Pico projects
#

# Set your target app name here
set(APP_NAME pico-button)

cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 20)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_FLAGS_RELEASE "-O2")

# Set CMake defines for the Raspberry Pi Pico and required components
set(TARGET_ZERO2W off)
set(TARGET_PICO on)

set(HAVE_I2C off)
set(HAVE_SPI off)
set(HAVE_PWM off)

set(HAVE_MAX7219 off)
set(HAVE_LCD2X16 off)

set(HAVE_LED on)
set(HAVE_BUTTON on)

#
# The pico-sdk location is taken from the PICO_SDK_PATH environment variable
#if(DEFINED $ENV{PICO_SDK_PATH})
  set(PICO_SDK_PATH $ENV{PICO_SDK_PATH})
#else()
#  set(PICO_SDK_PATH "/home/user/pico-sdk")
#endif()

#
# The CppRaspberry library location is taken from the CPP_RASPBERRY_PATH environment variable
if(DEFINED $ENV{CPP_RASPBERRY_PATH})
  set(CPP_RASPBERRY_PATH $ENV{CPP_RASPBERRY_PATH})
else()
  # If this is not set, you can set it here
  set(CPP_RASPBERRY_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../CppRaspberry")
endif()

set(PICO_BOARD pico CACHE STRING "Board type")
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

# initialize the SDK based on PICO_SDK_PATH
# note: this must happen before project()
include(${PICO_SDK_PATH}/external/pico_sdk_import.cmake)

if (PICO_SDK_VERSION_STRING VERSION_LESS "1.4.0")
  message(FATAL_ERROR "Raspberry Pi Pico SDK version 1.4.0 (or later) required. Your version is ${PICO_SDK_VERSION_STRING}")
endif()

project(${APP_NAME} C CXX ASM)

set(PICO_CXX_ENABLE_EXCEPTIONS 1)

# initialize the Raspberry Pi Pico SDK
pico_sdk_init()

# include the CppRaspberry library
include(${CPP_RASPBERRY_PATH}/CppRaspberry.cmake)

# And now our local project
add_executable(${APP_NAME} ${CMAKE_CURRENT_LIST_DIR}/src/pico-button.cpp)

# Add CppRaspberry library
add_cpp_raspberry_app(${APP_NAME})

# create map/bin/hex/uf2 file in addition to ELF.
pico_add_extra_outputs(${APP_NAME})

pico_set_program_name(${APP_NAME} "${APP_NAME}")
pico_set_program_version(${APP_NAME} "0.1")

pico_enable_stdio_uart(${APP_NAME} 0)
pico_enable_stdio_usb(${APP_NAME} 1)

target_include_directories(${APP_NAME} PUBLIC ${CMAKE_CURRENT_LIST_DIR}/include)

target_compile_options(${APP_NAME} PRIVATE -Wall -Wextra -Werror)

target_link_libraries(${APP_NAME} pico_stdlib pico_stdio_usb)
