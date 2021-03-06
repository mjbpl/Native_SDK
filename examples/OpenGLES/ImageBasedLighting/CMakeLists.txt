cmake_minimum_required(VERSION 3.3)

project(OpenGLESImageBasedLighting)

set (OPENGLES_EXAMPLE 1)

#Include common functionality.  (Common.cmake)
# Sets up variables ( ${PROJECT_ARCH}, ${SDK_ROOT},${EXTERNAL_LIB_FOLDER}), sets up some defaults (e.g. CMAKE_BUILD_TYPE),sets up the include folders,
# sets up necessary libraries in EXTRA_LIBS (like dynamic linking, Android libaries, X11/xcb/Wayland etc. for Linux), 
# sets up some compilation flags (e.g. link time code generation, disables some warnings which hit on system files etc.)
include (../../../cmake/Common.cmake)

set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT OpenGLESImageBasedLighting)

set (SRC_FILES OpenGLESImageBasedLighting.cpp)

set (ASSET_FOLDER ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/Assets_OpenGLESImageBasedLighting)

# ASSET FILES (textures, scene files, textures etc) - Packaged with the executable for runtime use
set (ASSET_FILES
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/Default_albedo.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/Default_AO.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/Default_emissive.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/Default_MetalSmooth_converted_metalRoughness.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/Default_normal.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/MonValley/MonValley.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/MonValley/MonValley_baked_lightmap.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/MonValley/Lightmap.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/MonValley/PrefilterEnvMap.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/MonValley/PrefilterL0MipMap.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/MonValley/DiffuseIrradianceMap.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/brdfLUT.pvr
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/damagedHelmet.bin
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/damagedHelmet/damagedHelmet.gltf
	${CMAKE_CURRENT_SOURCE_DIR}/../../assets/ParticleSystem/sphere.pod
	)
list (APPEND SRC_FILES ${ASSET_FILES})
source_group("assets" FILES ${ASSET_FILES})

# TEXTUAL SHADER FILES (OpenGL ES) - Packaged with the executable for runtime use
set (SHADER_FILES
	${CMAKE_CURRENT_SOURCE_DIR}/IrradianceVertShader.vsh
	${CMAKE_CURRENT_SOURCE_DIR}/IrradianceFragShader.fsh
	${CMAKE_CURRENT_SOURCE_DIR}/PBRFragShader.fsh
	${CMAKE_CURRENT_SOURCE_DIR}/PBRVertShader.vsh
	${CMAKE_CURRENT_SOURCE_DIR}/PreFilterFragShader.fsh
	${CMAKE_CURRENT_SOURCE_DIR}/SkyboxFragShader.fsh
	${CMAKE_CURRENT_SOURCE_DIR}/SkyboxVertShader.vsh
	)
list (APPEND SRC_FILES ${SHADER_FILES})
source_group("shaders" FILES ${SHADER_FILES})

set (RESOURCE_FILES ${ASSET_FILES} ${SHADER_FILES} ${SHADER_GENERATED_FILES})

# Adds Windows resouces.rc, macOS plists etc. For macOS/iOS, also set the opengl dynamic libs in the "frameworks" group
add_platform_specific_resource_files("SRC_FILES" "RESOURCE_FILES")

if (WIN32)
	add_executable( OpenGLESImageBasedLighting WIN32 ${SRC_FILES})
elseif (ANDROID)
	add_library( OpenGLESImageBasedLighting SHARED ${SRC_FILES})
elseif (APPLE)
	if (IOS)
		message("Skipping OpenGLESImageBasedLighting : Not supported on iOS")
		return()
	endif()
	add_executable(OpenGLESImageBasedLighting MACOSX_BUNDLE ${SRC_FILES} ${RESOURCE_FILES} ${FRAMEWORK_FILES})
	set_target_properties(OpenGLESImageBasedLighting PROPERTIES MACOSX_BUNDLE_INFO_PLIST  "${INFO_PLIST_FILE}")
	set_target_properties(OpenGLESImageBasedLighting PROPERTIES RESOURCE "${RESOURCE_FILES}")
elseif (UNIX OR QNX)
	add_executable( OpenGLESImageBasedLighting ${SRC_FILES})
endif()

add_rule_copy_assets_to_asset_folder("${RESOURCE_FILES}" "${ASSET_FOLDER}")

# Add the Framework subprojects.
if (NOT ANDROID) # Android builds the Framework libraries by using Gradle to kick CMake (this is an Android thing...)
	add_subdirectory_if_not_already_included(PVRCore ${SDK_ROOT}/framework/PVRCore ${FRAMEWORK_CMAKE_FILES_FOLDER}/PVRCore)
	add_subdirectory_if_not_already_included(PVRAssets ${SDK_ROOT}/framework/PVRAssets ${FRAMEWORK_CMAKE_FILES_FOLDER}/PVRAssets)
	add_subdirectory_if_not_already_included(PVRShell ${SDK_ROOT}/framework/PVRShell ${FRAMEWORK_CMAKE_FILES_FOLDER}/PVRShell)
	add_subdirectory_if_not_already_included(PVRUtilsGles ${SDK_ROOT}/framework/PVRUtils/OpenGLES ${FRAMEWORK_CMAKE_FILES_FOLDER}/PVRUtilsGles)

	add_dependencies(OpenGLESImageBasedLighting PVRCore  PVRAssets  PVRShell  PVRUtilsGles) #... We should still add the dependencies, build or not.
endif()

target_link_libraries(OpenGLESImageBasedLighting
${FRAMEWORK_LIB_FOLDER}/${CMAKE_STATIC_LIBRARY_PREFIX}PVRUtilsGles${CMAKE_STATIC_LIBRARY_SUFFIX}
${FRAMEWORK_LIB_FOLDER}/${CMAKE_STATIC_LIBRARY_PREFIX}PVRShell${CMAKE_STATIC_LIBRARY_SUFFIX}
${FRAMEWORK_LIB_FOLDER}/${CMAKE_STATIC_LIBRARY_PREFIX}PVRAssets${CMAKE_STATIC_LIBRARY_SUFFIX}
${FRAMEWORK_LIB_FOLDER}/${CMAKE_STATIC_LIBRARY_PREFIX}PVRCore${CMAKE_STATIC_LIBRARY_SUFFIX}
${EXTRA_LIBS})

target_compile_definitions(OpenGLESImageBasedLighting PUBLIC $<$<CONFIG:Debug>:DEBUG=1> $<$<NOT:$<CONFIG:Debug>>:RELEASE=1> ) #Defines DEBUG=1 or RELEASE=1

