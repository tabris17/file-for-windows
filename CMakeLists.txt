cmake_minimum_required(VERSION 3.30)

project(file)

file(READ "${CMAKE_SOURCE_DIR}/VERSION" FILE_VERSION)

set(PATCH_DIR "${CMAKE_SOURCE_DIR}/patch")
set(OUTPUT_DIR "${CMAKE_SOURCE_DIR}/out")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OUTPUT_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_DIR})
set(LIBMAGIC_SOURCE_FILES
  file/src/apprentice.c
  file/src/apptype.c
  file/src/ascmagic.c
  file/src/asctime_r.c
  file/src/asprintf.c
  file/src/buffer.c
  file/src/cdf_time.c
  file/src/cdf.c
  file/src/compress.c
  file/src/ctime_r.c
  file/src/der.c
  file/src/dprintf.c
  file/src/encoding.c
  file/src/fmtcheck.c
  file/src/fsmagic.c
  file/src/funcs.c
  file/src/getline.c
  file/src/getopt_long.c
  file/src/gmtime_r.c
  file/src/is_csv.c
  file/src/is_json.c
  file/src/is_simh.c
  file/src/is_tar.c
  file/src/localtime_r.c
  file/src/magic.c
  file/src/pread.c
  file/src/print.c
  file/src/readcdf.c
  file/src/readelf.c
  file/src/seccomp.c
  file/src/softmagic.c
  file/src/strcasestr.c
  file/src/strlcat.c
  file/src/strlcpy.c
  file/src/vasprintf.c
  include/magic.def
)
set(BZIP2_SOURCE_FILES
  bzip2/blocksort.c
  bzip2/bzlib.c
  bzip2/compress.c
  bzip2/crctable.c
  bzip2/decompress.c
  bzip2/huffman.c
  bzip2/randtable.c
)
set(LIBMAGIC_LIBS shlwapi pcre2-posix zlibstatic bzip2 liblzma)
set(LIBMAGIC_DEFINITIONS -DHAVE_CONFIG_H -DVERSION="${FILE_VERSION}")

# Replace the version in the magic.h.in and write it to magic.h
file(READ file/src/magic.h.in MAGIC_H_CONTENT)
string(REPLACE "." "" FILE_VERSION_WITHOUT_DOT "${FILE_VERSION}")
string(REPLACE "X.YY" ${FILE_VERSION_WITHOUT_DOT} MAGIC_H_CONTENT_NEW "${MAGIC_H_CONTENT}")
file(WRITE file/src/magic.h "${MAGIC_H_CONTENT_NEW}")

add_definitions(-DPCRE2_STATIC -DLZMA_API_STATIC)
include_directories("${CMAKE_SOURCE_DIR}/include" pcre2/src file/src zlib "${CMAKE_BINARY_DIR}/zlib" bzip2 "${CMAKE_BINARY_DIR}/pcre2" xz/src/liblzma/api)

add_library(bzip2 STATIC ${BZIP2_SOURCE_FILES})
target_include_directories(bzip2 PRIVATE bzip2)

set(ZLIB_BUILD_EXAMPLES OFF CACHE BOOL "Build zlib examples" FORCE)
add_subdirectory(zlib)

set(BUILD_SHARED_LIBS OFF)
add_subdirectory(xz)

set(PCRE2_BUILD_PCRE2GREP OFF CACHE BOOL "Build pcre2grep" FORCE)
set(PCRE2_BUILD_TESTS OFF CACHE BOOL "Build pcre2grep tests" FORCE)
add_subdirectory(pcre2)

add_library(magicstatic STATIC ${LIBMAGIC_SOURCE_FILES})
target_compile_definitions(magicstatic PUBLIC ${LIBMAGIC_DEFINITIONS})

add_library(magic SHARED ${LIBMAGIC_SOURCE_FILES})
target_link_libraries(magic ${LIBMAGIC_LIBS})
target_compile_definitions(magic PUBLIC ${LIBMAGIC_DEFINITIONS})

add_executable(file file/src/file.c)
target_include_directories(file PRIVATE ${CMAKE_SOURCE_DIR})
target_link_libraries(file magicstatic ${LIBMAGIC_LIBS})
target_compile_definitions(file PUBLIC ${LIBMAGIC_DEFINITIONS})

if(NOT EXISTS ${OUTPUT_DIR})
  file(MAKE_DIRECTORY ${OUTPUT_DIR})  
endif()

set(MAGIC_DIR file/magic/Magdir)
file(GLOB MAGIC_FRAGMENTS "${MAGIC_DIR}/*")
file(WRITE "${OUTPUT_DIR}/magic" "")

foreach(MAGIC_FRAGMENT ${MAGIC_FRAGMENTS})
  file(READ ${MAGIC_FRAGMENT} MAGIC_FRAGMENT_CONTENTS)
  file(APPEND "${OUTPUT_DIR}/magic" "${MAGIC_FRAGMENT_CONTENTS}\n")
endforeach()

add_custom_command(
  OUTPUT "${OUTPUT_DIR}/magic.mgc"
  COMMAND cd "${OUTPUT_DIR}"
  COMMAND file -C -m magic
  DEPENDS "${OUTPUT_DIR}/file"
  COMMENT "Compiling magic file"
)
add_custom_target(magic_file ALL DEPENDS "${OUTPUT_DIR}/magic.mgc")

execute_process(
  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/file"
  COMMAND git apply "${PATCH_DIR}/cdf_h.patch"
)
