cmake_minimum_required(VERSION 3.5)
project(eosio_wasm_sdk)

set(VERSION_MAJOR 1)
set(VERSION_MINOR 2)
set(VERSION_PATCH 0)

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
   message(WARNING "CMAKE_INSTALL_PREFIX is set to default path of ${CMAKE_INSTALL_PREFIX}, resetting to ${CMAKE_INSTALL_PREFIX}/eosio.cdt")
   set(CMAKE_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}/eosio.cdt")
elseif ("${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr/local")
   message(WARNING "CMAKE_INSTALL_PREFIX is explicitly set to /usr/local. This is not recommended.")
endif()

include(GNUInstallDirs)

include(ClangExternalProject.txt)

add_subdirectory(external)

set(WASM_SDK_BUILD true)

configure_file(${CMAKE_SOURCE_DIR}/modules/EosioWasmToolchainBuild.cmake.in ${CMAKE_BINARY_DIR}/lib/cmake/EosioWasmToolchain.cmake @ONLY)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_BINARY_DIR}/lib/cmake")
include(EosioWasmToolchain)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/modules")

if ("${CORE_SYMBOL_NAME}" STREQUAL "")
  set( CORE_SYMBOL_NAME "SYS" )
endif()
string(TOUPPER ${CORE_SYMBOL_NAME} CORE_SYMBOL_NAME)

string(LENGTH ${CORE_SYMBOL_NAME} CORE_SYMBOL_NAME_LENGTH)
if (CORE_SYMBOL_NAME_LENGTH GREATER 7)
  message(FATAL_ERROR "CORE_SYMBOL_NAME length must be between 1 and 7 characters")
endif()


add_subdirectory(libraries)

include(InstallClang.txt)

message( STATUS "Using '${CORE_SYMBOL_NAME}' as CORE symbol name" )

# install examples
#install(DIRECTORY examples DESTINATION "${CMAKE_INSTALL_PREFIX}")

configure_file(${CMAKE_SOURCE_DIR}/modules/EosioWasmToolchain.cmake.in ${CMAKE_BINARY_DIR}/modules/EosioWasmToolchain.cmake @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/modules/EosioWasmToolchain.cmake DESTINATION ./lib/cmake)
configure_file(${CMAKE_SOURCE_DIR}/eosio.imports.in ${CMAKE_BINARY_DIR}/eosio.imports COPYONLY)
install(FILES ${CMAKE_BINARY_DIR}/eosio.imports DESTINATION .)

configure_file(${CMAKE_SOURCE_DIR}/scripts/ricardeos/ricardeos.py ${CMAKE_BINARY_DIR}/scripts/ricardeos.py COPYONLY)
install(FILES ${CMAKE_BINARY_DIR}/scripts/ricardeos.py DESTINATION ./scripts)

include (InstallRequiredSystemLibraries)
if (WIN32)
elseif (APPLE)
   #set (CPACK_GENERATOR "DragNDrop")
   #set (CPACK_GENERATOR "OSXX11")
   install(FILES ${CMAKE_SOURCE_DIR}/setup.sh DESTINATION .)
   set (CPACK_GENERATOR "TGZ")
   set (CPACK_BUNDLE_NAME "eosio.cdt")
	#CFBundleGetInfoString
   #set(MACOSX_BUNDLE_INFO_STRING "${CPACK_PACKAGE_DESCRIPTION_SUMMARY} Version ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}, Copyright ${CPACK_PACKAGE_VENDOR}")
	#set(MACOSX_BUNDLE_ICON_FILE "eosio.cdt.icns")
	#set(MACOSX_BUNDLE_GUI_IDENTIFIER "${CPACK_PACKAGE_DESCRIPTION_SUMMARY}")
	##CFBundleLongVersionString
   #set(MACOSX_BUNDLE_LONG_VERSION_STRING "${CPACK_PACKAGE_DESCRIPTION_SUMMARY} Version ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
   #set(MACOSX_BUNDLE_SHORT_VERSION_STRING ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})
   #set (CPACK_PACKAGE_FILE_NAME "eosio.cdt-${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
   #set (CPACK_PACKAGE_ICON "${CMAKE_SOURCE_DIR}/eos-logo.ico")
   #set (CPACK_BUNDLE_ICON "${CMAKE_SOURCE_DIR}/eos-logo.ico")

   #configure_file(${CMAKE_SOURCE_DIR}/Info.plist.in ${CMAKE_BINARY_DIR}/Info.plist @ONLY)
   #set (CPACK_BUNDLE_PLIST ${CMAKE_BINARY_DIR}/Info.plist)
   #set (CPACK_BUNDLE_STARTUP_COMMAND "${CMAKE_SOURCE_DIR}/startup.sh")
else()
   set(CPACK_GENERATOR "DEB")
   set(CPACK_DEBIAN_PACKAGE_RELEASE 0)
   if (CMAKE_VERSION VERSION_GREATOR 3.6.0)
      set(CPACK_DEBIAN_FILE_NAME "DEB-DEFAULT")
   else()
      string(TOLOWER ${CPACK_PACKAGE_NAME} CPACK_DEBIAN_PACKAGE_NAME)
      execute_process(COMMAND dpkg --print-architecture OUTPUT_VARIABLE CPACK_DEBIAN_PACKAGE_ARCHITECTURE OUTPUT_STRIP_TRAILING_WHITESPACE)
      set (CPACK_PACKAGE_FILE_NAME ${CPACK_DEBIAN_PACKAGE_NAME}_${CPACK_PACKAGE_VERSION}-${CPACK_DEBIAN_PACKAGE_RELEASE}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE})
   endif()
   set (CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
   set (CPACK_INCLUDE_TOPLEVEL_DIRECTORY TRUE)
   set (CPACK_DEBIAN_PACKAGE_CONTROL_STRICT_PERMISSION TRUE)
   set (CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/eosio/eosio.cdt")
endif()

set (CPACK_PACKAGE_CONTACT "support@block.one")
set (CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/packages)
set (CPACK_PACKAGE_DIRECTORY "${CMAKE_BINARY_DIR}/install")
set (CPACK_PACKAGE_NAME "EOSIO.CDT")
set (CPACK_PACKAGE_VENDOR "block.one")
set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
set (CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
set (CPACK_PACKAGE_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set (CPACK_PACKAGE_DESCRIPTION "Toolkit for developing EOSIO smart contracts")
set (CPACK_PACKAGE_INSTALL_DIRECTORY "/usr/local/eosio.cdt-${CPACK_PACKAGE_VERSION}") #EOSIO.CDT ${CPACK_PACKAGE_VERSION}")
set (CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})

include (CPack)
