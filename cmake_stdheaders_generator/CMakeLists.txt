cmake_minimum_required(VERSION 3.0)

function(generate_mingw_stdthreads_header header_file_name 
                                          mingw_stdthreads_folder)
    # Check if compiler is gcc or clang
    if (NOT CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
        # Actually this should never happen because it should have already
        # been checked in the parent CMakeLists.txt
        message(FATAL_ERROR "Unsupported compiler")
    endif()

    # Call g++ to retrieve header path
    # The -H option will let g++ outputs header dependencies to stderr, and the
    # content of stderr is saved to variable compiler_output.
    try_compile(unused_compiler_exit_code ${CMAKE_CURRENT_BINARY_DIR}
        "${PROJECT_SOURCE_DIR}/cmake_stdheaders_generator/template.cpp"
        COMPILE_DEFINITIONS
            -H "-DMINGW_STDTHREADS_DETECTING_SYSTEM_HEADER=<${header_file_name}>"
        OUTPUT_VARIABLE compiler_output)
    
    # Get full path to system header
    string(REGEX MATCH "[.] ([^\r\n]*)" _ "${compiler_output}")
    set(mingw_stdthreads_headers_generator_system_header "${CMAKE_MATCH_1}")
    message("Matched: <${mingw_stdthreads_headers_generator_system_header}>")

    # Ensure file exists
    if(NOT EXISTS "${mingw_stdthreads_headers_generator_system_header}")
        message(FATAL_ERROR "<${header_file_name}>'s path not found, "
                            "compiler output was:\n${compiler_output}")
    endif()

    get_filename_component(header_file_name "${header_file_name}" NAME_WE)
    # Get full path to mingw-stdthreads header
    set(mingw_stdthreads_headers_generator_library_header
        "${mingw_stdthreads_folder}/mingw.${header_file_name}.h")

    # Normalize paths
    file(TO_CMAKE_PATH "${mingw_stdthreads_headers_generator_system_header}"
        mingw_stdthreads_headers_generator_system_header)
    file(TO_CMAKE_PATH "${mingw_stdthreads_headers_generator_library_header}"
        mingw_stdthreads_headers_generator_library_header)

    configure_file("${PROJECT_SOURCE_DIR}/cmake_stdheaders_generator/template.cpp" "${PROJECT_BINARY_DIR}/include/${header_file_name}.h")
endfunction()

if(EXISTS "${MINGW_STDTHREADS_DIR}")
    message("${PROJECT_NAME}: MINGW_STDTHREADS_DIR: "
            "${MINGW_STDTHREADS_DIR}")
else()
    message(FATAL_ERROR "${PROECT_NAME}: MINGW_STDTHREADS_DIR does not "
                        "exist: ${MINGW_STDTHREADS_DIR}")
endif()

# <condition_variable>
generate_mingw_stdthreads_header(condition_variable "${MINGW_STDTHREADS_DIR}")
# <future>
generate_mingw_stdthreads_header(future "${MINGW_STDTHREADS_DIR}")
# <mutex>
generate_mingw_stdthreads_header(mutex "${MINGW_STDTHREADS_DIR}")
# <shared_mutex>
generate_mingw_stdthreads_header(shared_mutex "${MINGW_STDTHREADS_DIR}")
# <thread>
generate_mingw_stdthreads_header(thread "${MINGW_STDTHREADS_DIR}")
# <bits/c++config.h>
generate_mingw_stdthreads_header(bits/c++config.h "${MINGW_STDTHREADS_DIR}")

# the generated headers are to be considered as a header only library
# so we create an interface target
target_compile_definitions(${PROJECT_NAME} INTERFACE 
                           MINGW_STDTHREADS_GENERATED_STDHEADERS)
target_include_directories(${PROJECT_NAME} INTERFACE "${PROJECT_BINARY_DIR}/include")
