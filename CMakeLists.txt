cmake_minimum_required(VERSION 3.10.3)

project(Elektron-SDK-BinaryPack 
						VERSION 
							1.3.0.0
		)

set(RCDEV_NORMALIZE_COMPILER_VERERSION ON)
if (EXTERNAL_MODULE_PATH)
	string(REPLACE "|" ";" _extern_modules "${EXTERNAL_MODULE_PATH}")
	list(APPEND CMAKE_MODULE_PATH "${_extern_modules}")
endif()

set(Elektron-SDK-BinaryPack_CMAKE_DIR "${Elektron-SDK-BinaryPack_SOURCE_DIR}/CMake")
list(APPEND CMAKE_MODULE_PATH "${Elektron-SDK-BinaryPack_CMAKE_DIR}")
set(CMAKE_DEBUG_TARGET_PROPERTIES TRUE)

include(rcdevEnvironment)
include(rcdevSupport)

DEBUG_PRINT(Elektron-SDK-BinaryPack_OUTPUT_DIRECTORY)

if (Elektron-SDK-BinaryPack_OUTPUT_DIRECTORY)
	set(_root_dir "${Elektron-SDK-BinaryPack_OUTPUT_DIRECTORY}")
else()
	set(_root_dir "${CMAKE_SOURCE_DIR}")
endif()

# Because of some legacy static path definitions, 
# the suffix of the root paths for libs and includes are
# not dynamicly defined.  Future releases will hope to 
# move to having fully dynamic defined locations for libs
# and header paths within the bianry pack.
# The default values are relative, well-known paths.
set(_dacs_include_root "Cpp-C/Eta/Utils/Dacs/Include")
set(_ansi_include_root "Cpp-C/Eta/Utils/Ansi/Include")
set(_vacache_include_root "Cpp-C/Eta/Include/Cache")
set(_dacs_lib_root "${_root_dir}/Cpp-C/Eta/Utils/Dacs/Libs")
set(_ansi_lib_root "${_root_dir}/Cpp-C/Eta/Utils/Ansi/Libs")
set(_vacache_lib_root "${_root_dir}/Cpp-C/Eta/Libs")

set(_header_suffix ${_dacs_include_root})
find_path(Elektron-SDK-BinaryPack_dacs_INCLUDE_DIRS 
            NAMES dacs_lib.h
            PATHS ${_header_suffix}
            DOC "Dacs library Include path"
        )
set(_header_suffix ${_ansi_include_root})
find_path(Elektron-SDK-BinaryPack_ansi_INCLUDE_DIRS 
            NAMES ansi/ansi_int.h dev/platform.h
            PATHS ${_header_suffix}
            DOC "Ansi library Include path"
            #PATH_SUFFIXES ansi dev
        )
set(_header_suffix ${_vacache_include_root})
find_path(Elektron-SDK-BinaryPack_rsslVACache_INCLUDE_DIRS 
            NAMES rtr/rsslCacheDefs.h
            PATHS ${_header_suffix}
            DOC "Rssl Value Add Cache library Include path"
            #PATH_SUFFIXES rtr
        )

set(Elektron-SDK-BinaryPack_INCLUDE_DIRS	"${Elektron-SDK-BinaryPack_dacs_INCLUDE_DIRS}"
											"${Elektron-SDK-BinaryPack_ansi_INCLUDE_DIRS}"
											"${Elektron-SDK-BinaryPack_rsslVACache_INCLUDE_DIRS}"
											"${Elektron-SDK-BinaryPack_DIR}"
											 )
mark_as_advanced(Elektron-SDK-BinaryPack_DIR
                 Elektron-SDK-BinaryPack_dacs_INCLUDE_DIRS
                 Elektron-SDK-BinaryPack_ansi_INCLUDE_DIRS
                 Elektron-SDK-BinaryPack_rsslVACache_INCLUDE_DIRS
                )

set(_plat_suffix)
set(_plat_suffix2)
if ( CMAKE_HOST_UNIX )
	# Run the LinuxSoLink to generate binary version and .so links
	execute_process(COMMAND ./LinuxSoLink
				RESULT_VARIABLE _ret_val
				WORKING_DIRECTORY "${Elektron-SDK-BinaryPack_SOURCE_DIR}"
				)
    # Because of some legacy static path definitions, some paths
    # have GCC and/or OL6 in place of GNU and/or RHEL6.  So in order to support
    # old past haunts, hints will be added in case past wrongs
    # are corrected
    rcdev_get_platform_suffix(_plat_suffix)
    if ( RCDEV_HOST_SYSTEM_FLAVOR_U MATCHES "ORACLE" 
        AND RCDEV_HOST_SYSTEM_FLAVOR_REL EQUAL 6 )
        rcdev_get_platform_suffix(_plat_suffix2 "gcc" "rhel")
    elseif (((RCDEV_HOST_SYSTEM_FLAVOR_U MATCHES "CENTOS") OR
			(RCDEV_HOST_SYSTEM_FLAVOR_U MATCHES "REDHATLINUX")) AND
            (RCDEV_HOST_SYSTEM_FLAVOR_REL EQUAL 7) )
        rcdev_get_platform_suffix(_plat_suffix2 "gcc" "centOS")
    else()
        rcdev_get_platform_suffix(_plat_suffix2 "gcc" )
    endif()

    set(_hints "${_dacs_lib_root}/${_plat_suffix}/Optimized"
               "${_dacs_lib_root}/${_plat_suffix2}/Optimized")
    DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for dacs in :"
						"${_hints}")
    find_library(Elektron-SDK-BinaryPack_dacs_LIBRARY dacs
                    PATHS ${_hints}
                    DOC "DACS library from Elektron-SDK-BinaryPack"
                    NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

    set(_hints "${_ansi_lib_root}/${_plat_suffix}/Optimized"
               "${_ansi_lib_root}/${_plat_suffix2}/Optimized")
    DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for ansi in : "
							"${_hints}")
    find_library(Elektron-SDK-BinaryPack_ansi_LIBRARY ansi 
                    PATHS ${_hints}
                    DOC "Ansi library from Elektron-SDK-BinaryPack"
                    NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

    # For UNIX, when not crosscompiling, we only need to handle the current
    # build config type and not all possible config types
    set(_hints "${_vacache_lib_root}/${_plat_suffix}/${CMAKE_BUILD_TYPE}"
               "${_vacache_lib_root}/${_plat_suffix2}/${CMAKE_BUILD_TYPE}")
    DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for rsslVACache in : ${_hints}")
    find_library(Elektron-SDK-BinaryPack_rsslVACache_LIBRARY rsslVACache 
                    PATHS ${_hints}
                    DOC "Rssl Value Add Cache library from Elektron-SDK-BinaryPack"
                    NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

    # For UNIX, when not crosscompiling, we only need to handle the current
    # build config type and not all possible config types
    set(_hints "${_vacache_lib_root}/${_plat_suffix}/${CMAKE_BUILD_TYPE}/Shared"
               "${_vacache_lib_root}/${_plat_suffix2}/${CMAKE_BUILD_TYPE}/Shared")
    DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for rsslVACache_shared in : "
							"${_hints}")
    find_library(Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY rsslVACache 
                    PATHS ${_hints} 
                    DOC "Rssl Value Add Cache shared library from Elektron-SDK-BinaryPack"
                    NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

    DEBUG_PRINT(::MESSAGE "
    IN ${CMAKE_CURRENT_LIST_DIR}
	Elektron-SDK-BinaryPack_
        dacs_LIBRARY               : ${Elektron-SDK-BinaryPack_dacs_LIBRARY}
        ansi_LIBRARY               : ${Elektron-SDK-BinaryPack_ansi_LIBRARY}
        rsslVACache_static_LIBRARY : ${Elektron-SDK-BinaryPack_rsslVACache_LIBRARY}
        rsslVACache_shared_LIBRARY : ${Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY}
    ")
    mark_as_advanced(Elektron-SDK-BinaryPack_dacs_LIBRARY
                     Elektron-SDK-BinaryPack_ansi_LIBRARY
                     Elektron-SDK-BinaryPack_rsslVACache_LIBRARY
                     Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY
                    )
    string(TOUPPER ${CMAKE_BUILD_TYPE} _CONFIG_TYPE_U)

else() # if (WIN32)

    rcdev_get_platform_suffix(_plat_suffix)

    list(APPEND _config_type_list Debug_MDd Release_MD)
    foreach(_cType IN LISTS _config_type_list)

        string(TOUPPER ${_cType} _cType_U)

        set(_hint "${_dacs_lib_root}/${_plat_suffix}/${_cType}")
        DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for dacs in : "
								"${_hint}")
        find_library(Elektron-SDK-BinaryPack_dacs_LIBRARY_${_cType_U} 
                        NAMES dacs libdacs 
                        HINTS ${_hint}
                        DOC "DACS ${_cType} library from Elektron-SDK-BinaryPack"
                        NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

        set(_hint "${_ansi_lib_root}/${_plat_suffix}/${_cType}")
        DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for ansi in : "
								"${_hint}")
        find_library(Elektron-SDK-BinaryPack_ansi_LIBRARY_${_cType_U} 
                        NAMES ansi libansi 
                        HINTS ${_hint}
                        DOC "Ansi ${_cType} library from Elektron-SDK-BinaryPack"
                        NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

        set(_hint "${_vacache_lib_root}/${_plat_suffix}/${_cType}")
        DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for rsslVACache in : ${_hint}")
        find_library(Elektron-SDK-BinaryPack_rsslVACache_LIBRARY_${_cType_U} 
                        NAMES rsslVACache librsslVACache 
                        HINTS ${_hint}
                        DOC "Rssl Value Add Cache ${_cType} library from Elektron-SDK-BinaryPack"
                        NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

        set(_hint "${_vacache_lib_root}/${_plat_suffix}/${_cType}/Shared")
        DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for rsslVACache_shared in : "
								"${_hint}")
        find_library(Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY_${_cType_U}
                        NAMES rsslVACache librsslVACache NAMES_PER_DIR
                        HINTS ${_hint} 
                        DOC "Rssl Value Add Cache shared ${_cType} library from Elektron-SDK-BinaryPack"
                        NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

        set(CMAKE_FIND_LIBRARY_SUFFIXES ".dll")
        set(_hint "${_vacache_lib_root}/${_plat_suffix}/${_cType}/Shared")
        DEBUG_PRINT(::MESSAGE "${PROJECT_NAME}: Looking for rsslVACache_shared_DLL in : "
								"${_hint}")
        find_library(Elektron-SDK-BinaryPack_rsslVACache_shared_DLL_LIBRARY_${_cType_U}
                        NAMES rsslVACache librsslVACache NAMES_PER_DIR
                        HINTS ${_hint} 
                        DOC "Rssl Value Add Cache shared ${_cType} library from Elektron-SDK-BinaryPack"
                        NO_SYSTEM_ENVIRONMENT_PATH NO_DEFAULT_PATH)

        set(CMAKE_FIND_LIBRARY_SUFFIXES ".lib")
    endforeach()

    DEBUG_PRINT(::MESSAGE "
    IN ${CMAKE_CURRENT_LIST_DIR}
    Elektron-SDK-BinaryPack_
    dacs_LIBRARY_DEBUG_MDD                  : ${Elektron-SDK-BinaryPack_dacs_LIBRARY_DEBUG_MDD}
    dacs_LIBRARY_RELEASE_MD                 : ${Elektron-SDK-BinaryPack_dacs_LIBRARY_RELEASE_MD}
    ansi_LIBRARY_DEBUG_MDD                  : ${Elektron-SDK-BinaryPack_ansi_LIBRARY_DEBUG_MDD}
    ansi_LIBRARY_RELEASE_MD                 : ${Elektron-SDK-BinaryPack_ansi_LIBRARY_RELEASE_MD}
    rsslVACache_static_LIBRARY_DEBUG_MDD    : ${Elektron-SDK-BinaryPack_rsslVACache_LIBRARY_DEBUG_MDD}
    rsslVACache_static_LIBRARY_RELEASE_MD   : ${Elektron-SDK-BinaryPack_rsslVACache_LIBRARY_RELEASE_MD}
    rsslVACache_shared_LIBRARY_DEBUG_MDD    : ${Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY_DEBUG_MDD}
    rsslVACache_shared_DLL_LIBRARY_DEBUG_MDD: ${Elektron-SDK-BinaryPack_rsslVACache_shared_DLL_LIBRARY_DEBUG_MDD}
    rsslVACache_shared_LIBRARY_RELEASE_MD   : ${Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY_RELEASE_MD}
    rsslVACache_shared_DLL_LIBRARY_RELEASE_MD: ${Elektron-SDK-BinaryPack_rsslVACache_shared_DLL_LIBRARY_RELEASE_MD}
    ")
    mark_as_advanced(Elektron-SDK-BinaryPack_dacs_LIBRARY_DEBUG_MDD
                     Elektron-SDK-BinaryPack_dacs_LIBRARY_RELEASE_MD
                     Elektron-SDK-BinaryPack_ansi_LIBRARY_DEBUG_MDD
                     Elektron-SDK-BinaryPack_ansi_LIBRARY_RELEASE_MD
                     Elektron-SDK-BinaryPack_rsslVACache_LIBRARY_DEBUG_MDD
                     Elektron-SDK-BinaryPack_rsslVACache_LIBRARY_RELEASE_MD
                     Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY_DEBUG_MDD
                     Elektron-SDK-BinaryPack_rsslVACache_shared_LIBRARY_RELEASE_MD
                     Elektron-SDK-BinaryPack_rsslVACache_shared_DLL_LIBRARY_DEBUG_MDD
                     Elektron-SDK-BinaryPack_rsslVACache_shared_DLL_LIBRARY_RELEASE_MD
                    )
endif()

# Configuration
set(_config_dir "${CMAKE_INSTALL_PREFIX}/lib/cmake/${PROJECT_NAME}")
file(MAKE_DIRECTORY "${CMAKE_INSTALL_PREFIX}/lib/cmake" "${_config_dir}")

configure_file("${Elektron-SDK-BinaryPack_CMAKE_DIR}/${PROJECT_NAME}Targets.cmake.in" 
                    "${_config_dir}/${PROJECT_NAME}Targets.cmake" @ONLY )

configure_file( "${Elektron-SDK-BinaryPack_CMAKE_DIR}/${PROJECT_NAME}Config.cmake.in" 
                    "${_config_dir}/${PROJECT_NAME}Config.cmake" @ONLY)

configure_file( "${Elektron-SDK-BinaryPack_CMAKE_DIR}/${PROJECT_NAME}ConfigVersion.cmake.in" 
                    "${_config_dir}/${PROJECT_NAME}ConfigVersion.cmake" @ONLY)

unset(_header_suffix)
unset(_plat_suffix)
unset(_plat_suffix2)
unset(_hints)
unset(_cType)
unset(_config_dir)
unset(_config_type_list)
unset(_cType_U)
unset(_CONFIG_TYPE_U)
unset(_dacs_include_root)
unset(_ansi_include_root)
unset(_vacache_include_root)
unset(_dacs_lib_root)
unset(_ansi_lib_root)
unset(_vacache_lib_root)
