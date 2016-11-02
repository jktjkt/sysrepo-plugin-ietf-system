project(sysrepo-plugin-ietf-system C)
cmake_minimum_required(VERSION 3.0)
# pkg_get_variable is unfortunately a 3.4+ feature

include(GNUInstallDirs)

find_package(PkgConfig)
pkg_check_modules(SYSREPO REQUIRED libsysrepo)

execute_process(COMMAND ${PKG_CONFIG_EXECUTABLE} --variable=SR_PLUGINS_DIR libsysrepo
    OUTPUT_VARIABLE SR_PLUGINS_DIR OUTPUT_STRIP_TRAILING_WHITESPACE
    RESULT_VARIABLE _sr_pkg_query_ok)

if(NOT ${_sr_pkg_query_ok} EQUAL 0)
    message(SEND_ERROR "Unable to determine SR_PLUGINS_DIR via pkg-config")
else()
    message(STATUS "  sysrepo plugins at ${SR_PLUGINS_DIR}")
endif()

set(CMAKE_CFLAGS "-Wall -pedantic -O2 ${CMAKE_CFLAGS}")

add_definitions(${SYSREPO_CFLAGS})
include_directories(${SYSREPO_INCLUDE_DIRS})
link_directories(${SYSREPO_LIBRARY_DIRS})

set(YANG_SRCS
    ${CMAKE_CURRENT_SOURCE_DIR}/iana-crypt-hash.yang
    ${CMAKE_CURRENT_SOURCE_DIR}/ietf-netconf-acm.yang
    ${CMAKE_CURRENT_SOURCE_DIR}/ietf-system.yang
)
add_custom_target(YANG_files ${YANG_SRCS})

add_library(sysrepo-plugin-ietf-system SHARED
    ${CMAKE_CURRENT_SOURCE_DIR}/system-plugin.c
)
target_link_libraries(sysrepo-plugin-ietf-system ${SYSREPO_LIBRARIES})

set(YANG_DIR ${CMAKE_INSTALL_PREFIX}/share/sysrepo/yang)

install(FILES ${YANG_SRCS} DESTINATION ${YANG_DIR})
install(TARGETS sysrepo-plugin-ietf-system DESTINATION ${SR_PLUGINS_DIR})
install(CODE "MESSAGE(STATUS
\"To enable this plugin, run:

- sysrepoctl --install -s ${YANG_DIR} --yang ${YANG_DIR}/ietf-system.yang
 - systemctl restart sysrepo-plugind
\")")
