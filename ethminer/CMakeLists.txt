cmake_policy(SET CMP0015 NEW)
set(CMAKE_AUTOMOC OFF)

aux_source_directory(. SRC_LIST)

include_directories(BEFORE ..)
include_directories(${Boost_INCLUDE_DIRS})

if (JSONRPC)
	include_directories(BEFORE ${JSONCPP_INCLUDE_DIRS})
	include_directories(${JSON_RPC_CPP_INCLUDE_DIRS})
endif()

if (ETHASHCUDA)
	include_directories(${CUDA_INCLUDE_DIRS})
endif ()

set(EXECUTABLE ethminer)

file(GLOB HEADERS "*.h")

if (NOT DEFINED WIN32)
	list(REMOVE_ITEM SRC_LIST ./speedfan.cpp)
	list(REMOVE_ITEM HEADERS ./speedfan.h)
endif()

add_executable(${EXECUTABLE} ${SRC_LIST} ${HEADERS})

add_dependencies(${EXECUTABLE} BuildInfo.h)

target_link_libraries(${EXECUTABLE} ${Boost_REGEX_LIBRARIES})
target_link_libraries(${EXECUTABLE} ${CMAKE_DL_LIBS})

if (JSONRPC)
	target_link_libraries(${EXECUTABLE} ${JSON_RPC_CPP_CLIENT_LIBRARIES})
	target_link_libraries(${EXECUTABLE} ${CURL_LIBRARIES})
	if (DEFINED WIN32 AND NOT DEFINED CMAKE_COMPILER_IS_MINGW)
		eth_copy_dlls(${EXECUTABLE} CURL_DLLS)
	endif()
endif()

target_link_libraries(${EXECUTABLE} ethcore)
target_link_libraries(${EXECUTABLE} ethash)
target_link_libraries(${EXECUTABLE} devcore)

if (ETHSTRATUM)
target_link_libraries(${EXECUTABLE} ethstratum)
endif()

if (DEFINED WIN32 AND NOT DEFINED CMAKE_COMPILER_IS_MINGW)
	eth_copy_dlls("${EXECUTABLE}" MHD_DLLS)
	eth_copy_dlls("${EXECUTABLE}" OpenCL_DLLS)
endif()


if (APPLE)
	install(TARGETS ${EXECUTABLE} DESTINATION bin)
else()
	eth_install_executable(${EXECUTABLE})
endif()
