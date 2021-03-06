cmake_minimum_required(VERSION 3.10)
project(RLBot LANGUAGES CXX)

# find boost
add_definitions(-DBOOST_ALL_NO_LIB)

set(Boost_USE_STATIC_LIBS        ON)
set(Boost_USE_MULTITHREADED      ON)
set(Boost_USE_STATIC_RUNTIME    OFF)

find_package(Boost COMPONENTS date_time system REQUIRED)

# Include directories for flatbuffers library and generated files
set(rlbot_generated ${CMAKE_CURRENT_SOURCE_DIR}/../../../generated/cpp/flatbuffers)
set(flatbuffers ${CMAKE_CURRENT_SOURCE_DIR}/lib/flatbuffers/include)

# Include directory for chips ball prediction
set(rlu_include ${CMAKE_CURRENT_SOURCE_DIR}/chip/cpp/inc)

# ---- RLBotMessage ----
file(GLOB_RECURSE rlbotmessages_sources  ${CMAKE_CURRENT_SOURCE_DIR}/src/RLBotMessages/*.?pp)

add_library(RLBotMessages STATIC ${rlbotmessages_sources})
target_include_directories(RLBotMessages PRIVATE
    src/RLBotMessages
    ${rlbot_generated}
    ${flatbuffers}
    ${Boost_INCLUDE_DIRS})
target_link_libraries(RLBotMessages ${Boost_LIBRARIES})
target_compile_features(RLBotMessages PRIVATE cxx_std_17)

if (UNIX)
    set_target_properties(RLBotMessages PROPERTIES COMPILE_FLAGS -fPIC)
endif (UNIX)

# ---- RLBotInterface ----
file(GLOB_RECURSE rlbotinterface_sources  ${CMAKE_CURRENT_SOURCE_DIR}/src/RLBotInterface/*.?pp)

add_library(RLBotInterface SHARED ${rlbotinterface_sources})
target_include_directories(RLBotInterface PRIVATE
    src/RLBotInterface
    src/RLBotMessages
    ${rlbot_generated}
    ${flatbuffers}
    ${Boost_INCLUDE_DIRS})

if (UNIX)
    target_link_libraries(RLBotInterface RLBotMessages ${Boost_LIBRARIES} rt)
    set_target_properties(RLBotInterface PROPERTIES COMPILE_FLAGS -fPIC)
else (UNIX)
    target_link_libraries(RLBotInterface RLBotMessages ${Boost_LIBRARIES})
endif (UNIX)

target_compile_features(RLBotInterface PRIVATE cxx_std_17)

# ---- RLBot ----
file(GLOB_RECURSE rlbot_sources  ${CMAKE_CURRENT_SOURCE_DIR}/src/RLBot/*.?pp)
file(GLOB_RECURSE rlu_sources  ${CMAKE_CURRENT_SOURCE_DIR}/chip/cpp/*.cc)

add_executable(RLBot ${rlbot_sources} ${rlu_sources})
target_include_directories(RLBot PRIVATE
    src/RLBot
    src/RLBotMessages
    ${rlbot_generated}
    ${flatbuffers}
    ${Boost_INCLUDE_DIRS}
    ${rlu_include})

if (UNIX)
    target_link_libraries(RLBot RLBotMessages ${Boost_LIBRARIES} pthread rt)

    add_custom_command(TARGET RLBotMessages
                       PRE_BUILD
                       COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/src/RLBotMessages/generate-flatbuffers.sh )

else (UNIX)
    target_link_libraries(RLBot RLBotMessages ${Boost_LIBRARIES})
endif (UNIX)

target_compile_features(RLBot PRIVATE cxx_std_17)

if (WIN32)
    # Set target version to windows xp. This should in theory work as a minimum supported platform.
    target_compile_definitions(RLBot PRIVATE _WIN32_WINNT=0x501)

    add_custom_command(TARGET RLBotMessages
                       PRE_BUILD
                       COMMAND ${CMAKE_CURRENT_SOURCE_DIR}\\src\\RLBotMessages\\generate-flatbuffers.bat )
endif()

# ---- InterfaceTester ----
file(GLOB_RECURSE interfacetester_sources  ${CMAKE_CURRENT_SOURCE_DIR}/src/InterfaceTester/*.?pp)

add_executable(InterfaceTester ${interfacetester_sources})
target_include_directories(InterfaceTester PRIVATE
    src/InterfaceTester
    src/RLBotInterface
    src/RLBotMessages
    ${rlbot_generated}
    ${flatbuffers})

if (UNIX)
    target_link_libraries(InterfaceTester RLBotInterface pthread rt)
else (UNIX)
    target_link_libraries(InterfaceTester RLBotInterface)
endif (UNIX)

target_compile_features(InterfaceTester PRIVATE cxx_std_17)
