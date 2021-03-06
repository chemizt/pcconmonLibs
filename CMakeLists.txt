cmake_minimum_required(VERSION 3.10)

project(pcConMonLibs)

cmake_policy(SET CMP0074 NEW)
cmake_policy(SET CMP0079 NEW)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/out/lib/debug)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/out/lib/debug)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}/out/lib/release)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}/out/lib/release)

set(CMAKE_STATIC_LIBRARY_PREFIX "")
set(CMAKE_SHARED_LIBRARY_PREFIX "")
set(JSON_BuildTests OFF CACHE INTERNAL "")

if (UNIX)
    set(CMAKE_CXX_FLAGS_DEBUG "-g3 -ggdb -Wall -pedantic -std=c++17 -Wno-unknown-pragmas")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -pedantic -std=c++17 -Wno-unknown-pragmas")
    set(ADDITIONAL_LIBRARIES pthread)
else()
    if(MINGW)
        set(CMAKE_CXX_FLAGS_DEBUG "-g3 -ggdb -Wall -pedantic -std=c++17 -Wno-unknown-pragmas")
        set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -pedantic -std=c++17 -Wno-unknown-pragmas")
        set(ADDITIONAL_LIBRARIES ws2_32)
    elseif(WIN32)
        set(CMAKE_CXX_FLAGS_DEBUG "/Od /Wall /std:c++17")
        set(CMAKE_CXX_FLAGS_RELEASE "/O2 /Oi /Ot /GL /W0 /std:c++17")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /LTCG")
        set(ADDITIONAL_LIBRARIES ws2_32)
    endif()
endif()

if (CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_CONFIGURATION_TYPES Debug Release)
endif()

add_subdirectory(src/lib/structureLib)
add_subdirectory(src/lib/providerLib)
add_subdirectory(external/json)

target_include_directories(
    providerLib
    PUBLIC src/lib/structureLib/include
    PUBLIC external/pstreams
)

target_include_directories(
    structureLib
    PUBLIC external/json/single_include/nlohmann
)

target_link_libraries(structureLib PRIVATE nlohmann_json::nlohmann_json)
target_link_libraries(providerLib structureLib)

add_custom_target(
    debugLibs
    COMMAND ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=Debug ${CMAKE_SOURCE_DIR}
    COMMAND ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR} --target all
)

add_custom_target(
    releaseLibs
    COMMAND ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=Release ${CMAKE_SOURCE_DIR}
    COMMAND ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR} --target all
)
