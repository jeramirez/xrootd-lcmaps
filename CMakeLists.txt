
cmake_minimum_required( VERSION 2.6 )
project( xrootd-lcmaps )

set( CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake )

find_package( Xrootd REQUIRED )
find_package( Lcmaps REQUIRED )
find_package( Voms   REQUIRED )

if( CMAKE_COMPILER_IS_GNUCXX )
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -DLCMAPS_GSI_MODE=1" )
endif()

if( CMAKE_COMPILER_IS_GNUCC )
  set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -Wall -Werror -DLCMAPS_GSI_MODE=1" )
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")

include (FindPkgConfig)
pkg_check_modules(GLOBUS REQUIRED globus-gsi-credential globus-gsi-cert-utils globus-common globus-gsi-sysconfig globus-gsi-callback)
set (INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} ${GLOBUS_INCLUDE_DIRS})

include_directories( "${PROJECT_SOURCE_DIR}" "${XROOTD_INCLUDES}" "${LCMAPS_INCLUDES}" "${VOMS_INCLUDES}")

add_library(XrdLcmaps MODULE src/XrdLcmaps.cc src/XrdHttpLcmaps.cc src/XrdLcmapsConfig.cc src/XrdLcmapsKey.cc src/GlobusSupport.cc src/GlobusError.cc)
target_link_libraries(XrdLcmaps ${XROOTD_SECGSI} ${XROOTD_CRYPTOSSL} ${LCMAPS_LIB} ${DL_LIB} ${VOMS_LIB})
set_target_properties(XrdLcmaps PROPERTIES LINK_FLAGS "-Wl,--version-script=configs/export-symbols")

if (NOT DEFINED LIB_INSTALL_DIR)
  SET(LIB_INSTALL_DIR "lib")
endif()

install(
  FILES
  ${CMAKE_SOURCE_DIR}/configs/lcmaps.cfg
  DESTINATION ${SYSCONF_INSTALL_DIR}/xrootd
  PERMISSIONS
  OWNER_EXECUTE OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install(
  TARGETS XrdLcmaps
  LIBRARY DESTINATION ${LIB_INSTALL_DIR} )

set_target_properties(
  XrdLcmaps
  PROPERTIES
  VERSION   "0.0.2"
  SOVERSION "0"
  LINK_INTERFACE_LIBRARIES "" )

