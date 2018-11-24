cmake_minimum_required(VERSION 3.2)

if (DEFINED ENV{TRAVIS_BUILD_NUMBER})
    project(otus-async VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})
else()
    project(otus-async VERSION 0.0.0)
endif()

option(WITH_SANITIZE_ADDRESS "Enable address sanitizer" OFF)
option(WITH_SANITIZE_THREAD "Enable thread sanitizer" OFF)

find_package(GTest REQUIRED)

add_library(async SHARED src/async_impl.cpp)
target_link_libraries(async PUBLIC pthread)

add_executable(async-load-test src/load-test.cpp)
add_executable(async-test src/test.cpp)

set_target_properties(async PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
        COMPILE_OPTIONS "-Wpedantic -Wall -Wextra")

set_target_properties(async-test PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
        COMPILE_OPTIONS "-Wpedantic -Wall -Wextra")

set_target_properties(async-load-test PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
        COMPILE_OPTIONS "-Wpedantic -Wall -Wextra")

if (WITH_SANITIZE_ADDRESS)
    set_target_properties(async-load-test PROPERTIES COMPILE_OPTIONS "-fsanitize=address")
    set_target_properties(async-test PROPERTIES COMPILE_OPTIONS "-fsanitize=address")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
endif()

if (WITH_SANITIZE_THREAD)
    set_target_properties(async-load-test PROPERTIES COMPILE_OPTIONS "-fsanitize=thread")
    set_target_properties(async-test PROPERTIES COMPILE_OPTIONS "-fsanitize=thread")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=thread")
endif()

include_directories(${GTEST_INCLUDE_DIRS})
target_link_libraries(async-test async ${GTEST_LIBRARIES})
target_link_libraries(async-load-test async ${GTEST_LIBRARIES})

install(TARGETS async-load-test RUNTIME DESTINATION bin)
install(TARGETS async-test RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT yukigaru@gmail.com)

include(CPack)

enable_testing()
add_test(test async-test)
