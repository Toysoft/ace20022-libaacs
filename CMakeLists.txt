cmake_minimum_required(VERSION 3.0)

project(libaacs VERSION 0.8.2 LANGUAGES C)

#find_package(libpgp-error 1.2.7 REQUIRED NO_MODULE)

if(MSVC)
  #set(CMAKE_DEBUG_POSTFIX "d")
endif()

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/cmake/win_bison.exe)
  set(BISON_CMD ${CMAKE_CURRENT_SOURCE_DIR}/cmake/win_bison)
  message("BISON command: ${BISON_CMD}")

else()
  message("BISON path not set, can't generate error codes")
  return()
endif()
if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/cmake/win_flex.exe)
  set(FLEX_CMD ${CMAKE_CURRENT_SOURCE_DIR}/cmake/win_flex)
  message("FLEX command: ${FLEX_CMD}")

else()
  message("FLEX path not set, can't generate error codes")
  return()
endif()

   execute_process(COMMAND ${FLEX_CMD} -L ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-lexer.l)
   execute_process(COMMAND ${CMAKE_COMMAND} -E rename ${CMAKE_CURRENT_BINARY_DIR}/lex.yy.c
                   ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-lexer.c)
   execute_process(COMMAND ${BISON_CMD} -l -y -d -p libaacs_yy -o ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-parser.c ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-parser.y)
set(SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/libaacs.def
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/aacs-version.h
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/headers_fixup.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/aacs.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/aacs.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/cci_data.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/cci.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/cci.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/content_cert.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/content_cert.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/crypto.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/crypto.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/mkb.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/mkb.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/mmc.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/mmc.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/unit_key.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/unit_key.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/dirs.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/file.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/file.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/filesystem.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/filesystem.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydb.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-parser.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-parser.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/keydbcfg-lexer.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/mmc_device.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/path.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/util/attributes.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/util/macro.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/util/logging.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/util/logging.h
  ${CMAKE_CURRENT_SOURCE_DIR}/src/util/strutl.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/util/strutl.h
#  ${CMAKE_CURRENT_SOURCE_DIR}/src/libaacs/ec.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/dirs_win32.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/file_win32.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/file/mmc_device_win32.c
)

ADD_LIBRARY(libaacs SHARED ${SRCS})
SET_TARGET_PROPERTIES(libaacs PROPERTIES LINK_FLAGS "/DEF:\"libaacs.def\"")

target_compile_definitions(libaacs
  PRIVATE
  HAVE_CONFIG_H
  PKGDATADIR=""
  _CRT_SECURE_NO_WARNINGS
  _CRT_NONSTDC_NO_WARNINGS
  __PRETTY_FUNCTION__=__FUNCTION__
)
  
target_include_directories(libaacs
  PRIVATE
  $<BUILD_INTERFACE:.;cmake;src;src/util;${CMAKE_CURRENT_BINARY_DIR}>
  INTERFACE
  $<INSTALL_INTERFACE:include/libaacs>
)

target_link_libraries(libaacs PRIVATE "C:/XBMC/libgcrypt-1.7.6_cmake/build/RelWithDebInfo/libgcrypt.lib")

set(EX_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/examples/aacs_info.c
)

ADD_EXECUTABLE(aacsinfo ${EX_SRCS})
target_compile_definitions(aacsinfo
  PRIVATE
  HAVE_CONFIG_H
  _CRT_SECURE_NO_WARNINGS
  _CRT_NONSTDC_NO_WARNINGS)
  
target_include_directories(aacsinfo
  PRIVATE
  $<BUILD_INTERFACE:.;cmake;src;${CMAKE_CURRENT_BINARY_DIR}>
  INTERFACE
)
add_dependencies(aacsinfo libaacs)
target_link_libraries(aacsinfo PRIVATE libaacs)