cmake_minimum_required (VERSION 3.11)

project (validator-keys-tool)
#[===========================================[
  The tool depends on the xrpl_core
  library which is defined by the rippled
  project. This looks for installed rippled
  libs and, if not found, pulls them in with
  FetchContent.
#]===========================================]
get_directory_property(has_parent PARENT_DIRECTORY)
if (NOT has_parent)
  project (validator-keys-tool)
  find_package(Blocmatrix QUIET)
  if (NOT TARGET Blocmatrix::bmcl_core)
    find_package(Git)

    if (NOT GIT_FOUND)
      message (FATAL_ERROR "git is required to determine branch name")
    endif ()

    execute_process (COMMAND ${GIT_EXECUTABLE} "rev-parse" "--abbrev-ref" "HEAD"
                     WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                     RESULT_VARIABLE _git_exit_code
                     OUTPUT_VARIABLE _branch
                     OUTPUT_STRIP_TRAILING_WHITESPACE
                     ERROR_QUIET)
    if (NOT _git_exit_code EQUAL 0)
      message (WARNING "git command failed - deduced branch might be incorrect")
    endif ()

    # blocmatrixd_tag is cache string and can be overriden when configuring
    # with -Dblocmatrixd_tag=commit_or_tag in order to pick a specific
    # blocmatrixd version to download. Default tag is develop/master/release as 
    # determined by the branch of this project
    if (NOT (_branch STREQUAL "master" OR _branch STREQUAL "release"))
      set (blocmatrixd_tag "develop" CACHE STRING
        "tag/commit of blocmatrixd to fetch from if a local install is not found")
    else ()
      set (blocmatrixd_tag "${_branch}" CACHE STRING
        "tag/commit of blocmatrixd to fetch from if a local install is not found")
    endif ()
    message(STATUS "Installed blocmatrixd not found... \
      using local copy from tag/commit [${blocmatrixd_tag}]")
    include (FetchContent)
    FetchContent_Declare(
      blocmatrixd_src
      GIT_REPOSITORY https://github.com/ReddyArunreddy/blocmatrixd.git
      GIT_TAG ${blocmatrixd_tag}
    )
    FetchContent_GetProperties(blocmatrixd_src)
    if(NOT blocmatrixd_src_POPULATED)
      message (STATUS "Pausing to download blocmatrixd source...")
      FetchContent_Populate(blocmatrixd_src)
    endif()
    add_subdirectory(${blocmatrixd_src_SOURCE_DIR} xrpl_core)
  endif ()
endif ()

add_executable (validator-keys
  src/ValidatorKeys.cpp
  src/ValidatorKeysTool.cpp
  ## UNIT TESTS:
  src/test/ValidatorKeys_test.cpp
  src/test/ValidatorKeysTool_test.cpp)
target_include_directories (validator-keys PRIVATE src)
target_link_libraries (validator-keys Blocmatrix::bmcl_core)

if (has_parent)
  set_target_properties (validator-keys PROPERTIES EXCLUDE_FROM_ALL ON)
  set_target_properties (validator-keys PROPERTIES EXCLUDE_FROM_DEFAULT_BUILD ON)
endif ()



