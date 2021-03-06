CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

PROJECT(octoon)

SET(ENGINE_DIR ${PROJECT_SOURCE_DIR}/lib/engine)
SET(DEPENDENCIES_PATH ${PROJECT_SOURCE_DIR}/contrib)
SET(SAMPLE_PATH ${PROJECT_SOURCE_DIR}/sample)
SET(TOOLS_PATH ${PROJECT_SOURCE_DIR}/tools)
SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
SET(COTOON_HEADER_PATH ${CMAKE_SOURCE_DIR}/include/octoon)
SET(COTOON_SOURCE_PATH ${CMAKE_SOURCE_DIR}/source)

OPTION(BUILD_SSE "ON for use OFF for ignore" OFF)       
OPTION(BUILD_DEBUG_MODE "ON for debug or OFF for release" ON)
OPTION(BUILD_MUTILTHREAD_DLL "ON for /MD OFF for /MT" ON)

IF(ANDROID_ABI OR CMAKE_SYSTEM_NAME MATCHES "VCMDDAndroid")
    SET(PLATFORM 3)
    SET(VCMDDAndroid 1)
ELSEIF(WIN32)
    SET(PLATFORM 4)
ELSEIF(APPLE)
    SET(PLATFORM 1)
ELSEIF(UNIX)
    SET(PLATFORM 2)
ELSE()
    SET(PLATFORM 4)
ENDIF()

SET(BUILD_PLATFORM ${PLATFORM} CACHE STRING
"Specify the platform. Possible values:
  1 - Apple
  2 - Linux
  3 - Adnroid
  4 - Windows"
)

IF(BUILD_PLATFORM EQUAL 1)
    SET(BUILD_PLATFORM_APPLE TRUE)
    ADD_DEFINITIONS(-DAPPLE)
    ADD_DEFINITIONS(-D_BUILD_PLATFORM_APPLE)
ELSEIF(BUILD_PLATFORM EQUAL 2)
    SET(BUILD_PLATFORM_LINUX TRUE)
    ADD_DEFINITIONS(-DLINUX)
    ADD_DEFINITIONS(-D_BUILD_PLATFORM_LINUX)
ELSEIF(BUILD_PLATFORM EQUAL 3)
    SET(BUILD_PLATFORM_ANDROID TRUE)
    ADD_DEFINITIONS(-DANDROID)
    ADD_DEFINITIONS(-D_BUILD_PLATFORM_ANDROID)
ELSEIF(BUILD_PLATFORM EQUAL 4)
    SET(BUILD_PLATFORM_WINDOWS TRUE)
    ADD_DEFINITIONS(-DWINDOWS)
    ADD_DEFINITIONS(-D_BUILD_PLATFORM_WINDOWS)
ENDIF()

IF(BUILD_DEBUG_MODE)
    SET(CMAKE_BUILD_TYPE Debug CACHE STRING "One of None Debug Release RelWithDebInfo MinSizeRel" FORCE)
ELSE()
    SET(CMAKE_BUILD_TYPE Release CACHE STRING "One of None Debug Release RelWithDebInfo MinSizeRel" FORCE)
ENDIF()

IF(BUILD_PLATFORM_LINUX OR BUILD_PLATFORM_ANDROID)
    SET(PLATFORM_NAME "linux")
ELSEIF(BUILD_PLATFORM_APPLE)
    SET(PLATFORM_NAME "apple")
ELSEIF(BUILD_PLATFORM_WINDOWS)
    SET(PLATFORM_NAME "win")
    ADD_DEFINITIONS(-D_WINDOWS)
    ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)

    SET(COMPILER_NAME "vc")
    IF(MSVC_VERSION GREATER 1600)
        SET(COMPILER_NAME "11")
    ELSEIF(MSVC_VERSION GREATER 1500)
        SET(COMPILER_NAME "10")
    ELSEIF(MSVC_VERSION GREATER 1400)
        SET(COMPILER_NAME "9")
    ENDIF()
ENDIF()

IF(CMAKE_GENERATOR MATCHES "ARM")
    SET(LIBRARY_OUT_NAME "ARM")
ELSEIF(CMAKE_GENERATOR MATCHES "Win64")
    SET(LIBRARY_OUT_NAME "64")
    SET(WIN64 TRUE)
ELSEIF(CMAKE_GENERATOR MATCHES "Visual Studio")
    SET(LIBRARY_OUT_NAME "86")
    SET(WIN32 TRUE)
ELSE()
    SET(LIBRARY_OUT_NAME "86")
ENDIF()

# 设置编译文件后缀
SET(CMAKE_DEBUG_POSTFIX "_d" CACHE STRING "adds a postfix for debug-built libraries.")
SET(CMAKE_RELEASE_POSTFIX "" CACHE STRING "Adds a postfix for release-built libraries.")
SET(CMAKE_RELWITHDEBINFO_POSTFIX "_RelWithDebugInfo" CACHE STRING "adds a postfix for ReleaseWithDebug-built libraries.")
SET(CMAKE_MINSIZEREL_POSTFIX "_MinsizeRel" CACHE STRING "adds a postfix for MinsizeRelease-built libraries.")

# 指定库文件输出路径
SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib/binaries/${PLATFORM_NAME}${LIBRARY_OUT_NAME})
SET(LIBRARY_OUTPUT_PATH_DEBUG ${LIBRARY_OUTPUT_PATH})
SET(LIBRARY_OUTPUT_PATH_RELEASE ${LIBRARY_OUTPUT_PATH})
SET(LIBRARY_OUTPUT_PATH_RELWITHDEBINFO ${LIBRARY_OUTPUT_PATH})
SET(LIBRARY_OUTPUT_PATH_MINSIZEREL ${LIBRARY_OUTPUT_PATH})

SET(CMAKE_CXX_STANDARD 17)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_CXX_EXTENSIONS OFF)

SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)

MACRO(SET_TARGET_ATTRIBUTE target directory)
    SET_TARGET_PROPERTIES(${target} PROPERTIES
        ARCHIVE_OUTPUT_DIRECTORY ${LIBRARY_OUTPUT_PATH}
        ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${LIBRARY_OUTPUT_PATH_DEBUG}
        ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${LIBRARY_OUTPUT_PATH_RELEASE}
        ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LIBRARY_OUTPUT_PATH_RELWITHDEBINFO}
        ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL ${LIBRARY_OUTPUT_PATH_MINSIZEREL}
        LIBRARY_OUTPUT_DIRECTORY ${LIBRARY_OUTPUT_PATH}
        LIBRARY_OUTPUT_DIRECTORY_DEBUG ${LIBRARY_OUTPUT_PATH_DEBUG}
        LIBRARY_OUTPUT_DIRECTORY_RELEASE ${LIBRARY_OUTPUT_PATH_RELEASE}
        LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LIBRARY_OUTPUT_PATH_RELWITHDEBINFO}
        LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL ${LIBRARY_OUTPUT_PATH_MINSIZEREL}
        RUNTIME_OUTPUT_DIRECTORY ${LIBRARY_OUTPUT_PATH}
        RUNTIME_OUTPUT_DIRECTORY_DEBUG ${LIBRARY_OUTPUT_PATH_DEBUG}
        RUNTIME_OUTPUT_DIRECTORY_RELEASE ${LIBRARY_OUTPUT_PATH_RELEASE}
        RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LIBRARY_OUTPUT_PATH_RELWITHDEBINFO}
        RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${LIBRARY_OUTPUT_PATH_MINSIZEREL}
    )
    SET_TARGET_PROPERTIES(${target} PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
    SET_TARGET_PROPERTIES(${target} PROPERTIES PROJECT_LABEL ${target})
    SET_TARGET_PROPERTIES(${target} PROPERTIES FOLDER ${directory})
    IF(MINGW OR CMAKE_GENERATOR MATCHES "Unix")
        STRING(REGEX REPLACE "lib" "" target_name ${target})
        SET_TARGET_PROPERTIES(${target} PROPERTIES OUTPUT_NAME ${target_name})
    ENDIF()
    IF(CMAKE_GENERATOR MATCHES "Visual Studio")
        IF(VCMDDAndroid)
            SET_PROPERTY(TARGET ${LIB_NAME} PROPERTY VC_MDD_ANDROID_API_LEVEL "android-21")
        ELSEIF(${CMAKE_VS_PLATFORM_TOOLSET} STREQUAL "v140_clang_3_7")
            SET_PROPERTY(TARGET ${LIB_NAME} PROPERTY VS_PLATFORM_TOOLSET_OVERRIDE "v140_clang_3_7")
        ENDIF()
    ENDIF()
ENDMACRO()

IF(MINGW OR CMAKE_GENERATOR MATCHES "Unix")
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -W")

    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -W")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wignored-qualifiers")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -finput-charset=utf-8")

    IF(UNIX)
        SET(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
    ENDIF()

    IF(SSE)
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse")
    ELSEIF(SSE2)
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse2")
    ELSEIF(SSE3)
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse3")
    ELSEIF(SSE4)
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse3")
    ENDIF()
ELSEIF(CMAKE_GENERATOR MATCHES "Xcode")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -frtti")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w")

        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -frtti")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fexceptions")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++1y")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w")
ELSEIF(CMAKE_GENERATOR MATCHES "Visual Studio")
    IF(${CMAKE_VS_PLATFORM_TOOLSET} STREQUAL "v140_clang_3_7" OR ${CMAKE_VS_PLATFORM_TOOLSET} STREQUAL VCMDDAndroid)
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -frtti")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w")

        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -frtti")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fexceptions")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++1y")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w")
    ELSE()
       # 运行时类型信息
        SET(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} /GR")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /GR")

        # 快速的浮点模型
        SET(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} /fp:fast")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fp:fast")

        # 多处理器编译
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")

        SET(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} /W0")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W0")
        
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /std:c++latest")

        # SEH
        SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /EHa")
        SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /EHa")
        SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /EHa")
        SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /EHa")

        IF(BUILD_MUTILTHREAD_DLL)
            SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MDd")
            SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MD")
            SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
            SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
        ELSE()
            SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd")
            SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")
            SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
            SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
        ENDIF()

        SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /O2")
        SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Ob2")
        SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Ot")
        SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /GL")

        IF(BUILD_SSE)
            SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /arch:SSE")
        ENDIF()
    ENDIF()
ENDIF()

# 添加依赖库
ADD_SUBDIRECTORY(contrib)

# 添加项目
ADD_SUBDIRECTORY(source)

# 工具
# ADD_SUBDIRECTORY(tools)