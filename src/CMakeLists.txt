list(APPEND tetgen_src_sources ${CMAKE_CURRENT_SOURCE_DIR}/predicates.cxx)
list(APPEND tetgen_src_sources ${CMAKE_CURRENT_SOURCE_DIR}/tetgen.cxx)

list(APPEND tetgen_src_headers ${CMAKE_CURRENT_SOURCE_DIR}/tetgen.h)

list(APPEND tetgen_src_includes ${CMAKE_CURRENT_SOURCE_DIR})

set(tetgen_src_sources ${tetgen_src_sources} PARENT_SCOPE)
set(tetgen_src_headers ${tetgen_src_headers} PARENT_SCOPE)
set(tetgen_src_includes ${tetgen_src_includes} PARENT_SCOPE)
