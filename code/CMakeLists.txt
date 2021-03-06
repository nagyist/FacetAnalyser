
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

if(POLICY CMP0023)
  message(STATUS "Explicitly setting policy CMP0023 to OLD (needed by PV 4.4)")
  cmake_policy(SET CMP0023 OLD)
endif(POLICY CMP0023)

#Change PROJECT_NAME to the name of your project
PROJECT(FacetAnalyser)

#The following lines are required to use Dart
ENABLE_TESTING()
INCLUDE(CTest)


FIND_PACKAGE(ParaView REQUIRED)
INCLUDE(${PARAVIEW_USE_FILE})

FIND_PACKAGE(ITK REQUIRED)
INCLUDE(${USE_ITK_FILE})

INCLUDE_DIRECTORIES(${ITK_INCLUDE_DIRS})
LINK_DIRECTORIES(${ITK_LIBRARY_DIRS})



IF(BUILD_TESTING)

## build test programs
ADD_EXECUTABLE(FacetAnalyserTest
  FacetAnalyser.cxx vtkGaussianSplatterExtended.cxx
  FacetAnalyserTest.cxx)
TARGET_LINK_LIBRARIES(FacetAnalyserTest ${ITK_LIBRARIES} ${VTK_LIBRARIES})

## run tests
ADD_TEST(NAME basicTest01 WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/testing/ COMMAND FacetAnalyserTest 8faced-rhombic-dodecahedron_twinned_simplified.vtk)

IF(BUILD_PLUGIN)
  ADD_TEST(NAME basicPluginTest01 WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/demos/ COMMAND ${ParaView_DIR}/bin/pvpython ${PROJECT_SOURCE_DIR}/scripts/pvsm2webgl.py -i 8faced-rhombic-dodecahedron_twinned_simp+def.pvsm -o ${PROJECT_SOURCE_DIR}/docs/demos/WebGL/8faced-rhombic-dodecahedron_twinned_simp+def.webgl -p ${CMAKE_CURRENT_BINARY_DIR}/libFacetAnalyser.so )

  ADD_TEST(NAME basicPluginTest02 WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/demos/ COMMAND ${ParaView_DIR}/bin/pvpython ${PROJECT_SOURCE_DIR}/scripts/pvsm2x3d.py -i 8faced-rhombic-dodecahedron_twinned_simp+def.pvsm -o ../docs/demos/X3D/8faced-rhombic-dodecahedron_twinned_simp+def.x3d -p ${CMAKE_CURRENT_BINARY_DIR}/libFacetAnalyser.so ) # relative path as it is stored in X3D

  set_tests_properties(basicPluginTest01 basicPluginTest02 PROPERTIES PASS_REGULAR_EXPRESSION "Export succeded")    ## regex-check because script itself fails with: Inconsistency detected by ld.so: dl-close.c: _dl_close: Assertion `map->l_init_called' failed!

ENDIF(BUILD_PLUGIN)

IF(BUILD_EXAMPLE)

  ADD_TEST( NAME basicExampleTest01 WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/demos/ COMMAND FacetAnalyserCLI 8faced-rhombic-dodecahedron_twinned_010_deformed_wss.vtp 51 15 0.2 0.001 2 8faced-rhombic-dodecahedron_twinned_010_deformed_wss_FA.vtp )

ENDIF(BUILD_EXAMPLE)

ENDIF(BUILD_TESTING)



OPTION( BUILD_PLUGIN
        "Build ParaView plugin"
        ON )

IF(BUILD_PLUGIN)

# Use the PLUGIN macro to create a plugin.
ADD_PARAVIEW_PLUGIN(FacetAnalyser "1.0"
   SERVER_MANAGER_SOURCES FacetAnalyser.cxx vtkGaussianSplatterExtended.cxx
   SERVER_MANAGER_XML FacetAnalyser.xml)

TARGET_LINK_LIBRARIES(FacetAnalyser ${ITK_LIBRARIES})

INSTALL(TARGETS FacetAnalyser LIBRARY DESTINATION . )

ENDIF(BUILD_PLUGIN)



OPTION( BUILD_EXAMPLE
        "Build example"
        OFF )

IF( BUILD_EXAMPLE )
# EXAMPLE ---------------------------------------------------------

ADD_EXECUTABLE(FacetAnalyserCLI
  FacetAnalyser.cxx vtkGaussianSplatterExtended.cxx
  FacetAnalyserCLI.cxx)
TARGET_LINK_LIBRARIES(FacetAnalyserCLI ${ITK_LIBRARIES} ${VTK_LIBRARIES})

#------------------------------------------------------------------
ENDIF( BUILD_EXAMPLE )

