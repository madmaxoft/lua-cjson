# CMake script clobbered by madmaxoft to support static linkage, both of this library and of Lua
project(lua-cjson C)
cmake_minimum_required(VERSION 3.0)

option(LUA_CJSON_STATIC_LIB "Build as a static library" OFF)

# Use libc number conversion routines (strtod(), sprintf())
set(FPCONV_SOURCES fpconv.c)

# Handle platforms missing isinf() macro (Eg, some Solaris systems).
include(CheckSymbolExists)
CHECK_SYMBOL_EXISTS(isinf math.h HAVE_ISINF)
if(NOT HAVE_ISINF)
	add_definitions(-DUSE_INTERNAL_ISINF)
endif()

if(APPLE)
	set(CMAKE_SHARED_MODULE_CREATE_C_FLAGS
		"${CMAKE_SHARED_MODULE_CREATE_C_FLAGS} -undefined dynamic_lookup")
endif()

if(WIN32)
	# Windows sprintf()/strtod() handle NaN/inf differently. Not supported.
	add_definitions(-DDISABLE_INVALID_NUMBERS)
else()
	set(_lua_module_dir "${_lua_lib_dir}/lua/5.1")
endif()

if(MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	# add_definitions(-Dinline=__inline)
	# add_definitions(-Dsnprintf=_snprintf)
	add_definitions(-Dstrncasecmp=_strnicmp)
endif()

add_library(cjson STATIC lua_cjson.c strbuf.c ${FPCONV_SOURCES})
target_link_libraries(cjson lua)
