project(mac-dal-plugin)

find_library(AVFOUNDATION AVFoundation)
find_library(COCOA Cocoa)
find_library(COREFOUNDATION CoreFoundation)
find_library(COREMEDIA CoreMedia)
find_library(COREVIDEO CoreVideo)
find_library(COCOA Cocoa)
find_library(COREMEDIAIO CoreMediaIO)
find_library(IOSURFACE IOSurface)
find_library(IOKIT IOKit)

# Possible we could remove osme of these
include_directories(${AVFOUNDATION}
					${COCOA}
					${COREFOUNDATION}
					${COREMEDIA}
					${COREVIDEO}
					${COREMEDIAIO}
					${COCOA}
					${IOSURFACE}
					./
					../common)

set(mac-dal-plugin_HEADERS
	Defines.h
	Logging.h
	PlugInInterface.h
	ObjectStore.h
	PlugIn.h
	Device.h
	Stream.h
	CMSampleBufferUtils.h
	MachClient.h
	TestCard.h
	../common/MachProtocol.h)

set(mac-dal-plugin_SOURCES
	PlugInMain.mm
	PlugInInterface.mm
	ObjectStore.mm
	PlugIn.mm
	Device.mm
	Stream.mm
	CMSampleBufferUtils.mm
	MachClient.mm
	TestCard.mm)

add_library(mac-dal-plugin MODULE
	${mac-dal-plugin_SOURCES}
	${mac-dal-plugin_HEADERS})

set_target_properties(mac-dal-plugin PROPERTIES
	FOLDER "plugins"
	BUNDLE TRUE
	OUTPUT_NAME "obs-mac-virtualcam"
	COMPILE_FLAGS "-std=gnu++14 -stdlib=libc++ -fobjc-arc -fobjc-weak")

target_link_libraries(mac-dal-plugin
	${AVFOUNDATION}
	${COCOA}
	${COREFOUNDATION}
	${COREMEDIA}
	${COREVIDEO}
	${COREMEDIAIO}
	${IOSURFACE}
	${IOKIT})

add_custom_command(TARGET mac-dal-plugin
	POST_BUILD
	COMMAND rm -rf obs-mac-virtualcam.plugin || true
	COMMAND ${CMAKE_COMMAND} -E copy_directory obs-mac-virtualcam.bundle obs-mac-virtualcam.plugin
	COMMENT "Rename bundle to plugin"
)

# Note: Xcode seems to run a command `builtin-infoPlistUtility` to generate the Info.plist, but I'm
# not sure where to find that binary. If we had access to it, the command would look something like:
# builtin-infoPlistUtility ${PROJECT_SOURCE_DIR}/../common/CoreMediaIO/DeviceAbstractionLayer/Devices/Sample/PlugIn/SampleVCam-Info.plist -producttype com.apple.product-type.bundle -expandbuildsettings -platform macosx -o mac-virtualcam.bundle/Contents/Info.plist
# Instead, just copy in one that was already generated from Xcode.
add_custom_command(TARGET mac-dal-plugin
	POST_BUILD
	COMMAND cp ${PROJECT_SOURCE_DIR}/Info.plist obs-mac-virtualcam.plugin/Contents/Info.plist
	DEPENDS {PROJECT_SOURCE_DIR}/Info.plist
	COMMENT "Copy in Info.plist"
)

add_custom_command(TARGET mac-dal-plugin
	POST_BUILD
	COMMAND /usr/bin/codesign --force --deep --sign - --timestamp=none obs-mac-virtualcam.plugin
	COMMENT "Codesign plugin"
)

add_custom_command(TARGET mac-dal-plugin
	POST_BUILD
	COMMAND rm -rf "${OBS_OUTPUT_DIR}/$<CONFIGURATION>/data/obs-mac-virtualcam.plugin" || true
	COMMAND ${CMAKE_COMMAND} -E copy_directory obs-mac-virtualcam.plugin "${OBS_OUTPUT_DIR}/$<CONFIGURATION>/data/obs-mac-virtualcam.plugin"
	COMMENT "Copy plugin to destination"
)
