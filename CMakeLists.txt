cmake_minimum_required (VERSION 2.8)

if (POLICY CMP0054)
  cmake_policy(PUSH)
  cmake_policy(SET CMP0054 OLD)
endif()
project (PhreeqcRM CXX C)
if (POLICY CMP0054)
  cmake_policy(POP)
endif()

# check if this is the root project
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
  set(STANDALONE_BUILD 1)
else()
  set(STANDALONE_BUILD 0)
endif()

# enable swig java
option (SWIG_ENABLE_JAVA "Generate java wrappers using SWIG" OFF)

# enable swig python
option (SWIG_ENABLE_PYTHON "Generate python wrappers using SWIG" OFF)

# enable swig ruby
if (NOT MSVC)
  option (SWIG_ENABLE_RUBY "Generate ruby wrappers using SWIG" OFF)
endif()

# swig
if (SWIG_ENABLE_JAVA OR SWIG_ENABLE_PYTHON OR SWIG_ENABLE_RUBY)
  find_package(SWIG REQUIRED)
  include(${SWIG_USE_FILE})
endif()

# overide docdir on windows
if(WIN32 AND NOT CMAKE_INSTALL_DOCDIR)
  set(CMAKE_INSTALL_DOCDIR "" CACHE PATH "documentation root (doc)")
  set(CMAKE_INSTALL_DOCDIR "doc")
endif()

option(PHREEQCRM_FORTRAN_TESTING "Build Fortran test" OFF)
if (PHREEQCRM_FORTRAN_TESTING)
  enable_language(Fortran)
endif()

include(GNUInstallDirs)

# set examples directory
if (WIN32)
  set(EXAMPLES_DIR examples)
else()
  set(EXAMPLES_DIR ${CMAKE_INSTALL_DOCDIR}/examples)
endif()

# iphreeqc defs
ADD_DEFINITIONS(-DSWIG_SHARED_OBJ)
ADD_DEFINITIONS(-DUSE_PHRQ_ALLOC)

# Disable deprecation warnings for standard C and STL functions in VS2005
# and later
IF(MSVC_VERSION EQUAL 1400 OR MSVC_VERSION GREATER 1400)
  ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE)
  ADD_DEFINITIONS(-D_SCL_SECURE_NO_WARNINGS)
ENDIF(MSVC_VERSION EQUAL 1400 OR MSVC_VERSION GREATER 1400) 

# check for zlib
if (NOT PHREEQCRM_EXTERNALLY_CONFIGURED)
  if(NOT ZLIB_ROOT)
    if(EXISTS $ENV{ZLIB_ROOT})
      set(ZLIB_ROOT $ENV{ZLIB_ROOT})
    endif()
  endif()
  option(PHREEQCRM_USE_ZLIB "Use zlib compression for dump files" OFF)
  if (PHREEQCRM_USE_ZLIB)
    find_package(ZLIB REQUIRED)
  endif()
endif()

if (PHREEQCRM_USE_ZLIB AND ZLIB_FOUND)
  ADD_DEFINITIONS(-DUSE_GZ)
  include_directories(${ZLIB_INCLUDE_DIRS})
endif()

# check for openmp
find_package(OpenMP)

# check for MPI
option(PHREEQCRM_BUILD_MPI "Build MPI configuration" "OFF")
if (PHREEQCRM_BUILD_MPI)
  if (POLICY CMP0054)
    cmake_policy(SET CMP0054 OLD)
  endif()
  find_package(MPI)
  if (POLICY CMP0054)
    cmake_policy(SET CMP0054 NEW)
  endif()
endif (PHREEQCRM_BUILD_MPI)


SET(PhreeqcRM_SOURCES
src/IPhreeqcPhast/IPhreeqc/CSelectedOutput.cpp
src/IPhreeqcPhast/IPhreeqc/CSelectedOutput.hxx
src/IPhreeqcPhast/IPhreeqc/CVar.hxx
src/IPhreeqcPhast/IPhreeqc/Debug.h
src/IPhreeqcPhast/IPhreeqc/ErrorReporter.hxx
src/IPhreeqcPhast/IPhreeqc/IPhreeqc.cpp
src/IPhreeqcPhast/IPhreeqc/IPhreeqc.h
src/IPhreeqcPhast/IPhreeqc/IPhreeqc.hpp
src/IPhreeqcPhast/IPhreeqc/IPhreeqc_interface_F.cpp
src/IPhreeqcPhast/IPhreeqc/IPhreeqc_interface_F.h
src/IPhreeqcPhast/IPhreeqc/IPhreeqcCallbacks.h
src/IPhreeqcPhast/IPhreeqc/IPhreeqcLib.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/advection.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/basicsubs.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ChartHandler.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ChartHandler.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ChartObject.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ChartObject.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cl1.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/Parser.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/Parser.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/PHRQ_base.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/PHRQ_base.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/PHRQ_io.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/PHRQ_io.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/phrqtype.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/Utils.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/Utils.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/CurveObject.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/CurveObject.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cvdense.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cvdense.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cvode.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cvode.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cxxKinetics.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cxxKinetics.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cxxMix.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/cxxMix.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/dense.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/dense.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/dumper.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/dumper.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/dw.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Exchange.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Exchange.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ExchComp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ExchComp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/GasComp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/GasComp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/gases.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/GasPhase.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/GasPhase.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/global_structures.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/input.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/integrate.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/inverse.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ISolution.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ISolution.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ISolutionComp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ISolutionComp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/isotopes.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/kinetics.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/KineticsComp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/KineticsComp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/mainsubs.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/model.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/NA.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/NameDouble.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/NameDouble.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/NumKeyword.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/NumKeyword.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/nvector.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/nvector.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/nvector_serial.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/nvector_serial.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/parse.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PBasic.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PBasic.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/phqalloc.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/phqalloc.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Phreeqc.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Phreeqc.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PhreeqcKeywords/Keywords.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PhreeqcKeywords/Keywords.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PHRQ_io_output.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/pitzer.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/pitzer_structures.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PPassemblage.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PPassemblage.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PPassemblageComp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PPassemblageComp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/prep.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Pressure.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Pressure.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/print.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Reaction.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Reaction.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/read.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/ReadClass.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/readtr.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/runner.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/runner.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SelectedOutput.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SelectedOutput.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/sit.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/smalldense.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/smalldense.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Solution.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Solution.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SolutionIsotope.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SolutionIsotope.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/spread.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SS.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SS.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SSassemblage.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SSassemblage.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SScomp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SScomp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/step.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/StorageBin.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/StorageBin.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/StorageBinList.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/StorageBinList.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/structures.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/sundialsmath.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/sundialsmath.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/sundialstypes.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Surface.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Surface.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SurfaceCharge.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SurfaceCharge.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SurfaceComp.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/SurfaceComp.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/System.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/System.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/tally.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Temperature.cxx
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Temperature.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/tidy.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/transport.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Use.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/Use.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/UserPunch.cpp
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/UserPunch.h
src/IPhreeqcPhast/IPhreeqc/phreeqcpp/utilities.cpp
src/IPhreeqcPhast/IPhreeqc/thread.h
src/IPhreeqcPhast/IPhreeqc/Var.c
src/IPhreeqcPhast/IPhreeqc/Var.h
src/IPhreeqcPhast/IPhreeqc/Version.h
src/IPhreeqcPhast/IPhreeqcPhast.cxx
src/IPhreeqcPhast/IPhreeqcPhast.h
src/IPhreeqcPhast/IPhreeqcPhastLib.cpp
src/IPhreeqcPhast/IPhreeqcPhastLib.h
src/IrmResult.h
src/PhreeqcRM.cpp
src/PhreeqcRM.h
src/RM_interface_C.cpp
src/RM_interface_C.h
src/RM_interface_F.cpp
src/RM_interface_F.h
)

# setup include directories
include_directories("${PROJECT_SOURCE_DIR}/src")
include_directories("${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast")
include_directories("${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc")
include_directories("${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp")
include_directories("${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common")
include_directories("${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PhreeqcKeywords")

# dll/shared objects
OPTION(BUILD_SHARED_LIBS "Build Shared Libraries" OFF)
SET(LIB_TYPE STATIC)
if (BUILD_SHARED_LIBS)
  SET(LIB_TYPE SHARED)
endif()
add_library(PhreeqcRM ${LIB_TYPE} ${PhreeqcRM_SOURCES})
set_target_properties(PhreeqcRM PROPERTIES DEBUG_POSTFIX "d")

# At least with MSMPI PhreeqcRM.dll requires MPI_CXX_LIBRARIES to link
if (MSVC AND BUILD_SHARED_LIBS AND PHREEQCRM_BUILD_MPI)
  target_link_libraries(PhreeqcRM ${MPI_CXX_LIBRARIES})
endif()

if (MSVC AND BUILD_SHARED_LIBS AND PHREEQCRM_USE_ZLIB AND ZLIB_FOUND)
  target_link_libraries(PhreeqcRM ${ZLIB_LIBRARIES})
endif()

SET(PhreeqcRM_Headers
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/IPhreeqc.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/IPhreeqc.hpp
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/IPhreeqcCallbacks.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/Parser.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/PHRQ_base.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/PHRQ_io.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/common/phrqtype.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/NameDouble.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/PhreeqcKeywords/Keywords.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/StorageBin.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/phreeqcpp/System.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/Var.h
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqcPhast.h
${PROJECT_SOURCE_DIR}/src/IrmResult.h
${PROJECT_SOURCE_DIR}/src/PhreeqcRM.h
${PROJECT_SOURCE_DIR}/src/RM_interface_C.h
)

SET(PhreeqcRM_SRC
${PROJECT_SOURCE_DIR}/src/RM_interface.F90
${PROJECT_SOURCE_DIR}/src/IPhreeqcPhast/IPhreeqc/IPhreeqc_interface.F90
)

if (PHREEQCRM_BUILD_MPI)
  ADD_DEFINITIONS(-DUSE_MPI)
  include_directories(${MPI_CXX_INCLUDE_PATH})
  if (PHREEQCRM_FORTRAN_TESTING AND MPI_Fortran_INCLUDE_PATH)
    if("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
      include_directories(${MPI_Fortran_INCLUDE_PATH}/x64)
      include_directories(${MPI_Fortran_INCLUDE_PATH}/amd64)
    else()
      include_directories(${MPI_Fortran_INCLUDE_PATH}/x86)
      include_directories(${MPI_Fortran_INCLUDE_PATH}/i386)
    endif()
  endif()
  set(LINK_LIBS ${LINK_LIBS} ${MPI_CXX_LIBRARIES})
  if (PHREEQCRM_FORTRAN_TESTING AND MPI_Fortran_LIBRARIES MATCHES "msmpi.lib$")
    STRING(REGEX REPLACE "msmpi.lib$" "msmpifec.lib" EXTRA_FORTRAN_LIB ${MPI_Fortran_LIBRARIES})
    SET(LINK_LIBS ${LINK_LIBS} ${EXTRA_FORTRAN_LIB})
  endif ()  
else()
  if (OPENMP_FOUND)
    ADD_DEFINITIONS(-DUSE_OPENMP)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")
    if (PHREEQCRM_FORTRAN_TESTING)
      set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${OpenMP_Fortran_FLAGS}")
    endif()
  endif()
endif()  

# install
install(TARGETS PhreeqcRM DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES ${PhreeqcRM_Headers} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${PhreeqcRM_SRC} DESTINATION src)

if (STANDALONE_BUILD EQUAL 1 AND MSVC)
  # PHREEQCRM_STATIC_RUNTIME (VCOMP???.dll) cannot be made static
  option (PHREEQCRM_STATIC_RUNTIME "Build with a static runtime (excluding VCOMP???.DLL)" OFF)  
  if (PHREEQCRM_STATIC_RUNTIME)    
    # compile with static runtime
    
    set(CompilerFlags
    CMAKE_CXX_FLAGS
    CMAKE_CXX_FLAGS_DEBUG
    CMAKE_CXX_FLAGS_RELEASE
    CMAKE_CXX_FLAGS_MINSIZEREL
    CMAKE_CXX_FLAGS_RELWITHDEBINFO
    CMAKE_C_FLAGS
    CMAKE_C_FLAGS_DEBUG
    CMAKE_C_FLAGS_RELEASE
    CMAKE_C_FLAGS_MINSIZEREL
    CMAKE_C_FLAGS_RELWITHDEBINFO
    )

    foreach(CompilerFlag ${CompilerFlags})
      if(${CompilerFlag} MATCHES "/MD")
        string(REGEX REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
      endif()
    endforeach()
  
    set(FortranCompilerFlags
    CMAKE_Fortran_FLAGS
    )
  
    foreach(FortranCompilerFlags ${FortranCompilerFlags})
      if(${FortranCompilerFlags} MATCHES "/libs:dll")
        string(REGEX REPLACE "/libs:dll" "/libs:static" ${FortranCompilerFlags} "${${FortranCompilerFlags}}")
      endif()  
    endforeach()
  endif()
endif()

if (STANDALONE_BUILD EQUAL 1)
  # swig
  if (SWIG_FOUND)
    set(CMAKE_SWIG_OUTDIR "${CMAKE_BINARY_DIR}/swig")
    set_source_files_properties(src/PhreeqcRM.i PROPERTIES CPLUSPLUS ON)
    set_source_files_properties(src/PhreeqcRM.i PROPERTIES SWIG_FLAGS "-includeall")
    set_source_files_properties(src/PhreeqcRM.i PROPERTIES SWIG_FLAGS "-Wall")
    file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/src)
    
    # swig python
    if (SWIG_ENABLE_PYTHON)
      find_package(PythonLibs)

      swig_add_module(phreeqcrm python src/PhreeqcRM.i ${PhreeqcRM_SOURCES})
      swig_link_libraries(phreeqcrm ${PYTHON_LIBRARIES})
      
      target_include_directories(${SWIG_MODULE_phreeqcrm_REAL_NAME} BEFORE PUBLIC ${PYTHON_INCLUDE_PATH})
      target_compile_definitions(${SWIG_MODULE_phreeqcrm_REAL_NAME} PUBLIC NESTED_UNION_NOT_SUPPORTED SWIG_PHREEQCRM)
      target_compile_definitions(${SWIG_MODULE_phreeqcrm_REAL_NAME} PUBLIC SWIG_IPHREEQC)

      install(TARGETS ${SWIG_MODULE_phreeqcrm_REAL_NAME} DESTINATION ${EXAMPLES_DIR}/swig/python)
    endif()

    # swig java  
    if (SWIG_ENABLE_JAVA)
      find_package(JNI)
      
      swig_add_module(phreeqcrm_java java src/PhreeqcRM.i ${PhreeqcRM_SOURCES})
      swig_link_libraries(${SWIG_MODULE_phreeqcrm_java_REAL_NAME} ${JNI_LIBRARIES})
      
      target_include_directories(${SWIG_MODULE_phreeqcrm_java_REAL_NAME} BEFORE PUBLIC ${JAVA_INCLUDE_PATH} ${JAVA_INCLUDE_PATH2})      
      target_compile_definitions(${SWIG_MODULE_phreeqcrm_java_REAL_NAME} PUBLIC SWIG_PHREEQCRM)
      target_compile_definitions(${SWIG_MODULE_phreeqcrm_java_REAL_NAME} PUBLIC SWIG_IPHREEQC)
      
      install(TARGETS ${SWIG_MODULE_phreeqcrm_java_REAL_NAME} DESTINATION ${EXAMPLES_DIR}/swig/java)
    endif()
    
    # swig ruby  
    if (SWIG_ENABLE_RUBY)
      find_package(Ruby)
    
      swig_add_module(phreeqcrm_ruby ruby src/PhreeqcRM.i ${PhreeqcRM_SOURCES})
      swig_link_libraries(phreeqcrm_ruby ${RUBY_LIBRARY})
      
      target_include_directories(${SWIG_MODULE_phreeqcrm_ruby_REAL_NAME} BEFORE PUBLIC ${RUBY_INCLUDE_PATH})
      target_compile_definitions(${SWIG_MODULE_phreeqcrm_ruby_REAL_NAME} PUBLIC NESTED_UNION_NOT_SUPPORTED SWIG_PHREEQCRM)
      target_compile_definitions(${SWIG_MODULE_phreeqcrm_ruby_REAL_NAME} PUBLIC SWIG_IPHREEQC)

      # must not prefix ruby .so on linux
      set_target_properties(phreeqcrm_ruby PROPERTIES PREFIX "" OUTPUT_NAME "phreeqcrm_ruby")      
      
      install(TARGETS ${SWIG_MODULE_phreeqcrm_ruby_REAL_NAME} DESTINATION ${EXAMPLES_DIR}/swig/ruby)
    endif()
  endif()

  # tests
  enable_testing()

  # subdirs
  add_subdirectory(database)
  add_subdirectory(doc)
  add_subdirectory(Doxygen)
  add_subdirectory(Tests)
  if (SWIG_FOUND)
    add_subdirectory(examples)
  endif()
endif()

# Turn off Incremental Linking for Debug and RelWithDebInfo
if (MSVC)
  foreach(FLAG_TYPE EXE MODULE SHARED)
    # Debug
    string(REPLACE "/INCREMENTAL:YES " "/INCREMENTAL:NO " CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG})
    string(REGEX REPLACE " /INCREMENTAL:YES$" " /INCREMENTAL:NO" CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG})
    string(REPLACE "/INCREMENTAL " "/INCREMENTAL:NO " CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG})
    string(REGEX REPLACE " /INCREMENTAL$" " /INCREMENTAL:NO" CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG})
    set(CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG
        "${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_DEBUG}" CACHE STRING 
        "Flags used by the shared libraries linker during Debug builds."
        FORCE)

    # RelWithDebInfo
    string(REPLACE "/INCREMENTAL:YES " "/INCREMENTAL:NO " CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO})
    string(REGEX REPLACE " /INCREMENTAL:YES$" " /INCREMENTAL:NO" CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO})
    string(REPLACE "/INCREMENTAL " "/INCREMENTAL:NO " CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO})
    string(REGEX REPLACE " /INCREMENTAL$" " /INCREMENTAL:NO" CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO ${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO})
    set(CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO
        "${CMAKE_${FLAG_TYPE}_LINKER_FLAGS_RELWITHDEBINFO}" CACHE STRING 
        "Flags used by the shared libraries linker during Release with Debug Info builds."
        FORCE)
  endforeach()
endif()
