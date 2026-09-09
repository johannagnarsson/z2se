# minimum cmake version
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(z2se VERSION 1.2)

# only make release builds
set(CMAKE_CONFIGURATION_TYPES "Release")

# create config file
configure_file(config.h.in config.h)
add_definitions(-DHAVE_CONFIG_H)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# enable compiler warnings
if(MSVC)
  add_compile_options(/W4 /WX /wd4458)
else()
  add_compile_options(-Wall -Wextra -pedantic -Werror)
endif()

# setup the module path
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}")

# locate and add wxWidgets
if(WIN32)
  set(wxWidgets_ROOT_DIR "C:/wxWidgets/3.1.2/release-static")
endif()

find_package(wxWidgets COMPONENTS core base adv html xml xrc REQUIRED)
include( "${wxWidgets_USE_FILE}" )
link_libraries(${wxWidgets_LIBRARIES})

# add include directories
include_directories("${PROJECT_BINARY_DIR}" "source")

# compile xrc resource file if wxrc is available
find_program(WXRC_EXECUTABLE wxrc)
if(WXRC_EXECUTABLE)
  add_custom_command(
    OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/source/res/resource.cpp"
    COMMAND ${WXRC_EXECUTABLE} -c -o resource.cpp z2se.xrc
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/source/res"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/source/res/z2se.xrc"
    COMMENT "Compiling XRC resources with wxrc..."
  )
endif()

# add the sources
set(SRCS source/GenApp.cc source/GenApp.hh
  source/model/SaveSlot.cc source/model/SaveSlot.hh
  source/model/SRAMFile.cc source/model/SRAMFile.hh
  source/view/FileDropTarget.cc source/view/FileDropTarget.hh
  source/view/MainFrame.cc source/view/MainFrame.hh
  source/res/resource.cpp)

# add the resource file on Windows
if(WIN32)
  set(SRCS ${SRCS} source/res/windows.rc)
endif(WIN32)

# add the icon file for macOS
set(MACOSX_BUNDLE_ICON_FILE z2se.icns)

set(APP_ICON "${PROJECT_SOURCE_DIR}/source/res/z2se.icns")
set_source_files_properties("${APP_ICON}" PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")

# add the executable
add_executable(${PROJECT_NAME} WIN32 MACOSX_BUNDLE "${APP_ICON}" ${SRCS})
