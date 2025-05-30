set(project FullSample)
set(folder "RTXDI SDK")

set(sources
	"DebugViz/DebugVizPasses.cpp"
	"DebugViz/DebugVizPasses.h"
	"DebugViz/PackedDataVizPass.cpp"
	"DebugViz/PackedDataVizPass.h"
	"RenderPasses/AccumulationPass.cpp"
	"RenderPasses/AccumulationPass.h"
	"RenderPasses/CompositingPass.cpp"
	"RenderPasses/CompositingPass.h"
	"RenderPasses/ConfidencePass.cpp"
	"RenderPasses/ConfidencePass.h"
	"RenderPasses/FilterGradientsPass.cpp"
	"RenderPasses/FilterGradientsPass.h"
	"RenderPasses/GBufferPass.cpp"
	"RenderPasses/GBufferPass.h"
	"RenderPasses/GenerateMipsPass.cpp"
	"RenderPasses/GenerateMipsPass.h"
	"RenderPasses/GlassPass.cpp"
	"RenderPasses/GlassPass.h"
	"RenderPasses/LightingPasses.cpp"
	"RenderPasses/LightingPasses.h"
	"RenderPasses/PrepareLightsPass.cpp"
	"RenderPasses/PrepareLightsPass.h"
	"RenderPasses/RaytracingPass.cpp"
	"RenderPasses/RaytracingPass.h"
	"RenderPasses/RenderEnvironmentMapPass.cpp"
	"RenderPasses/RenderEnvironmentMapPass.h"
	"RenderPasses/VisualizationPass.cpp"
	"RenderPasses/VisualizationPass.h"
	"AppDefines.h"
	"DLSS-DX12.cpp"
	"DLSS-VK.cpp"
	"DLSS.cpp"
	"DLSS.h"
	"main.cpp"
	"NrdIntegration.cpp"
	"NrdIntegration.h"
	"Profiler.cpp"
	"Profiler.h"
	"ProfilerSections.h"
	"RenderTargets.cpp"
	"RenderTargets.h"
	"RtxdiResources.cpp"
	"RtxdiResources.h"
	"SampleScene.cpp"
	"SampleScene.h"
	"Testing.cpp"
	"Testing.h"
	"UserInterface.cpp"
	"UserInterface.h")

# Organize MSVS filters (the little folders in the solution explorer) to match the folder structure
foreach(source IN LISTS sources)
    get_filename_component(source_path "${source}" PATH)
    string(REPLACE "/" "\\" source_path_msvc "${source_path}")
    source_group("${source_path_msvc}" FILES "${source}")
endforeach()


include(CMakeDependentOption)

cmake_dependent_option(RTXDI_CONSOLE_APP "Build the sample as a console application" OFF WIN32 OFF)

if (RTXDI_CONSOLE_APP)
	add_executable(${project} ${sources})
	target_compile_definitions(${project} PRIVATE IS_CONSOLE_APP=1)
else()
	add_executable(${project} WIN32 ${sources})
endif()

target_link_libraries(${project} donut_core donut_engine donut_app donut_render Rtxdi cxxopts)
add_dependencies(${project} FullSampleShaders)
set_target_properties(${project} PROPERTIES FOLDER ${folder})

if (TARGET NRD)
	target_compile_definitions(${project} PRIVATE WITH_NRD=1)
	target_link_libraries(${project} NRD)
	
	# NRD doesn't add a public include path at this time, work around that
	target_include_directories(${project} PRIVATE "${CMAKE_SOURCE_DIR}/NRD/Include")
endif()

if (TARGET DLSS)
	target_compile_definitions(${project} PRIVATE WITH_DLSS=1)
	target_link_libraries(${project} DLSS)
	add_custom_command(TARGET ${project} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_if_different
		"${DLSS_SHARED_LIBRARY_PATH}"
		"$<TARGET_FILE_DIR:${project}>/")
endif()
