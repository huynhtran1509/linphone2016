############################################################################
# CMakeLists.txt
# Copyright (C) 2014  Belledonne Communications, Grenoble France
#
############################################################################
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#
############################################################################

cmake_minimum_required(VERSION 3.0)
project(mssilk VERSION 1.0.2 LANGUAGES C)


option(ENABLE_SHARED "Build shared library." YES)
option(ENABLE_STATIC "Build static library." YES)


include(GNUInstallDirs)

if(NOT CMAKE_INSTALL_RPATH AND CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})
	message(STATUS "Setting install rpath to ${CMAKE_INSTALL_RPATH}")
endif()

if(LINPHONE_BUILDER_GROUP_EXTERNAL_SOURCE_PATH_BUILDERS)
	include("${EP_ms2_CONFIG_DIR}/Mediastreamer2Config.cmake")
else()
	find_package(Mediastreamer2 REQUIRED)
endif()

set(FLAVOUR "FIX")
file(GLOB SILK_SOURCE_FILES "sdk/SILK_SDK_SRC_v1.0.9/SILK_SDK_SRC_${FLAVOUR}_v1.0.9/src/*.c")


include_directories(
	${MEDIASTREAMER2_INCLUDE_DIRS}
	sdk/SILK_SDK_SRC_v1.0.9/SILK_SDK_SRC_${FLAVOUR}_v1.0.9/interface
)

add_definitions(-DVERSION=\"${PROJECT_VERSION}\")


set(LIBS ${MEDIASTREAMER2_LIBRARIES})

set(SOURCE_FILES
	${SILK_SOURCE_FILES}
	silk_dec.c
	silk_enc.c
)

if(ENABLE_STATIC)
	add_library(mssilk-static STATIC ${SOURCE_FILES})
	set_target_properties(mssilk-static PROPERTIES OUTPUT_NAME mssilk)
	target_link_libraries(mssilk-static ${LIBS})
	install(TARGETS mssilk-static
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
	)
endif()
if(ENABLE_SHARED)
	add_library(mssilk MODULE ${SOURCE_FILES})
	target_link_libraries(mssilk ${LIBS})
	if(MSVC)
		if(CMAKE_BUILD_TYPE STREQUAL "Debug")
			install(FILES ${CMAKE_CURRENT_BINARY_DIR}/Debug/libmssilk.pdb
				DESTINATION ${CMAKE_INSTALL_BINDIR}
				PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
			)
		endif()
		set_target_properties(mssilk PROPERTIES PREFIX "lib")
	endif()
	install(TARGETS mssilk
		RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
	)
endif()
