SET(ODBC_BINARY_DIR ${CMAKE_SOURCE_DIR}/${CMAKE_BUILD_TYPE})
MESSAGE(STATUS "BINARY_DIR ${ODBC_BINARY_DIR}")
# Get revision number
EXECUTE_PROCESS(COMMAND bzr log -l1
                COMMAND FINDSTR revno
  OUTPUT_VARIABLE revno)
STRING(REPLACE "revno: " "" revno ${revno})
STRING(REPLACE "\n" "" revno ${revno})
SET(WIX_DIR "$ENV{WIX}/bin/")
SET(PRODUCT_NAME "MariaDB ODBC Driver")
SET(PRODUCT_MANUFACTURER "MariaDB")
SET(PRODUCT_VERSION "${MARIADB_ODBC_VERSION_MAJOR}.${MARIADB_ODBC_VERSION_MINOR}.${MARIADB_ODBC_VERSION_PATCH}")

IF(WITH_SIGNCODE)
ENDIF()

IF(${MSI_64})
  SET(PRODUCT_NAME "${PRODUCT_NAME} 64-bit")
  SET(PLATFORM "win64")
  SET(IS_WIN64 "yes")
  SET(WIXPLATFORM "x64")
  SET(FOLDER "ProgramFiles64Folder")
  SET(GUID_REGISTRY "E35BF41F-89A1-4691-8F62-09922C04C13B")
  SET(GUID_SETUP "BF255F46-18CA-4244-9A17-6B33BDAAFBF4")
  SET(GUID_DRIVERA "3535FD95-1F44-454E-A6E4-81F865E8C57F")
  SET(GUID_DRIVERU "DFB1DA9D-C236-40BF-943A-F11D4BD275B7")
  SET(GUID_DEBUG "4FA6E79A-4630-4CB8-A4E1-00A4740E9280")
ELSE()
  SET(PLATFORM "win32")
  SET(IS_WIN64 "no")
  SET(WIXPLATFORM "x86")
  SET(FOLDER "ProgramFilesFolder")
  SET(GUID_REGISTRY "ACFC9B33-5D1F-4EA2-A4DB-1E37A2BAF86B")
  SET(GUID_SETUP "16E13D0B-7BFE-4BC4-A524-940716EE749F")
  SET(GUID_DRIVERA "8BD16D93-30E0-4DF0-8B40-9A5A3D967DD6")
  SET(GUID_DRIVERU "85447DF4-3BAA-4166-9870-836D6D080E66")
  SET(GUID_DEBUG "2EA8B4DD-F470-4362-8D87-59090D255981")
ENDIF()

CONFIGURE_FILE(${CMAKE_SOURCE_DIR}/wininstall/mariadb_odbc.xml.in
               ${CMAKE_SOURCE_DIR}/wininstall/mariadb_odbc.xml)
IF(${revno})
  SET(MSI_PACKAGE "mariadb-odbc-${PRODUCT_VERSION}-r${revno}-${PLATFORM}.msi")
ELSE()
  SET(MSI_PACKAGE "mariadb-odbc-${PRODUCT_VERSION}-${PLATFORM}.msi")
ENDIF()

IF(WITH_SIGNCODE)
  SET(SIGN_OPTIONS " /a /t http://timestamp.verisign.com/scripts/timstamp.dll ")
  EXECUTE_PROCESS(
	  COMMAND signtool sign ${SIGN_OPTIONS} ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/maodbca.dll)
  EXECUTE_PROCESS(
	  COMMAND signtool sign ${SIGN_OPTIONS} ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/maodbcs.dll)
  EXECUTE_PROCESS(
	  COMMAND signtool sign ${SIGN_OPTIONS} ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/maodbcu.dll)
ENDIF()

ADD_CUSTOM_TARGET(
        MSI_PACKAGE_ALL
        DEPENDS ${MSI_PACKAGE} maodbca maodbcu maodbcs)

ADD_CUSTOM_COMMAND(
        OUTPUT ${MSI_PACKAGE}
        DEPENDS mariadb_odbc.wixobj
                mariadb_odbc_fragment.wixobj
                COMMAND ${WIX_DIR}light.exe -ext WixUIExtension mariadb_odbc.wixobj -o ${MSI_PACKAGE})

ADD_CUSTOM_COMMAND(
        OUTPUT mariadb_odbc.wixobj
        DEPENDS mariadb_odbc.xml
        COMMAND ${WIX_DIR}candle.exe mariadb_odbc.xml -o mariadb_odbc.wixobj)
