include("../common/common.cmake")

######################################################################
# fp_test target

add_executable(fp_test
        # slowest
        fixed_point/elastic/elastic_fixed_point.cpp

        cnlint.cpp
        p0381.cpp
        boost.simd.cpp
        precise/elastic/precise_elastic_integer.cpp
        precise/precise_integer.cpp
        overflow.cpp
        elastic/elastic_integer.cpp
        elastic/precise/safe/precise_safe_elastic_integer.cpp
        snippets.cpp
        zero_cost_square.cpp
        p0554.cpp
        index.cpp
        integral_constant.cpp
        limits.cpp
        number_test.cpp
        zero_cost_free_functions.cpp
        boost.multiprecision.cpp
        zero_cost_average.cpp
        readme.cpp
        cppnow2017.cpp
        safft.cpp
        safe/elastic/safe_elastic_integer.cpp
        safe/safe_integer.cpp
        num_traits.cpp
        numeric.cpp
        p0675.cpp
        glm.cpp
        common.cpp
        fixed_point/precise_fixed_point.cpp
        fixed_point/fixed_point_constants.cpp
        fixed_point/fixed_point_extras.cpp
        fixed_point/fixed_point_math.cpp
        fixed_point/elastic/precise/safe/precise_safe_elastic_fixed_point.cpp
        fixed_point/elastic/make_elastic_fixed_point.cpp
        fixed_point/safe/fixed_point_native_integer.cpp
        fixed_point/safe/fixed_point_throwing_integer.cpp
        fixed_point/safe/fixed_point_saturated_integer.cpp
        fixed_point/p0037.cpp
        fixed_point/fixed_point_built_in.cpp
        main.cpp)

######################################################################
# find external lib, boost.multiprecision

find_package(Boost)

if(Boost_FOUND)
    if(Boost_VERSION GREATER 105500 OR EXCEPTIONS)
        # only compile multiprecision.cpp if Boost.Multiprecision is available
        set(BOOST_FOUND_CXX_FLAGS "-DCNL_BOOST_ENABLED")
    else(Boost_VERSION GREATER 105500 OR EXCEPTIONS)
        message("Boost version is too low to use without exception handling")
    endif(Boost_VERSION GREATER 105500 OR EXCEPTIONS)
else(Boost_FOUND)
    message("Boost not found")
endif(Boost_FOUND)

######################################################################
# pull external project, boost.simd

if(Boost_FOUND AND Boost_VERSION GREATER 106099)
    ExternalProject_Add(
            boost.simd
            URL "https://github.com/johnmcfarlane/boost.simd/archive/5d67a72d77411dc610126151a182ebbae1bfbac6.zip"
            URL_MD5 "97d1aad967965ee79a751d099cd0ee6f"
            UPDATE_COMMAND ""
            INSTALL_COMMAND ""
    )

    ExternalProject_Get_Property(boost.simd source_dir)
    target_include_directories(
            fp_test
            PRIVATE
            SYSTEM
            "${source_dir}/include"
    )

    add_dependencies(
            fp_test
            boost.simd
    )

    set(BOOST_SIMD_FOUND_CXX_FLAGS "-DCNL_BOOST_SIMD_ENABLED")
endif(Boost_FOUND AND Boost_VERSION GREATER 106099)

######################################################################
# pull external project, glm

ExternalProject_Add(
    glm
    URL "https://github.com/johnmcfarlane/glm/archive/878086621522255b14017c26d1df3b9fb0b36009.zip"
    URL_MD5 "fe3b032c230df63474bc26549526aaec"
    UPDATE_COMMAND ""
    INSTALL_COMMAND ""
)

ExternalProject_Get_Property(glm source_dir)
target_include_directories(
    fp_test
    PRIVATE
    SYSTEM
    ${source_dir}
)

add_dependencies(
    fp_test
    glm
)

######################################################################
# pull external project, google_test

ExternalProject_Add(
        google_test
        URL "https://github.com/google/googletest/archive/c99458533a9b4c743ed51537e25989ea55944908.zip"
        URL_MD5 "4552721bde3dcaab1eaa9582afc28c9d"
        UPDATE_COMMAND ""
        INSTALL_COMMAND ""
        CMAKE_ARGS -Dgtest_force_shared_crt=ON
)

ExternalProject_Get_Property(google_test source_dir)
target_include_directories(
        fp_test
        PRIVATE
        SYSTEM
        ${source_dir}/include
)

ExternalProject_Get_Property(google_test binary_dir)
if (${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
    target_link_libraries(
            fp_test
            debug ${binary_dir}/Debug/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${CMAKE_FIND_LIBRARY_SUFFIXES}
            optimized ${binary_dir}/Release/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${CMAKE_FIND_LIBRARY_SUFFIXES}
    )
else ()
    target_link_libraries(
            fp_test
            general ${binary_dir}/${CMAKE_FIND_LIBRARY_PREFIXES}gtest.a
            general pthread
    )
endif ()

add_dependencies(
        fp_test
        google_test
)

######################################################################
# fp_test compiler flags

set_target_properties(
        fp_test
        PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${BOOST_FOUND_CXX_FLAGS} ${BOOST_SIMD_FOUND_CXX_FLAGS}"
)

target_link_libraries(fp_test cnl)

# avoid GCC6 warning
# http://stackoverflow.com/a/38650955/671509
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set_source_files_properties(
            boost.multiprecision.cpp
            index.cpp
            PROPERTIES COMPILE_FLAGS "-fpermissive")

    set_source_files_properties(
            fixed_point/safe/fixed_point_saturated_integer.cpp
            safe/safe_integer.cpp
            PROPERTIES COMPILE_FLAGS "-Wno-sign-compare")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set_source_files_properties(
            fixed_point/safe/fixed_point_saturated_integer.cpp
            safe/safe_integer.cpp
            PROPERTIES COMPILE_FLAGS "-Wno-sign-compare")

    set_source_files_properties(
            fixed_point/fixed_point_math.cpp
            PROPERTIES COMPILE_FLAGS "-Wno-integer-overflow")
endif ()

######################################################################
# testing

enable_testing()
add_test(all_tests fp_test)
