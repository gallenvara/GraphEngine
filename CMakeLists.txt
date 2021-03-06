CMAKE_MINIMUM_REQUIRED(VERSION 3.0.0)

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
FIND_PACKAGE(Dotnet 2.0 REQUIRED)
INCLUDE(PostBuild)

EXECUTE_PROCESS(
    COMMAND ${DOTNET_EXE} help # to avoid "FIRST RUN EXPERIENCE" leaking into our version string
    OUTPUT_QUIET
    ERROR_QUIET)

EXECUTE_PROCESS(
        COMMAND ${DOTNET_EXE} run -p ${CMAKE_CURRENT_LIST_DIR}/tools/versioning/versioning.csproj
    OUTPUT_VARIABLE TRINITY_VERSION OUTPUT_STRIP_TRAILING_WHITESPACE)
SET(TRINITY_VERSION "2.0.${TRINITY_VERSION}")
MESSAGE("-- GraphEngine version is now ${TRINITY_VERSION}")

PROJECT(GraphEngine VERSION ${TRINITY_VERSION} LANGUAGES CXX)

SET(CMAKE_CXX_STANDARD 14)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_CXX_EXTENSIONS OFF)

SET(XPLAT_LIB_DIR ${CMAKE_CURRENT_LIST_DIR}/lib)
SET_OUTPUT_DIRECTORY(${CMAKE_BINARY_DIR})
DOTNET_REGISTER_LOCAL_REPOSITORY("Graph Engine OSS Local" "${CMAKE_BINARY_DIR}")

IF(WIN32)
    ADD_COMPILE_OPTIONS(/volatile:iso /EHsc /fp:except- /Zi /Gy /W3 /nologo /MP)
    SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /DEBUG:FULL")
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /DEBUG:FULL")
ELSEIF(UNIX)
    ADD_COMPILE_OPTIONS(-fpermissive -Wall -Wno-write-strings -Wno-unknown-pragmas -pthread)
ELSE()
    MESSAGE(FATAL_ERROR "Unsupported platform")
ENDIF()

ADD_SUBDIRECTORY(src/Trinity.C)
ADD_SUBDIRECTORY(src/Trinity.TSL)
ADD_SUBDIRECTORY(src/Trinity.Core)
ADD_SUBDIRECTORY(src/Modules/LIKQ)
ADD_SUBDIRECTORY(src/Modules/GraphEngine.Storage.Composite)
ADD_SUBDIRECTORY(src/Modules/GraphEngine.Client)
ADD_SUBDIRECTORY(src/Modules/GraphEngine.DynamicCluster)

ADD_MSBUILD(
    src/Modules/GraphEngine.ServiceFabric/GraphEngine.ServiceFabric.sln 
    PACKAGE 
        GraphEngine.ServiceFabric.Infrastructure
        GraphEngine.ServiceFabric.Remoting
    DEPENDS
        GraphEngine.Core
        GraphEngine.Client
        GraphEngine.Storage.Composite
        GraphEngine.DynamicCluster
        GraphEngine.Azure.BlobStorage
    X64
    CONFIG RelSkipSFApp
    VERSION ${TRINITY_VERSION}
)

# ADD_DOTNET(
#     src/GraphEngine.DataImporter/GraphEngine.DataImporter.sln
#     PACKAGE GraphEngine.DataImporter
#     DEPENDS GraphEngine.Core)

