cmake_minimum_required(VERSION 3.10)
project(n2n C)

# Add cmake module path
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")

# Determine version
execute_process(
    COMMAND sh ${CMAKE_SOURCE_DIR}/scripts/version.sh
    OUTPUT_VARIABLE N2N_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
string(REGEX REPLACE "^([0-9.]+).*" "\\1" PACKAGE_VERSION "${N2N_VERSION}")

execute_process(
    COMMAND sh ${CMAKE_SOURCE_DIR}/scripts/version.sh date
    OUTPUT_VARIABLE PACKAGE_BUILDDATE
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

set(PACKAGE_NAME "n2n")
set(PACKAGE_STRING "${PACKAGE_NAME} ${PACKAGE_VERSION}")
set(PACKAGE_TARNAME "${PACKAGE_NAME}")
set(PACKAGE_BUGREPORT "https://github.com/ntop/n2n/issues")
set(PACKAGE_URL "https://github.com/ntop/n2n")

message(STATUS "Configuring ${PACKAGE_STRING}")
message(STATUS "Build date: ${PACKAGE_BUILDDATE}")

# Options
option(N2N_OPTION_ZSTD "Enable ZSTD support" ON)
option(N2N_OPTION_OPENSSL "Enable OpenSSL support" ON)
option(N2N_OPTION_PCAP "Enable PCAP support" OFF)
option(N2N_OPTION_CAP "Enable CAP support" OFF)
option(N2N_OPTION_MINIUPNP "Enable MiniUPnP support" OFF)
option(N2N_OPTION_NATPMP "Enable NAT-PMP support" OFF)
option(N2N_OPTION_PTHREAD "Enable Pthread support" ON)

# Dependencies
find_package(PkgConfig)

if(N2N_OPTION_ZSTD)
    pkg_check_modules(ZSTD libzstd)
    if(ZSTD_FOUND)
        set(HAVE_LIBZSTD 1)
        include_directories(${ZSTD_INCLUDE_DIRS})
        link_directories(${ZSTD_LIBRARY_DIRS})
    endif()
endif()

if(N2N_OPTION_OPENSSL)
    find_package(OpenSSL)
    if(OPENSSL_FOUND)
        set(HAVE_LIBCRYPTO 1)
        include_directories(${OPENSSL_INCLUDE_DIR})
    endif()
endif()

if(N2N_OPTION_PCAP)
    pkg_check_modules(PCAP libpcap)
    if(PCAP_FOUND)
        set(HAVE_LIBPCAP 1)
        include_directories(${PCAP_INCLUDE_DIRS})
        link_directories(${PCAP_LIBRARY_DIRS})
    endif()
endif()

if(N2N_OPTION_CAP)
    pkg_check_modules(CAP libcap)
    if(CAP_FOUND)
        set(HAVE_LIBCAP 1)
        include_directories(${CAP_INCLUDE_DIRS})
        link_directories(${CAP_LIBRARY_DIRS})
    endif()
endif()

if(N2N_OPTION_MINIUPNP)
    pkg_check_modules(MINIUPNP miniupnpc)
    if(MINIUPNP_FOUND)
        set(HAVE_LIBMINIUPNPC 1)
        include_directories(${MINIUPNP_INCLUDE_DIRS})
        link_directories(${MINIUPNP_LIBRARY_DIRS})
    endif()
endif()

if(N2N_OPTION_NATPMP)
    pkg_check_modules(NATPMP libnatpmp)
    if(NATPMP_FOUND)
        set(HAVE_LIBNATPMP 1)
        include_directories(${NATPMP_INCLUDE_DIRS})
        link_directories(${NATPMP_LIBRARY_DIRS})
    endif()
endif()

if(N2N_OPTION_PTHREAD)
    set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
    find_package(Threads)
    if(Threads_FOUND)
        set(HAVE_LIBPTHREAD 1)
    endif()
endif()

# Check for headers
include(CheckIncludeFile)
check_include_file("netinet/in.h" HAVE_NETINET_IN_H)
check_include_file("arpa/inet.h" HAVE_ARPA_INET_H)
check_include_file("netdb.h" HAVE_NETDB_H)
check_include_file("sys/socket.h" HAVE_SYS_SOCKET_H)
check_include_file("sys/types.h" HAVE_SYS_TYPES_H)
check_include_file("unistd.h" HAVE_UNISTD_H)

# Configure config.h
configure_file(cmake/config.h.in include/config.h)
include_directories(${CMAKE_BINARY_DIR}/include) # For generated config.h
include_directories(include)

# Sources
set(N2N_SOURCES
    src/aes.c
    src/auth.c
    src/cc20.c
    src/curve25519.c
    src/edge_management.c
    src/edge_utils.c
    src/header_encryption.c
    src/hexdump.c
    src/json.c
    src/management.c
    src/minilzo.c
    src/n2n.c
    src/n2n_port_mapping.c
    src/n2n_regex.c
    src/network_traffic_filter.c
    src/pearson.c
    src/random_numbers.c
    src/sn_management.c
    src/sn_selection.c
    src/sn_utils.c
    src/speck.c
    src/tf.c
    src/transform_aes.c
    src/transform_cc20.c
    src/transform_lzo.c
    src/transform_null.c
    src/transform_speck.c
    src/transform_tf.c
    src/transform_zstd.c
    src/tuntap_freebsd.c
    src/tuntap_linux.c
    src/tuntap_netbsd.c
    src/tuntap_osx.c
    src/wire.c
)

if(WIN32)
    list(APPEND N2N_SOURCES
        src/win32/edge_utils_win32.c
        src/win32/getopt1.c
        src/win32/getopt.c
        src/win32/wintap.c
        src/win32/edge.rc
    )
    set(N2N_EXTRA_LIBS netapi32 ws2_32 iphlpapi)
elseif(SOLARIS)
    set(N2N_EXTRA_LIBS socket nsl)
endif()

# Library
add_library(n2n STATIC ${N2N_SOURCES})
target_link_libraries(n2n ${N2N_EXTRA_LIBS})

if(HAVE_LIBZSTD)
    target_link_libraries(n2n ${ZSTD_LIBRARIES})
endif()
if(HAVE_LIBCRYPTO)
    target_link_libraries(n2n OpenSSL::Crypto)
endif()
if(HAVE_LIBPCAP)
    target_link_libraries(n2n ${PCAP_LIBRARIES})
endif()
if(HAVE_LIBCAP)
    target_link_libraries(n2n ${CAP_LIBRARIES})
endif()
if(HAVE_LIBMINIUPNPC)
    target_link_libraries(n2n ${MINIUPNP_LIBRARIES})
endif()
if(HAVE_LIBNATPMP)
    target_link_libraries(n2n ${NATPMP_LIBRARIES})
endif()
if(HAVE_LIBPTHREAD)
    target_link_libraries(n2n Threads::Threads)
endif()

# Executables
add_executable(edge src/edge.c)
target_link_libraries(edge n2n)

add_executable(supernode src/supernode.c)
target_link_libraries(supernode n2n)

# Tools
add_executable(n2n-benchmark tools/n2n-benchmark.c)
target_link_libraries(n2n-benchmark n2n)

add_executable(n2n-keygen tools/n2n-keygen.c)
target_link_libraries(n2n-keygen n2n)

add_executable(n2n-route tools/n2n-route.c)
target_link_libraries(n2n-route n2n)

add_executable(n2n-portfwd tools/n2n-portfwd.c)
target_link_libraries(n2n-portfwd n2n)

add_executable(n2n-decode tools/n2n-decode.c)
target_link_libraries(n2n-decode n2n)

# Tests
add_executable(tests-compress tools/tests-compress.c)
target_link_libraries(tests-compress n2n)

add_executable(tests-elliptic tools/tests-elliptic.c)
target_link_libraries(tests-elliptic n2n)

add_executable(tests-hashing tools/tests-hashing.c)
target_link_libraries(tests-hashing n2n)

add_executable(tests-transform tools/tests-transform.c)
target_link_libraries(tests-transform n2n)

add_executable(tests-wire tools/tests-wire.c)
target_link_libraries(tests-wire n2n)

add_executable(tests-auth tools/tests-auth.c)
target_link_libraries(tests-auth n2n)

# Examples
add_executable(example_edge_embed_quick_edge_init examples/example_edge_embed_quick_edge_init.c)
target_link_libraries(example_edge_embed_quick_edge_init n2n)

add_executable(example_edge_embed examples/example_edge_embed.c)
target_link_libraries(example_edge_embed n2n)

add_executable(example_sn_embed examples/example_sn_embed.c)
target_link_libraries(example_sn_embed n2n)

# Install
install(TARGETS edge supernode RUNTIME DESTINATION sbin)
install(FILES doc/edge.8 DESTINATION share/man/man8)
install(FILES doc/supernode.1 DESTINATION share/man/man1)
install(FILES doc/n2n.7 DESTINATION share/man/man7)
