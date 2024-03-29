cmake_minimum_required(VERSION 3.15.2)

project(cppcheckOnModernJson)

find_program(
  CONAN
  NAMES conan.exe
  HINTS "C:\\Program Files\\Conan\\conan")

if(${CONAN}-NOTFOUND)
  message(FATAL_ERROR "Conan not found")
endif(${CONAN}-NOTFOUND)

execute_process(
  COMMAND
    ${CONAN} install ${CMAKE_CURRENT_LIST_DIR} -s arch=x86 -s build_type=Debug
    -s compiler.runtime=MTd --build=missing
  COMMAND
    ${CONAN} install ${CMAKE_CURRENT_LIST_DIR} -s arch=x86 -s build_type=Release
    -s compiler.runtime=MT --build=missing
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

include(${CMAKE_BINARY_DIR}/conanbuildinfo_multi.cmake)
conan_basic_setup(TARGETS)

add_executable(Conan_Jsonformoderncpp ${CMAKE_CURRENT_LIST_DIR}/src/main)
set_source_files_properties(Conan_Jsonformoderncpp PROPERTIES CXX_STANDARD 17)
target_link_libraries(Conan_Jsonformoderncpp
                      PRIVATE CONAN_PKG::jsonformoderncpp)

# ==============================================================================

include(ExternalProject)
ExternalProject_Add(
  OfficialLastReleaseRepository
  GIT_REPOSITORY https://github.com/nlohmann/json.git
  GIT_TAG v3.7.3
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
  TEST_COMMAND "")

add_executable(OfficialLastRelease_Jsonformoderncpp
               ${CMAKE_CURRENT_LIST_DIR}/src/main)
set_source_files_properties(OfficialLastRelease_Jsonformoderncpp PROPERTIES
                            CXX_STANDARD 17)

add_dependencies(OfficialLastRelease_Jsonformoderncpp
                 OfficialLastReleaseRepository)

ExternalProject_Get_Property(OfficialLastReleaseRepository SOURCE_DIR)
target_include_directories(OfficialLastRelease_Jsonformoderncpp
                           PRIVATE ${SOURCE_DIR}/include)

# ==============================================================================

include(ExternalProject)
ExternalProject_Add(
  PersonalForkRepository
  GIT_REPOSITORY https://github.com/Xav83/json.git
  GIT_TAG origin/removeMoreCppcheckWarnings
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
  TEST_COMMAND "")

add_executable(PersonalFork_Jsonformoderncpp ${CMAKE_CURRENT_LIST_DIR}/src/main)
set_source_files_properties(PersonalFork_Jsonformoderncpp PROPERTIES
                            CXX_STANDARD 17)

add_dependencies(PersonalFork_Jsonformoderncpp PersonalForkRepository)

ExternalProject_Get_Property(PersonalForkRepository SOURCE_DIR)
target_include_directories(PersonalFork_Jsonformoderncpp
                           PRIVATE ${SOURCE_DIR}/include)

# ==============================================================================

find_program(
  CPPCHECK
  NAMES cppcheck cppcheck.exe
  HINTS "C:\\Program Files\\Cppcheck")

if(${CPPCHECK} MATCHES CPPCHECK-NOTFOUND)
  message(FATAL_ERROR "Cppcheck not found")
endif(${CPPCHECK} MATCHES CPPCHECK-NOTFOUND)

enable_testing()

add_test(
  NAME cppcheck_OfficialLastRelease_Jsonformoderncpp
  COMMAND ${CPPCHECK} --error-exitcode=1
          --enable=warning,style,performance,portability --std=c++11
          --project=OfficialLastRelease_Jsonformoderncpp.vcxproj
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_test(
  NAME cppcheck_Conan_Jsonformoderncpp
  COMMAND ${CPPCHECK} --error-exitcode=1
          --enable=warning,style,performance,portability --std=c++11
          --project=Conan_Jsonformoderncpp.vcxproj
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_test(
NAME cppcheck_PersonalForkRepository
COMMAND ${CPPCHECK} --error-exitcode=1
        --enable=warning,style,performance,portability --std=c++11
        --project=PersonalFork_Jsonformoderncpp.vcxproj
WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
