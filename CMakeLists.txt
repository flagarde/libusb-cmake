cmake_minimum_required(VERSION 3.15)

set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(GetCMakeMM)
cmmm(VERSION "2.0" REPOSITORY "flagarde/CMakeMM" PROVIDER "gitlab" VERBOSITY VERBOSE)

cmmm_modules_list(REPOSITORY "flagarde/CMakeCM" PROVIDER "gitlab")

project(LibUSB C)

include(CPM)
cpm()

if(NOT DEFINED LIBUSB-ZERO_TAG)
  set(LIBUSB-ZERO_TAG "main")  
endif()

if(NOT DEFINED LIBUSB-ZERO_REPOSITORY)
  set(LIBUSB-ZERO_REPOSITORY "https://github.com/flagarde/libudev-zero.git")
endif()

CPMAddPackage(NAME libusb-zero
              GIT_REPOSITORY ${LIBUSB-ZERO_REPOSITORY}
              GIT_TAG ${LIBUSB-ZERO_TAG}
              FETCHCONTENT_UPDATES_DISCONNECTED ${IS_OFFLINE}
              OPTIONS "${libusb-zero_OPTIONS}")

find_package(Threads REQUIRED)

if (WIN32)
set(OS_SOURCES
    os/threads_windows.h 
    os/threads_windows.c
    os/poll_windows.h 
    os/poll_windows.c
    libusb-1.0.def
    libusb-1.0.rc
    os/windows_common.h
    os/windows_nt_common.h 
    os/windows_nt_common.c
	os/windows_nt_shared_types.h
	os/windows_usbdk.h 
    os/windows_usbdk.c
	os/windows_winusb.h
    os/windows_winusb.c
    os/msvc/config.h
  )
set(LibUSB_HEADER_PATHS
    ./ 
    ./os 
    ./os/msvc
    )
else()

set(OS_SOURCES
    os/threads_posix.h 
    os/threads_posix.c
    os/poll_posix.h 
    os/poll_posix.c
    os/linux_usbfs.h 
    os/linux_usbfs.c
    os/linux_udev.c
  )

set(LibUSB_HEADER_PATHS
    ./ 
    ./os
    ./os/linux
    )
endif()

set(LibUSB_SOURCES
    libusb.h 
    libusbi.h
    version.h 
    version_nano.h
	core.c 
    descriptor.c 
    hotplug.h 
    hotplug.c 
    io.c 
    strerror.c 
    sync.c
  )

add_library(LibUSB ${LibUSB_SOURCES} ${OS_SOURCES})
if(UNIX)
  target_link_libraries(LibUSB PRIVATE Threads::Threads PRIVATE libudev-zero)
else()
  target_link_libraries(LibUSB PRIVATE Threads::Threads PRIVATE libudev-zero)
endif()
target_include_directories(LibUSB PUBLIC ${LibUSB_HEADER_PATHS})
install(TARGETS LibUSB)


add_subdirectory(test)

