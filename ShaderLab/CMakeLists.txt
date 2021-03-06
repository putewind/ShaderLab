﻿set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -DNOMINMAX")

if (WIN32)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DPLATFORM_WINDOWS=1")
elseif (UNIX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DPLATFORM_LINUX=1")
endif ()

set(SL_PARSER_HDRS
    Parser/SLParser.h
    Parser/SLParserTypes.h
    Parser/SLParserData.h
    Parser/SLParserYacc.hpp
    Parser/SLParserLex.l
    Parser/SLParserYacc.y
)

set(SL_PARSER_SRCS
    Parser/SLParser.cpp
    Parser/SLParserLex.cpp
    Parser/SLParserYacc.cpp
)

set(SL_COMPILER_HDRS
    Compiler/SLCompiler.h
    Compiler/HLSLCompiler.h
    Compiler/GLSLCompiler.h
)

set(SL_COMPILER_SRCS
    Compiler/SLCompiler.cpp
    Compiler/HLSLCompiler.cpp
    Compiler/GLSLCompiler.cpp
)

set(SL_COMMON_HDRS
    Common/Common.h
)

set(SL_COMMON_SRCS
    
)

set(SL_UTILS_HDRS
    Utils/StringUtils.h
    Utils/json.h
    Utils/json-forwards.h
)
set(SL_UTILS_SRCS
    Utils/StringUtils.cpp
    Utils/jsoncpp.cpp
)

add_library(shaderlab STATIC
	${SL_PARSER_HDRS}
	${SL_PARSER_SRCS}

    ${SL_COMPILER_HDRS}
    ${SL_COMPILER_SRCS}

    ${SL_COMMON_HDRS}
    ${SL_COMMON_SRCS}

    ${SL_UTILS_HDRS}
    ${SL_UTILS_SRCS}
)

source_group(Parser\\ FILES ${SL_PARSER_HDRS} ${SL_PARSER_SRCS})
source_group(Compiler\\ FILES ${SL_COMPILER_HDRS} ${SL_COMPILER_SRCS})
source_group(Common\\ FILES ${SL_COMMON_HDRS} ${SL_COMMON_SRCS})
source_group(Utils\\ FILES ${SL_UTILS_HDRS} ${SL_UTILS_SRCS})

if(WIN32)
    set(DXCompilerName "dxcompiler.dll")
    set(DXCompilerOutputFolder "bin")
else()
    set(DXCompilerName "libdxcompiler.so")
    set(DXCompilerOutputFolder "lib")
endif()

add_custom_target(CopyDxcompiler ALL
    COMMAND 
        ${CMAKE_COMMAND} -E copy
        ${BINARY_ROOT_DIR}/ThirdParty/DirectXShaderCompiler/${CMAKE_CFG_INTDIR}/${DXCompilerOutputFolder}/${DXCompilerName}
        ${BINARY_ROOT_DIR}/${CMAKE_CFG_INTDIR}/${DXCompilerName}
    COMMENT "Copying dxcompiler..."
)
add_dependencies(CopyDxcompiler dxcompiler)

include_directories(
    ${SOURCE_ROOT_DIR}/ThirdParty/DirectXShaderCompiler/include
    ${SOURCE_ROOT_DIR}/ThirdParty/SPIRV-Cross/include
    ${SOURCE_ROOT_DIR}/ThirdParty/SPIRV-Headers/include
    ${SOURCE_ROOT_DIR}/ThirdParty/SPIRV-Tools/include
    ${SOURCE_ROOT_DIR}/ThirdParty/glslang/
)

target_link_libraries(
    shaderlab
    dxcompiler
    LLVMDxcSupport
    LLVMSupport
    spirv-cross-core
    spirv-cross-glsl
    spirv-cross-hlsl
    spirv-cross-msl
    spirv-cross-util
    SPIRV-Tools
    glslang
    OGLCompiler
    OSDependent
    SPIRV
    SPVRemapper
)

add_dependencies(
    shaderlab
    spirv-cross-core 
    spirv-cross-glsl 
    spirv-cross-hlsl 
    spirv-cross-msl
    CopyDxcompiler
    SPIRV-Tools
    glslang
    OGLCompiler
    OSDependent
    SPIRV
    SPVRemapper
)