cmake_minimum_required(VERSION 3.5)

project(aio)

find_package(ament_cmake REQUIRED)

macro(build_libaio)
  set(extra_cmake_args)

  # CMAKE_BUILD_TYPE was removed in ros2:pybind11_vendor#3 to eliminate warning
  # on Windows about unused variable. It is only used for tests, which we are
  # not building, and in pybind11Tools.cmake for Linux and Mac. In any case,
  # pybind11 is a headers-only class and does not require building at the time
  # of this vendor package, thus does not require passing in the variable here.
  # It will use the variable value in packages that depend on it for compiling.

  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND extra_cmake_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
    if(ANDROID)
      if(DEFINED ANDROID_ABI)
        list(APPEND extra_cmake_args "-DANDROID_ABI=${ANDROID_ABI}")
      endif()
      if(DEFINED ANDROID_CPP_FEATURES)
        list(APPEND extra_cmake_args "-DANDROID_CPP_FEATURES=${ANDROID_CPP_FEATURES}")
      endif()
      if(DEFINED ANDROID_FUNCTION_LEVEL_LINKING)
        list(APPEND extra_cmake_args "-DANDROID_FUNCTION_LEVEL_LINKING=${ANDROID_FUNCTION_LEVEL_LINKING}")
      endif()
      if(DEFINED ANDROID_NATIVE_API_LEVEL)
        list(APPEND extra_cmake_args "-DANDROID_NATIVE_API_LEVEL=${ANDROID_NATIVE_API_LEVEL}")
      endif()
      if(DEFINED ANDROID_NDK)
        list(APPEND extra_cmake_args "-DANDROID_NDK=${ANDROID_NDK}")
      endif()
      if(DEFINED ANDROID_STL)
        list(APPEND extra_cmake_args "-DANDROID_STL=${ANDROID_STL}")
      endif()
      if(DEFINED ANDROID_TOOLCHAIN_NAME)
        list(APPEND extra_cmake_args "-DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME}")
      endif()
    endif()
  else()
    list(APPEND extra_cmake_args "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
  endif()
  list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")

  set(CMAKE_POSITION_INDEPENDENT_CODE ON)
  file(GLOB SRCS "src/*.c")
  add_library(${PROJECT_NAME} SHARED ${SRCS})
  target_include_directories(
    ${PROJECT_NAME}
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/src>)
  set(CMAKE_BUILD_TYPE "Debug")

  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -fomit-frame-pointer -O2 -Wall -fPIC -D__aarch64__ -DANDROID")

  # The external project will install to the build folder, but we'll install that on make install.
  install(
    TARGETS ${PROJECT_NAME}
    EXPORT export_${PROJECT_NAME}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin)
  install(FILES ${CMAKE_CURRENT_LIST_DIR}/src/libaio.h DESTINATION include) 
endmacro()

# Currently, always build.
# This could be improved with logic to use installed platform packages.
build_libaio()

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_interfaces(export_${PROJECT_NAME})
ament_export_dependencies(ament_cmake)

ament_package()
