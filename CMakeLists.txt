cmake_minimum_required(VERSION 3.10)

if (NOT DEFINED ENV{TRAVIS_BUILD_NUMBER})
    set (ENV{TRAVIS_BUILD_NUMBER} "42")
endif()

project(helloworld VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})

find_package(GTest REQUIRED)

configure_file(version.h.in ${PROJECT_SOURCE_DIR}/version.h)

add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(gtest_version gtest_version.cpp)

set_target_properties(helloworld_cli helloworld gtest_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)

set_target_properties(gtest_version PROPERTIES
    INCLUDE_DIRECTORIES ${GTEST_INCLUDE_DIRS}
)

target_link_libraries(helloworld_cli  PRIVATE
    helloworld
)


target_link_libraries(gtest_version
    ${GTEST_LIBRARIES}
    helloworld
)


if (MSVC)
    target_compile_options(helloworld_cli PRIVATE
        /W4
    )
    target_compile_options(helloworld PRIVATE
        /W4
    )
    target_compile_options(gtest_version PRIVATE
        /W4
    )
else ()
    target_compile_options(helloworld_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(gtest_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()


install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT burlunder@gmail.com)

set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)

include(CPack)

enable_testing()


gtest_discover_tests(gtest_version
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
)
