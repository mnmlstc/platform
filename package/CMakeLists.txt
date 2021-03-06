list(APPEND GENERATORS "TGZ" "ZIP")

if (BUILD_PACKAGE_MSI)
  list(APPEND GENERATORS "WIX")
endif ()

if (BUILD_PACKAGE_DEB)
  list(APPEND GENERATORS "DEB")
endif ()

if (BUILD_PACKAGE_RPM)
  list(APPEND GENERATORS "RPM")
endif ()

#if (BUILD_PACKAGE_PKG)
#  list(APPEND GENERATORS "PKG")
#endif ()

#------------------------------------------------------------------------------
# Generic Package
#------------------------------------------------------------------------------
set(CPACK_OUTPUT_FILE_PREFIX ${PACK_BINARY_DIR})

set(CPACK_GENERATOR ${GENERATORS})
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "MNMLSTC")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++11 Platform Checking Library")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${PROJECT_SOURCE_DIR}/Readme.rst")
set(CPACK_RESOURCE_FILE_LICENSE "${PROJECT_SOURCE_DIR}/License.rst")

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

# Intended for installers such as NSIS or WIX
set(CPACK_PACKAGE_INSTALL_DIRECTORY "mnmlstc/${PROJECT_NAME}")

#------------------------------------------------------------------------------
# Wix Configuration
#------------------------------------------------------------------------------
set(CPACK_WIX_CMAKE_PACKAGE_REGISTRY ${PROJECT_NAME})
set(CPACK_WIX_UPGRADE_GUID "A72EB4BE-1FD6-44D4-BB5E-D102103645B4")
set(CPACK_WIX_PRODUCT_GUID "*")
set(CPACK_WIX_PRODUCT_ICON "${PACK_SOURCE_DIR}/mnmlstc.ico")
set(CPACK_WIX_LICENSE_RTF "${PACK_SOURCE_DIR}/License.rtf")
set(CPACK_WIX_UI_BANNER "${PACK_SOURCE_DIR}/banner.bmp")
set(CPACK_WIX_UI_DIALOG "${PACK_SOURCE_DIR}/dialog.bmp")

#------------------------------------------------------------------------------
# DEB Configuration
#------------------------------------------------------------------------------
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE any)

#------------------------------------------------------------------------------
# RPM Configuration
#------------------------------------------------------------------------------
set(CPACK_RPM_PACKAGE_ARCHITECTURE noarch)
set(CPACK_RPM_PACKAGE_REQUIRES "cmake >= 2.8.11")
set(CPACK_RPM_PACKAGE_RELEASE ${PLATFORM_VERSION})

include(CPack)
