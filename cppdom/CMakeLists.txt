# CMake-based build system for cppdom
# 2009-2011 Ryan Pavlik <rpavlik@iastate.edu>
# http://academic.cleardefinition.com/
# Iowa State University HCI Graduate Program/VRAC
set(CPPDOM_VERSION_MAJOR "${CPACK_PACKAGE_VERSION_MAJOR}")
set(CPPDOM_VERSION_MINOR "${CPACK_PACKAGE_VERSION_MINOR}")
set(CPPDOM_VERSION_PATCH "${CPACK_PACKAGE_VERSION_PATCH}")
configure_file(version.h.in "${CMAKE_CURRENT_SOURCE_DIR}/version.h" @ONLY)

set(API
    config.h
    cppdom.h
    predicates.h
    shared_ptr.h
    xmlparser.h
    xmltokenizer.h
    version.h)
set(EXT_API
    ext/OptionRepository.h)
set(SOURCES
    cppdom.cpp
    xmlparser.cpp
    xmltokenizer.cpp
    ext/OptionRepository.cpp)

if(BOOST_FOUND)
    include_directories(${Boost_INCLUDE_DIRS})
    list(APPEND SOURCES SpiritParser.cpp)
    list(APPEND API SpiritParser.h)
endif()

add_definitions(-DCPPDOM_EXPORTS)

set(CMAKE_RELEASE_POSTFIX "-${VERSION_UNDERSCORES}")
set(CMAKE_RELWITHDEBINFO_POSTFIX "-${VERSION_UNDERSCORES}")
set(CMAKE_MINSIZEREL_POSTFIX "-${VERSION_UNDERSCORES}")
set(CMAKE_DEBUG_POSTFIX  "_d-${VERSION_UNDERSCORES}")

add_library(cppdom SHARED ${API} ${EXT_API} ${SOURCES})

set_target_properties(cppdom PROPERTIES
                      PUBLIC_HEADER "${API}"
                      COMPILE_DEFINITIONS_DEBUG "CPPDOM_DEBUG")
install(TARGETS cppdom
        RUNTIME DESTINATION ${BIN_DIR} COMPONENT Runtime
        LIBRARY DESTINATION    ${LIB_DIR} COMPONENT Runtime
        ARCHIVE DESTINATION    ${ARCH_DIR} COMPONENT Development
        PUBLIC_HEADER DESTINATION ${INCLUDE_DIR}/cppdom COMPONENT Development)
install(FILES ${EXT_API}
        DESTINATION ${INCLUDE_DIR}/cppdom/ext
        COMPONENT Development)
