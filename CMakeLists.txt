cmake_minimum_required(VERSION 3.2)

set(HEADER_FILE "build_number.h")
set(CACHE_FILE "BuildNumberCache.txt")

#Reading data from file + incrementation
IF(EXISTS ${CACHE_FILE})
    file(READ ${CACHE_FILE} BUILD_NUMBER)
    math(EXPR BUILD_NUMBER "${BUILD_NUMBER}+1")
ELSE()
    set(BUILD_NUMBER "1")
ENDIF()

#Update the cache
file(WRITE ${CACHE_FILE} "${BUILD_NUMBER}")

#Create the header
file(WRITE ${HEADER_FILE} "#ifndef BUILD_NUMBER_H\n#define BUILD_NUMBER_H\n\n#define BUILD_NUMBER ${BUILD_NUMBER}\n\n#endif")

project(helloword VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

configure_file(version.h.in version.h)

add_executable(helloword_cli main.cpp)
add_library(helloword lib.cpp)
add_executable(test_version test_version.cpp)



set_target_properties(helloword_cli helloword test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)


target_include_directories(helloword
    PRIVATE "${CMAKE_BINARY_DIR}"
)

set_target_properties(test_version PROPERTIES
   COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

target_link_libraries(helloword_cli PRIVATE
    helloword
)
target_link_libraries(test_version
    ${Boost_LIBRARIES}
    helloword
)

if (MSVC)
    target_compile_options(helloword_cli PRIVATE
        /W4
    )
    target_compile_options(helloword PRIVATE
        /W4
    )
    target_compile_options(test_version PRIVATE
        /W4
    )
else ()
    target_compile_options(helloword_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloword PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()


install(TARGETS helloword_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT example@example.com)

include(CPack)

enable_testing()

add_test(test_version test_version)
