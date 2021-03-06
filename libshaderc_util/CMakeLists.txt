project(libshaderc_util)

add_library(shaderc_util STATIC
  include/libshaderc_util/counting_includer.h
  include/libshaderc_util/file_finder.h
  include/libshaderc_util/format.h
  include/libshaderc_util/io.h
  include/libshaderc_util/mutex.h
  include/libshaderc_util/message.h
  include/libshaderc_util/resources.h
  include/libshaderc_util/string_piece.h
  include/libshaderc_util/universal_unistd.h
  include/libshaderc_util/version_profile.h
  src/compiler.cc
  src/file_finder.cc
  src/io.cc
  src/message.cc
  src/resources.cc
  src/shader_stage.cc
  src/version_profile.cc
)

shaderc_default_compile_options(shaderc_util)
target_include_directories(shaderc_util
  PUBLIC include PRIVATE ${glslang_SOURCE_DIR})
find_package(Threads)
target_link_libraries(shaderc_util PRIVATE
  glslang OSDependent OGLCompiler glslang SPIRV
  SPIRV-Tools ${CMAKE_THREAD_LIBS_INIT})

shaderc_add_tests(
  TEST_PREFIX shaderc_util
  LINK_LIBS shaderc_util
  TEST_NAMES
    counting_includer
    string_piece
    format
    file_finder
    io
    message
    mutex)

if(${SHADERC_ENABLE_TESTS})
  target_include_directories(shaderc_util_counting_includer_test
    PRIVATE ${glslang_SOURCE_DIR})
endif()

shaderc_add_tests(
  TEST_PREFIX shaderc_util
  LINK_LIBS shaderc_util
  INCLUDE_DIRS ${glslang_SOURCE_DIR}
  TEST_NAMES
    compiler)

# This target copies content of testdata into the build directory.
add_custom_target(testdata COMMAND
  ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_SOURCE_DIR}/testdata/copy-to-build.cmake)

if(${SHADERC_ENABLE_TESTS})
  add_dependencies(shaderc_util_file_finder_test testdata)
  add_dependencies(shaderc_util_io_test testdata)
endif()

add_definitions(-DCURRENT_DIR="${CMAKE_CURRENT_BINARY_DIR}")
