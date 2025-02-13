cmake_minimum_required(VERSION 3.30)

project(file)

file(READ "${CMAKE_SOURCE_DIR}/VERSION" FILE_VERSION)

set(PATCH_DIR "${CMAKE_SOURCE_DIR}/patch")
set(OUTPUT_DIR "${CMAKE_SOURCE_DIR}/out")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${OUTPUT_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${OUTPUT_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${OUTPUT_DIR}")
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
set(LIBMAGIC_LIBS pcre2-posix shlwapi)

#if(CMAKE_COMPILER_IS_GNUCC)
#endif(CMAKE_COMPILER_IS_GNUCC)

add_definitions(-DHAVE_CONFIG_H -DVERSION="${FILE_VERSION}" -DPCRE2_STATIC)
include_directories("${CMAKE_SOURCE_DIR}/include" pcre2/src file/src)
add_subdirectory(pcre2)

add_library(magicstatic STATIC ${LIBMAGIC_SOURCE_FILES})
target_include_directories(magicstatic PRIVATE "${CMAKE_SOURCE_DIR}")
target_compile_definitions(magicstatic PRIVATE -DMAGIC_RESOURCE)
target_include_directories(magicstatic PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include" pcre2/src file/src)
target_compile_definitions(magicstatic PUBLIC -DHAVE_CONFIG_H -DVERSION="${FILE_VERSION}")

add_library(magic SHARED ${LIBMAGIC_SOURCE_FILES})
target_link_libraries(magic ${LIBMAGIC_LIBS})

add_executable(file file/src/file.c)
target_include_directories(file PRIVATE "${CMAKE_SOURCE_DIR}")
target_link_libraries(file magicstatic ${LIBMAGIC_LIBS})

if(NOT EXISTS "${OUTPUT_DIR}")
  file(MAKE_DIRECTORY "${OUTPUT_DIR}")  
endif()

set(MAGIC_DIR file/magic/Magdir)
file(GLOB MAGIC_FRAGMENTS "${MAGIC_DIR}/*")
file(WRITE "${OUTPUT_DIR}/magic" "")

foreach(MAGIC_FRAGMENT ${MAGIC_FRAGMENTS})
  file(READ "${MAGIC_FRAGMENT}" MAGIC_FRAGMENT_CONTENTS)
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
