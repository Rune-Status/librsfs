cmake_minimum_required(VERSION 3.0.2)
project(librsfs)
include(GenerateExportHeader)

# Boost configuration
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_SHARED_LINKER_FLAGS "-fPIC")

# Please, dear CMake gods, find ZLib
find_package(LibLZMA)
include_directories(${LIBLZMA_INCLUDE_DIRS})

# Make the C compiler know we have LZMA
if(LIBLZMA_FOUND)
  add_definitions(-DLZMA_FOUND)
endif()

# C++ flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_FILE_OFFSET_BITS=64 -std=c++11 -fPIC -static-libgcc -static-libstdc++")

# Our list of source files to include in this build
set(SOURCE_FILES src/main.cc src/file_system.cc src/file_system.h src/capi.h src/librsfs.h src/capi.cc src/compression.cc src/bzip2/bzlib.c src/bzip2/compress.c src/bzip2/decompress.c src/bzip2/crctable.c src/bzip2/huffman.c src/bzip2/randtable.c src/bzip2/blocksort.c)
set(ZLIB_SOURCES src/zlib/adler32.c src/zlib/compress.c src/zlib/crc32.c src/zlib/deflate.c src/zlib/gzclose.c src/zlib/gzlib.c src/zlib/gzread.c src/zlib/gzwrite.c src/zlib/inflate.c src/zlib/inffast.c src/zlib/inftrees.c src/zlib/uncompr.c src/zlib/trees.c src/zlib/zutil.c)

add_executable(librsfs ${SOURCE_FILES} ${ZLIB_SOURCES} src/index.cc src/index.h src/folder_info.cc src/folder_info.h src/compression.cc src/compression.h) # Define executable for command-line usage
add_library(rsfs SHARED ${SOURCE_FILES} ${ZLIB_SOURCES} src/index.cc src/index.h src/folder_info.cc src/folder_info.h src/compression.cc src/compression.h) # Define the shared library (.dll or .dylib or .a)

# Define how the exported library will come out
GENERATE_EXPORT_HEADER(rsfs
                       BASE_NAME librsfs # Reference name to avoid conflicts with the executable
                       EXPORT_MACRO_NAME RSFS_EXPORT # The exported compiler directive that's set if we build the library
                       EXPORT_FILE_NAME ${CMAKE_CURRENT_SOURCE_DIR}/src/librsfs.h # Output .h file containing library defines
                       STATIC_DEFINE RSFS_BUILT_AS_STATIC)

# If it is built as static then define that as cflag.
set_target_properties(librsfs PROPERTIES
                      COMPILE_FLAGS -DRSFS_BUILT_AS_STATIC)

# Define dependencies of our build
target_link_libraries(librsfs ${LIBLZMA_LIBRARIES})
target_link_libraries(rsfs ${LIBLZMA_LIBRARIES})
