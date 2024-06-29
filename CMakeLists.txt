cmake_minimum_required(VERSION 3.5)
project(parrot_arsdk)

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(ament_cmake_ros REQUIRED)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

# use the build space for downloading/building the package
# we populate the results to install space after the build

set(ARSDK_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include)
set(ARSDK_LIB_DIR ${CMAKE_INSTALL_PREFIX}/lib/${PROJECT_NAME})
set(ARSDK_LIBS arcommands arcontroller ardatatransfer ardiscovery armavlink armedia arnetworkal arnetwork arsal arstream2 arstream arupdater arutils)

set(ARDRONESDK3_DEVEL_PATH ${CMAKE_CURRENT_BINARY_DIR}/arsdk)
set(ARDRONESDK3_PATH ${ARDRONESDK3_DEVEL_PATH}/src/ARSDKBuildUtils/out/arsdk-native/staging/usr)

# 3.14.0 patch 1
set(ARSDK_ARCHIVE arsdk_3_14_0_p1_stripped.tgz)
set(ARSDK_ARCHIVE_MD5 72dd7575d3cd67522dcfc3f42c24c5a7)

# Determine the architecture of the host in a robust way
execute_process(COMMAND python3 ${PROJECT_SOURCE_DIR}/script/get_arch.py OUTPUT_VARIABLE BUILD_HOST_ARCH)
message(STATUS "Host architecture: ${BUILD_HOST_ARCH}")

add_custom_target(ARSDK_MKDIR
  COMMAND ${CMAKE_COMMAND} -E make_directory ${ARDRONESDK3_DEVEL_PATH})

include(ExternalProject)
ExternalProject_Add(ARSDKBuildUtils
    DEPENDS ARSDK_MKDIR
    URL ${PROJECT_SOURCE_DIR}/sdk/${ARSDK_ARCHIVE}
    URL_MD5 ${ARSDK_ARCHIVE_MD5}
    PREFIX ${ARDRONESDK3_DEVEL_PATH}
    CONFIGURE_COMMAND echo "No configure"
    BUILD_COMMAND TARGET_ARCH=${BUILD_HOST_ARCH} ./build.sh -p arsdk-native -t build-sdk -j --no-color
    INSTALL_COMMAND echo "No SDK install command"
    BUILD_IN_SOURCE 1
)

add_custom_target(ARSDKPopulate ALL
  COMMENT "Populating ARDroneSDK to install space: ${ARDRONESDK3_DEVEL_PATH}"
  WORKING_DIRECTORY ${ARDRONESDK3_DEVEL_PATH}/src/ARSDKBuildUtils
  # Populate the build results into the install space
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/include
  COMMAND cp -rp ./out/arsdk-native/staging/usr/include/* ${CMAKE_INSTALL_PREFIX}/include
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/lib/${PROJECT_NAME}
  COMMAND cp -rp ./out/arsdk-native/staging/usr/lib/* ${CMAKE_INSTALL_PREFIX}/lib/${PROJECT_NAME}
  COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME}
  COMMAND cp -rp ./out/arsdk-native/staging/usr/share/* ${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME}
)
add_dependencies(ARSDKPopulate ARSDKBuildUtils)

install(DIRECTORY ${ARDRONESDK3_PATH}/include/
  DESTINATION include
)

install(DIRECTORY ${ARDRONESDK3_PATH}/lib/
  DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY ${ARDRONESDK3_PATH}/share/
  DESTINATION share/${PROJECT_NAME}
)

install(DIRECTORY ${ARDRONESDK3_PATH}/../etc/
  DESTINATION etc/${PROJECT_NAME}
)

ament_package()