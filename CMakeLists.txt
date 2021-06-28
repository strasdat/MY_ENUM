cmake_minimum_required(VERSION 3.12)

project(my_enum VERSION 1.0
                DESCRIPTION "Small c++ macro library to add compile-time introspection to c++ enum classes."
                LANGUAGES CXX)

include(GNUInstallDirs)

option(USE_FMT_STRING_VIEW "Use string_view from fmt library" OFF)

if(USE_FMT_STRING_VIEW)
  set (CMAKE_CXX_STANDARD 14)
  find_package(fmt 5.2 REQUIRED)
  set (STRING_VIEW_LIB fmt::fmt)
else()
  set (CMAKE_CXX_STANDARD 17)
  add_definitions(-DMY_ENUM_USE_STD_STRING_VIEW)
  set (STRING_VIEW_LIB)
endif()

add_library(${PROJECT_NAME} INTERFACE)

set(MY_ENUM_HEADER_FILES
    my_enum/Enum.h
    my_enum/EnumO.h
    my_enum/EnumFlags.h
    my_enum/EnumFlagsO.h)


# set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER my_enum/)

# needed for boost preprocessor only
find_package(Boost 1.62 REQUIRED)


# set(MY_ENUM_HEADER_IMPL_FILES
#     my_enum/impl/EnumDetails.h
#     my_enum/impl/EnumFlagsDetails.h
#     my_enum/impl/CompilerAttributes.h
#     my_enum/impl/StringView.h)

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    include(CTest)
endif()

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    add_subdirectory(tests)
endif()

option(BUILD_EXAMPLES "Build examples." ON)
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()


# Following http://dominikberner.ch/cmake-interface-lib/

target_include_directories(
  ${PROJECT_NAME}
  INTERFACE $<BUILD_INTERFACE:${${PROJECT_NAME}_SOURCE_DIR}/>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_17)

install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}_Targets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

include(CMakePackageConfigHelpers)
write_basic_package_version_file("${PROJECT_NAME}ConfigVersion.cmake"
                                  VERSION ${PROJECT_VERSION}
                                  COMPATIBILITY SameMajorVersion)

configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
  "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  INSTALL_DESTINATION
  ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

install(EXPORT ${PROJECT_NAME}_Targets
  FILE ${PROJECT_NAME}Targets.cmake
  NAMESPACE ${PROJECT_NAME}::
  DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
              "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

install(DIRECTORY ${PROJECT_SOURCE_DIR}/my_enum DESTINATION include)
