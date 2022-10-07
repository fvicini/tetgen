cmake_minimum_required(VERSION 3.9)

project(tetgen LANGUAGES CXX C VERSION 1.0.0)

# Create library options
###############################################################################################
option(BUILD_EXECUTABLE "Build tetgen executable" OFF)
option(BUILD_LIBRARY    "Build libtetgen library" ON)

# Create library variables
###############################################################################################
set(TETGEN_LIBRARY_SOURCES ${PROJECT_NAME}_sources)
set(TETGEN_LIBRARY_HEADERS ${PROJECT_NAME}_headers)
set(TETGEN_LIBRARY_PUBLIC_HEADERS ${PROJECT_NAME}_public_headers)
set(TETGEN_LIBRARY_LINKED_LIBRARIES ${PROJECT_NAME}_linked_libraries)
set(TETGEN_LIBRARY_INCLUDE ${PROJECT_NAME}_include)
set(TETGEN_LIBRARY_ADDITIONAL_INCLUDE ${PROJECT_NAME}_additional_include)
set(TETGEN_INSTALL_DIR ${PROJECT_NAME})
set(TETGEN_INSTALL_BINARY_DIR bin/)
set(TETGEN_INSTALL_INCLUDE_DIR include/)
set(TETGEN_INSTALL_LIB_DIR lib/)
set(TETGEN_INSTALL_ARCHIVE_DIR lib/)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()
Message(STATUS "Tetgen Build configuration: " ${CMAKE_BUILD_TYPE})

# Add dependencies
###########################################################################################
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Insert Sources
###############################################################################################
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)
list(APPEND ${TETGEN_LIBRARY_SOURCES} ${tetgen_src_sources})
list(APPEND ${TETGEN_LIBRARY_HEADERS} ${tetgen_src_headers})
list(APPEND ${TETGEN_LIBRARY_INCLUDE} ${tetgen_src_includes})

# Create public headers
###############################################################################################
set(TETGEN_LIBRARY_ADDITIONAL_INCLUDE_PATH ${CMAKE_CURRENT_BINARY_DIR}/${TETGEN_LIBRARY_ADDITIONAL_INCLUDE})

foreach(header ${${TETGEN_LIBRARY_HEADERS}})
        get_filename_component(headerDirectory ${header} DIRECTORY)
        get_filename_component(headerFileName ${header} NAME_WE)
        get_filename_component(headerExtension ${header} EXT)

  if(${headerExtension} MATCHES ".hpp.in")
    set(headerExtension ".hpp")
    set(headerDirectory ${TETGEN_LIBRARY_ADDITIONAL_INCLUDE_PATH})

    configure_file(${header} ${headerDirectory}/${headerFileName}${headerExtension} @ONLY)
  elseif(NOT ${headerExtension} MATCHES ".hpp" AND NOT ${headerExtension} MATCHES ".h")
    message(SEND_ERROR "Header extension not recognized for file ${headerFileName}${headerExtension}")
  endif()

  list(APPEND ${TETGEN_LIBRARY_PUBLIC_HEADERS} ${headerDirectory}/${headerFileName}${headerExtension})
endforeach()

list(APPEND ${TETGEN_LIBRARY_INCLUDE} ${TETGEN_LIBRARY_ADDITIONAL_INCLUDE_PATH})

# Create library
###############################################################################################
add_library(${PROJECT_NAME} STATIC ${${TETGEN_LIBRARY_SOURCES}} ${${TETGEN_LIBRARY_HEADERS}})
add_library(tetgen::tetgen ALIAS ${PROJECT_NAME})

set_property(SOURCE ${${TETGEN_LIBRARY_SOURCES}} PROPERTY LANGUAGE C )
set_target_properties(${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_link_libraries(${PROJECT_NAME} ${${TETGEN_LIBRARY_LINKED_LIBRARIES}})

target_include_directories(${PROJECT_NAME} PRIVATE ${${TETGEN_LIBRARY_INCLUDE}})
target_include_directories(${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:${TETGEN_INSTALL_INCLUDE_DIR}>)

target_compile_definitions(${PROJECT_NAME} PRIVATE -DTETLIBRARY)
target_compile_options(${PROJECT_NAME} PUBLIC -fPIC)

# Create install target
###############################################################################################
set_target_properties(${PROJECT_NAME} PROPERTIES 
PUBLIC_HEADER "${${TETGEN_LIBRARY_PUBLIC_HEADERS}}" 
EXPORT_NAME tetgen)

install(
	TARGETS ${PROJECT_NAME} 
	EXPORT tetgenTargets
    RUNTIME DESTINATION ${TETGEN_INSTALL_BINARY_DIR} # runtime executable
    LIBRARY DESTINATION ${TETGEN_INSTALL_LIB_DIR} # dynamic libraries
    ARCHIVE DESTINATION ${TETGEN_INSTALL_ARCHIVE_DIR} # static libraries
    PUBLIC_HEADER DESTINATION ${TETGEN_INSTALL_INCLUDE_DIR} # headers
)


include(CMakePackageConfigHelpers)

configure_package_config_file(
	${CMAKE_CURRENT_SOURCE_DIR}/cmake/tetgenConfig.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/tetgenConfig.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/${TETGEN_INSTALL_LIB_DIR}/cmake/${TETGEN_INSTALL_DIR}
	PATH_VARS TETGEN_INSTALL_INCLUDE_DIR
  )

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/tetgenConfigVersion.cmake
	VERSION ${tetgen_VERSION}
	COMPATIBILITY AnyNewerVersion
)

export(
	TARGETS ${PROJECT_NAME} 
	NAMESPACE tetgen::
	FILE ${CMAKE_CURRENT_BINARY_DIR}/tetgenTargets.cmake
)

install(
	EXPORT tetgenTargets 
	NAMESPACE tetgen:: 
	DESTINATION ${CMAKE_INSTALL_PREFIX}/${TETGEN_INSTALL_LIB_DIR}/cmake/${TETGEN_INSTALL_DIR}
)

install(
	FILES
		${CMAKE_CURRENT_BINARY_DIR}/tetgenConfig.cmake
		${CMAKE_CURRENT_BINARY_DIR}/tetgenConfigVersion.cmake
	DESTINATION
		${CMAKE_INSTALL_PREFIX}/${TETGEN_INSTALL_LIB_DIR}/cmake/${TETGEN_INSTALL_DIR}
)
