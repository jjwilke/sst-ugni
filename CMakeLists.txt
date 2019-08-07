cmake_minimum_required (VERSION 3.12)

project (sstUgni)
# The version number.
set (SST_uGNI_VERSION_MAJOR 1)
set (SST_uGNI_VERSION_MINOR 0)

enable_testing()

set(SSTMAC_DIR "" CACHE PATH "The path to SST/macro installation")
find_path(SSTMAC_INC sstmac/sstmacro.h PATHS ${SSTMAC_DIR}/include)
find_library(SSTMAC_LIB sstmac ${SSTMAC_DIR}/lib)
find_library(SPKT_LIB sprockit ${SSTMAC_DIR}/lib)
find_program(SSTMAC_EXE sstmac ${SSTMAC_DIR}/bin)

configure_file(
 ${CMAKE_CURRENT_SOURCE_DIR}/cray-gni-headers.pc.in
 ${CMAKE_CURRENT_BINARY_DIR}/cray-gni-headers.pc @ONLY
)

configure_file(
 ${CMAKE_CURRENT_SOURCE_DIR}/env.sh.in
 ${CMAKE_CURRENT_BINARY_DIR}/env.sh @ONLY
)

add_library(ugni SHARED libugni.cc)
#make sure have C++11 for SST
target_compile_features(ugni PUBLIC cxx_std_11)
target_link_libraries(ugni
  PUBLIC ${SSTMAC_LIB} ${SPKT_LIB})
target_include_directories(ugni
  PUBLIC ${SSTMAC_INC})

add_library(pmi SHARED libpmi.cc)
#make sure have C++11 for SST
target_compile_features(pmi PUBLIC cxx_std_11)
target_link_libraries(pmi
  PUBLIC ${SSTMAC_LIB} ${SPKT_LIB})
target_include_directories(pmi
  PUBLIC ${SSTMAC_INC})

add_library(ibverbs SHARED libibverbs.c)
#make sure have C++11 for SST
target_compile_features(ibverbs PUBLIC cxx_std_11)
target_link_libraries(ibverbs
  PUBLIC ${SSTMAC_LIB} ${SPKT_LIB})
target_include_directories(ibverbs
  PUBLIC ${SSTMAC_INC})

INSTALL(
  TARGETS ugni pmi ibverbs
  EXPORT sstUgni
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

configure_file(
 ${CMAKE_CURRENT_SOURCE_DIR}/cray-ugni.pc.in
 ${CMAKE_CURRENT_BINARY_DIR}/cray-ugni.pc @ONLY
)

include(CMakePackageConfigHelpers)
configure_package_config_file(cmake/sstUgniConfig.cmake.in
                              ${CMAKE_CURRENT_BINARY_DIR}/sstUgniConfig.cmake 
                              INSTALL_DESTINATION cmake)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/sstUgniConfigVersion.cmake
                                 VERSION "${HCLIB_VERSION_MAJOR}.${HCLIB_VERSION_MINOR}"
                                 COMPATIBILITY SameMajorVersion)

install(
  FILES 
  ${CMAKE_CURRENT_BINARY_DIR}/sstUgniConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/sstUgniConfigVersion.cmake
  DESTINATION cmake 
)

FILE(GLOB HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
install(
  FILES 
  ${HEADERS}
  DESTINATION include
)

install(
  FILES
  ${CMAKE_CURRENT_BINARY_DIR}/cray-ugni.pc
  ${CMAKE_CURRENT_BINARY_DIR}/env.sh
  ${CMAKE_CURRENT_BINARY_DIR}/cray-gni-headers.pc
  DESTINATION lib/pkgconfig
)

