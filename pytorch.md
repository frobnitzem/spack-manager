Instructions for installing pytorch

```shell
git clone -b pytorch --recurse-submodules https://github.com/frobnitzem/spack-manager.git
cd spack-manager
(cd spack && git checkout develop -- var/spack/repos/builtin/packages/py-torch)

bash
module load cray-python
export SPACK_MANAGER=$PWD
. ./start.sh
quick-create --name torch --spec py-torch%clang+rocm+distributed amdgpu_target=gfx90a
spack concretize
spack install
exit
```

I also applied the following patch to `rocm-5.4.0/lib/cmake/hip/hip-config.cmake`:
```
160,209d159
< if(HIP_COMPILER STREQUAL "clang")
<   if(WIN32)
<     # Using SDK folder
<     file(TO_CMAKE_PATH "${HIP_PATH}" HIP_CLANG_ROOT)
<     if (NOT EXISTS "${HIP_CLANG_ROOT}/bin/clang.exe")
<       # if using install folder
<       file(TO_CMAKE_PATH "${HIP_PATH}/../lc" HIP_CLANG_ROOT)
<     endif()
<   else()
<     set(HIP_CLANG_ROOT "${ROCM_PATH}/llvm")
<   endif()
<   if(NOT HIP_CXX_COMPILER)
<     set(HIP_CXX_COMPILER ${CMAKE_CXX_COMPILER})
<   endif()
<   if(HIP_CXX_COMPILER MATCHES ".*hipcc" OR HIP_CXX_COMPILER MATCHES ".*clang\\+\\+")
<     execute_process(COMMAND ${HIP_CXX_COMPILER} --version
<                     OUTPUT_STRIP_TRAILING_WHITESPACE
<                     OUTPUT_VARIABLE HIP_CXX_COMPILER_VERSION_OUTPUT)
<     # Capture the repo, branch and patch level details of the HIP CXX Compiler.
<     # Ex. clang version 13.0.0 (https://github.com/ROCm-Developer-Tools/HIP main 12345 COMMIT_HASH)
<     # HIP_CLANG_REPO: https://github.com/ROCm-Developer-Tools/HIP
<     # HIP_CLANG_BRANCH: main
<     # HIP_CLANG_PATCH_LEVEL: 12345
<     if(${HIP_CXX_COMPILER_VERSION_OUTPUT} MATCHES "clang version [0-9]+\\.[0-9]+\\.[0-9]+ \\(([^ \n]*) ([^ \n]*) ([^ \n]*)")
<       set(HIP_CLANG_REPO ${CMAKE_MATCH_1})
<       set(HIP_CLANG_BRANCH ${CMAKE_MATCH_2})
<       set(HIP_CLANG_PATCH_LEVEL ${CMAKE_MATCH_3})
<     endif()
<   endif()
<   if(HIP_CXX_COMPILER MATCHES ".*hipcc")
<     if(HIP_CXX_COMPILER_VERSION_OUTPUT MATCHES "InstalledDir:[ \t]*([^\n]*)")
<       get_filename_component(HIP_CLANG_ROOT "${CMAKE_MATCH_1}" DIRECTORY)
<     endif()
<   elseif (HIP_CXX_COMPILER MATCHES ".*clang\\+\\+")
<     get_filename_component(_HIP_CLANG_REAL_PATH "${HIP_CXX_COMPILER}" REALPATH)
<     get_filename_component(_HIP_CLANG_BIN_PATH "${_HIP_CLANG_REAL_PATH}" DIRECTORY)
<     get_filename_component(HIP_CLANG_ROOT "${_HIP_CLANG_BIN_PATH}" DIRECTORY)
<   endif()
<   file(GLOB HIP_CLANG_INCLUDE_SEARCH_PATHS ${HIP_CLANG_ROOT}/lib/clang/*/include)
<   find_path(HIP_CLANG_INCLUDE_PATH stddef.h
<       HINTS
<           ${HIP_CLANG_INCLUDE_SEARCH_PATHS}
<       NO_DEFAULT_PATH)
<   if(NOT WIN32)
<     find_dependency(AMDDeviceLibs)
<   endif()
<   set(AMDGPU_TARGETS "gfx900;gfx906;gfx908;gfx90a;gfx1030" CACHE STRING "AMD GPU targets to compile for")
<   set(GPU_TARGETS "${AMDGPU_TARGETS}" CACHE STRING "GPU targets to compile for")
< endif()
< 
226a177
> set(HIP_CLANG_ROOT "${_IMPORT_PREFIX}/llvm")
```

To do this as a user, I symlinked all the entries from
/opt/rocm-5.4.0 to
/lustre/orion/stf006/world-shared/rogersdd/rocm-5.4.0
and then replaced just lib/cmake/hip.
Then modified py-torch/package.py to use:

    env.set("ROCM_PATH", "/lustre/orion/stf006/world-shared/rogersdd/rocm-5.4.0")
    ...

        env.set("CMAKE_MODULE_PATH", "/lustre/orion/stf006/world-shared/rogersdd/rocm-5.4.0/lib/cmake/hip")

