cmake_minimum_required(VERSION 3.0)
project(LSLOpenVR VERSION 0.0.1)
include(Findliblsl.cmake)

set(META_PROJECT_DESCRIPTION "Capture OpenVR pose data.")

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
find_package(Qt5 REQUIRED COMPONENTS Network Xml Widgets)

# OpenVR
LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
include(ExternalProject)
IF(NOT OpenVR_ROOT_DIR)
    message(STATUS "OpenVR_ROOT_DIR not set.")
    message(STATUS "\tUse cmake [...] -DOpenVR_ROOT_DIR=<path/to/openvr>")
    message(STATUS "\tAttempting Chad's default rel directory...")
    SET(OpenVR_ROOT_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../../VR/openvr")
ENDIF()
find_package(OpenVR)
IF(NOT OpenVR_FOUND)
    message(STATUS "OpenVR not found. It will be downloaded from GitHub during configure. This can be very slow.")
    ExternalProject_Add(openvr
        PREFIX ${CMAKE_CURRENT_LIST_DIR}/deps/openvr
        DOWNLOAD_NO_PROGRESS 0  # Do not disable download progress.
        GIT_REPOSITORY https://github.com/ValveSoftware/openvr.git
        GIT_SHALLOW 1
        BUILD_COMMAND "" # Disable build step.
        INSTALL_COMMAND "" # Disable install step.
    )
    ExternalProject_Get_Property(openvr SOURCE_DIR)
    # Cannot use find_package because the project is not downloaded yet.
    # Instead, assume file locations.
    set(SIZEOF_VOIDP ${CMAKE_SIZEOF_VOID_P})
    if((NOT APPLE) AND (CMAKE_SIZEOF_VOID_P EQUAL 8))
      set(PROCESSOR_ARCH "64")
    else()
      set(PROCESSOR_ARCH "32")
    endif()
    if(WIN32)
        set(PLATFORM_NAME "win")
    elseif(UNIX AND NOT APPLE)
      if(CMAKE_SYSTEM_NAME MATCHES ".*Linux")
        set(PLATFORM_NAME "linux")
      endif()
    elseif(APPLE)
      if(CMAKE_SYSTEM_NAME MATCHES ".*Darwin.*" OR CMAKE_SYSTEM_NAME MATCHES ".*MacOS.*")
        set(PLATFORM_NAME "osx")
      endif()
    endif()
    SET(OpenVR_INCLUDE_DIRS ${SOURCE_DIR}/headers)
    SET(OpenVR_BINARIES ${SOURCE_DIR}/bin/${PLATFORM_NAME}${PROCESSOR_ARCH}/${CMAKE_SHARED_LIBRARY_PREFIX}openvr_api${CMAKE_SHARED_LIBRARY_SUFFIX})
    SET(OpenVR_LIBRARIES ${SOURCE_DIR}/lib/${PLATFORM_NAME}${PROCESSOR_ARCH}/${CMAKE_SHARED_LIBRARY_PREFIX}openvr_api${CMAKE_IMPORT_LIBRARY_SUFFIX})
ENDIF()

###############
# APPLICATION #
###############
add_subdirectory(src) 
