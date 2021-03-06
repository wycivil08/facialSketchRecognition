CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

set(name "facialsketchrec")
set(the_library "opencv_${name}")
project(${the_library})

if(UNIX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=gnu++0x")
	message("C++11 Activated")
endif()

find_package(OpenMP)
if (OPENMP_FOUND)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
	message("OpenMP Found") 
endif()

## Compiler flags
if(CMAKE_COMPILER_IS_GNUCXX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")        ## Optimize
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -s")  ## Strip binary
endif()

#set(OpenCV_DIR /home/marco/opencv-2.4.6.1)

#---------------------------------
# packages
#---------------------------------
find_package(OpenCV REQUIRED) # http://opencv.willowgarage.com
find_package(Boost COMPONENTS system filesystem REQUIRED)

#---------------------------------
# includes & sources
#---------------------------------
include_directories(/home/marco/vlfeat-0.9.17)
link_directories(/home/marco/vlfeat-0.9.17/bin/glnxa64)

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include")

file(GLOB lib_hdrs "include/*.hpp")
file(GLOB lib_srcs "src/*.cpp")

#---------------------------------
# build the library & link
#---------------------------------
add_library(${the_library} ${lib_srcs} ${lib_hdrs})
target_link_libraries(${the_library} ${OpenCV_LIBS} ${Boost_LIBRARIES} vl)

#---------------------------------
# add tests
#---------------------------------
if(NOT DEFINED ENABLE_TESTS)
	set(ENABLE_TESTS FALSE CACHE BOOL "Enable to make the tests")
endif()

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/test AND ENABLE_TESTS)
	file(GLOB test_srcs "test/*.cpp")
	file(GLOB test_hdrs "test/*.h*")
	set(the_test_target "opencv_test_${name}")
	add_executable(${the_test_target} ${test_srcs} ${test_hdrs})
	# link to opencv_ts, as it is not in OpenCV_LIBS for OpenCV 2.3.1 
	target_link_libraries(${the_test_target} ${the_library} ${OpenCV_LIBS})
	enable_testing()
	get_target_property(LOC ${the_test_target} LOCATION)
	add_test(${the_test_target} "${LOC}")
endif()

#---------------------------------
# add samples
#---------------------------------
if(NOT DEFINED ENABLE_SAMPLES)
	set(ENABLE_SAMPLES FALSE CACHE BOOL "Enable to make the samples")
endif()

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/samples AND ENABLE_SAMPLES) 
	# probably there's a simpler way to link against multiple executables...
	add_executable(demoLFDA samples/demoLFDA.cpp)
	target_link_libraries(demoLFDA ${the_library} ${OpenCV_LIBS})	
	add_executable(demoGaborShape samples/demoGaborShape.cpp)
	target_link_libraries(demoGaborShape ${the_library} ${OpenCV_LIBS})	
	add_executable(demoKernelProtoPRS samples/demoKernelProtoPRS.cpp)
	target_link_libraries(demoKernelProtoPRS ${the_library} ${OpenCV_LIBS})
	add_executable(demoKernelProtoDRS samples/demoKernelProtoDRS.cpp)
	target_link_libraries(demoKernelProtoDRS ${the_library} ${OpenCV_LIBS})
	add_executable(demoDescriptors samples/demoDescriptors.cpp)
	target_link_libraries(demoDescriptors ${the_library} ${OpenCV_LIBS})
	add_executable(statisticalAnalysis samples/statisticalAnalysis.cpp)
	target_link_libraries(statisticalAnalysis ${OpenCV_LIBS})
	#add_executable(test samples/test.cpp)
	#target_link_libraries(test ${OpenCV_LIBS})
endif()
