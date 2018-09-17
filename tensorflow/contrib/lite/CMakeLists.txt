set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS_RELEASE "-O3")
set(ANDROID_ARM_NEON true)

#include $(MAKEFILE_DIR)/ios_makefile.inc

if (${CMAKE_SYSTEM_NAME} MATCHES "Android")
    include(AndroidNdkModules)
    android_ndk_import_module_cpufeatures()
endif()

file(GLOB CORE_CC_ALL_SRCS
	*.cc
	*.c
	c/*.c
	core/api/*.cc
	kernels/*.cc
	kernels/internal/*.cc
	kernels/internal/optimized/*.cc
	kernels/internal/reference/*.cc)
list(FILTER CORE_CC_ALL_SRCS EXCLUDE REGEX "./*_test.cc")
list(FILTER CORE_CC_ALL_SRCS EXCLUDE REGEX "/kernels/test_util.cc")
list(FILTER CORE_CC_ALL_SRCS EXCLUDE REGEX "/*/*/*/*test.cc")
list(FILTER CORE_CC_ALL_SRCS EXCLUDE REGEX "/*/*/*test.cc")
list(FILTER CORE_CC_ALL_SRCS EXCLUDE REGEX "/*/*test.cc")

file(GLOB PROFILER_SRCS
	kernels/*.c
	kernels/internal/*.c
	kernels/internal/optimized/*.c
	kernels/internal/reference/*.c
	tools/make/downloads/farmhash/src/farmhash.cc
	tools/make/downloads/fft2d/fftsg.c)


#list(FILTER SOURCES EXCLUDE REGEX "tools/benchmark/*.cc")


add_library(tensorflow-lite STATIC 
	${CORE_CC_ALL_SRCS}
	${PROFILER_SRCS})
target_link_libraries(tensorflow-lite
	stdc++)

if (${CMAKE_SYSTEM_NAME} MATCHES "Android")
    target_link_libraries(tensorflow-lite cpufeatures)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    target_link_libraries(tensorflow-lite
	m z stdc++ dl pthread)
endif()

target_include_directories(tensorflow-lite PUBLIC
	../../../
	tools/make/downloads
	tools/make/downloads/gemmlowp
	tools/make/downloads/neon_2_sse
	tools/make/downloads/eigen
	tools/make/downloads/flatbuffers/include
	tools/make/downloads/farmhash/src)

