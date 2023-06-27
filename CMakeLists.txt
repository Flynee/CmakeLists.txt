cmake_minimum_required(VERSION 3.2)



set(CMAKE_CXX_STANDARD 17)

PROJECT(myocc)



FILE (GLOB MYOCC_SRC "*.cpp" "*.c" "./src/*.cpp" "./src/*.cxx")
FILE (GLOB MYOCC_DIRS "*.hpp" "*.h" "./src/*.hpp"  "./src/*.h"  "./src/*.i" "./src/*.py")

macro(append_myocc_src DIRNAME FILES)
  foreach(FILE ${FILES})
    list(APPEND LIST ${DIRNAME}/${FILE})
  endforeach()
  set(MYOCC_SRC ${MYOCC_SRC};${LIST} PARENT_SCOPE)
  set(MYOCC_DIRS ${MYOCC_DIRS};${DIRNAME} PARENT_SCOPE)
endmacro()



INCLUDE_DIRECTORIES(
	.
	Common
	IO
	Mesh
	contrib/configor
	contrib/sqlite3
	contrib
	Tools
	test
	Builder
	Logger
	Primitive
	Transform
)

add_subdirectory(Common)
add_subdirectory(IO)
add_subdirectory(Mesh)
add_subdirectory(Tools)
add_subdirectory(test)
add_subdirectory(Builder)
add_subdirectory(Primitive)
add_subdirectory(Transform)
add_subdirectory(contrib/sqlite3)
add_subdirectory(Logger)

# group sources for easier navigation in IDEs
foreach(DIR ${MYOCC_DIRS})
  string(REGEX REPLACE "\\+" "\\\\+" DIR ${DIR})
  source_group(${DIR} REGULAR_EXPRESSION ${DIR}/.*)
endforeach()

FIND_PACKAGE( OpenMP REQUIRED)
if(OPENMP_FOUND)
message("OPENMP FOUND")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()


SET (ALL_SRCS 
	 ${MYOCC_SRC}
	 ${MYOCC_DIRS}
)

AUX_SOURCE_DIRECTORY(src DIR_SRCS)



#OCC
find_path(OCC_INC "Standard_Version.hxx" HINTS ENV CASROOT PATH_SUFFIXES
            inc include include/oce opencascade include/opencascade
            occt include/occt)

IF(OCC_INC)
	include_directories(${OCC_INC})
ENDIF()		
	
IF (WIN32)
	MESSAGE(STATUS "Now is windows")
	IF(OCC_INC_DIR AND OCC_LINK_DIR)
		include_directories(${OCC_INC_DIR})
		link_directories(${OCC_LINK_DIR})
	ENDIF ()

	link_libraries(TKBin TKBinL TKBinTObj TKBinXCAF TKBO
	TKBool TKBRep TKCAF TKCDF TKD3DHost TKD3DHostTest TKDCAF
	TKDraw TKernel TKFeat TKFillet TKG2d TKG3d TKGeomAlgo TKGeomBase
	TKHLR TKIGES TKLCAF TKMath TKMesh TKMeshVS TKOffset TKOpenGl TKOpenGlTest
	TKPrim TKQADraw TKRWMesh TKService TKShHealing TKStd TKStdL TKSTEP TKSTEP209
	TKSTEPAttr TKSTEPBase TKSTL TKTObj TKTObjDRAW TKTopAlgo TKTopTest TKV3d
	TKVCAF TKViewerTest TKVRML TKXCAF TKXDEDRAW TKXDEIGES TKXDESTEP TKXMesh TKXml
	TKXmlL TKXmlTObj TKXmlXCAF TKXSBase TKXSDRAW ws2_32)

ELSEIF (APPLE)
	MESSAGE(STATUS "Now is Apple systens.")
ELSEIF (UNIX)
	MESSAGE(STATUS "Now is UNIX-like OS's.")
	
	IF(OCC_INC_DIR AND OCC_LINK_DIR)
		include_directories(${OCC_INC_DIR})
		link_directories(${OCC_LINK_DIR})
	ENDIF ()

	link_libraries(TKBin TKBinL TKBinTObj TKBinXCAF TKBO
	TKBool TKBRep TKCAF TKCDF TKDCAF
	TKDraw TKernel TKFeat TKFillet TKG2d TKG3d TKGeomAlgo TKGeomBase
	TKHLR TKIGES TKLCAF TKMath TKMesh TKMeshVS TKOffset TKOpenGl TKOpenGlTest
	TKPrim TKQADraw TKRWMesh TKService TKShHealing TKStd TKStdL TKSTEP TKSTEP209
	TKSTEPAttr TKSTEPBase TKSTL TKTObj TKTObjDRAW TKTopAlgo TKTopTest TKV3d
	TKVCAF TKViewerTest TKVRML TKXCAF TKXDEDRAW TKXDEIGES TKXDESTEP TKXMesh TKXml
	TKXmlL TKXmlTObj TKXmlXCAF TKXSBase TKXSDRAW)
ENDIF ()

#EXE
ADD_EXECUTABLE(${PROJECT_NAME} ${ALL_SRCS})


##########pyd###########################################
#DLL
add_library(myocclib SHARED ${ALL_SRCS})
target_link_libraries(${PROJECT_NAME} myocclib)


#寻找PYTHONHOME的环境变量
if(Python3_INCLUDE_DIRS)
	MESSAGE(STATUS "sp Python3_INCLUDE_DIRS")
else()
	find_package (Python3 COMPONENTS Interpreter Development)
endif()

include_directories(${Python3_INCLUDE_DIRS})
link_libraries(${Python3_LIBRARIES})
MESSAGE(${Python3_INCLUDE_DIRS})
MESSAGE(${Python3_LIBRARIES})

#指定你的.cxx等文件的目录
include_directories(${PROJECT_SOURCE_DIR}/src)

#寻找安装好的swig,其实就是去电脑中找你安装好的Swig环境，所以我们需要提前安装环境。
find_package(SWIG REQUIRED)
include(${SWIG_USE_FILE})
#Python文件输出目录 -outdir
set(CMAKE_SWIG_OUTDIR ${CMAKE_CURRENT_BINARY_DIR}/python)
#指定一个输出目录名，用于放置生成的源文件。例如输出的examplePTYHON_wrap.cxx文件等的位置
set(SWIG_OUTFILE_DIR ${CMAKE_CURRENT_BINARY_DIR}/cpp)
#c ++模式
set_property(SOURCE Common/myocc.i PROPERTY CPLUSPLUS ON)
#开始生成 
swig_add_library(myocc LANGUAGE python SOURCES Common/myocc.i ${ALL_SRCS})
#####################################################

#生成
# set_property(SOURCE  MyCore/builder.i PROPERTY CPLUSPLUS ON)
# swig_add_library(builder LANGUAGE python SOURCES MyCore/builder.i ${ALL_SRCS})
# set_property(SOURCE  MyCore/meshfunc.i PROPERTY CPLUSPLUS ON)
# swig_add_library(meshfunc LANGUAGE python SOURCES MyCore/meshfunc.i ${ALL_SRCS})
# set_property(SOURCE  MyCore/stringutils.i PROPERTY CPLUSPLUS ON)
# swig_add_library(stringutils LANGUAGE python SOURCES MyCore/stringutils.i ${ALL_SRCS})
# set_property(SOURCE  MyCore/triangle.i PROPERTY CPLUSPLUS ON)
# swig_add_library(triangle LANGUAGE python SOURCES MyCore/triangle.i ${ALL_SRCS})
# set_property(SOURCE  MyCore/os.i PROPERTY CPLUSPLUS ON)
# swig_add_library(os LANGUAGE python SOURCES MyCore/os.i ${ALL_SRCS})
# set_property(SOURCE  MyCore/os_str.i PROPERTY CPLUSPLUS ON)
# swig_add_library(os_str LANGUAGE python SOURCES MyCore/os_str.i ${ALL_SRCS})


