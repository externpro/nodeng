cmake_minimum_required(VERSION 4.3)
project(nodeng VERSION 24.13.0)
# https://nodejs.org/dist/v24.13.0/SHASUMS256.txt
set(SHA256_Linux e798599612f4bb71333a3397ab0d095fd62214e115aea45aa858a145fc72d67e) # node-v24.13.0-linux-x64.tar.xz
set(SHA256_win64 ca2742695be8de44027d71b3f53a4bdb36009b95575fe1ae6f7f0b5ce091cb88) # node-v24.13.0-win-x64.zip
set(SHA256_arm64 aa881151bd0f9f154a0424dd60a72e9ce10672619121658c278a24327ef46831) # node-v24.13.0-linux-arm64.tar.xz
set(SHA256_macos c59a517e9147f25c6167426875a571432f1478c1d7ee7ecc10baa46b0d0e8545) # node-v24.13.0-darwin-arm64.tar.xz
include(FetchContent)
cmake_policy(SET CMP0168 OLD) # don't want to see download progress
string(TOLOWER ${CMAKE_SYSTEM_NAME} os)
set(ver ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  set(os win)
  set(arch x64)
  set(ext zip)
  set(sha ${SHA256_win64})
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(bin "bin/")
  set(ext tar.xz)
  if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64|AMD64)$")
    set(arch x64)
    set(sha ${SHA256_Linux})
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|ARM64)$")
    set(arch arm64)
    set(sha ${SHA256_arm64})
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  set(bin "bin/")
  set(ext tar.xz)
  if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|ARM64)$")
    set(arch arm64)
    set(sha ${SHA256_macos})
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
else()
  message(FATAL_ERROR "Unsupported OS: ${CMAKE_SYSTEM_NAME}")
endif()
set(url https://nodejs.org/dist/v${ver}/node-v${ver}-${os}-${arch}.${ext})
set(fcName ${PROJECT_NAME}_fc)
FetchContent_Declare(${fcName} URL ${url} URL_HASH SHA256=${sha})
FetchContent_MakeAvailable(${fcName})
string(TOLOWER ${fcName} fcName)
set(pth ${FETCHCONTENT_BASE_DIR}/${fcName}-src)
xpExternPackage(
  EXE_PATH ${CMAKE_INSTALL_BINDIR}/node
  BASE v0 XPDIFF "bin"
  WEB "https://nodejs.org/en/blog/release/v${ver}/"
  DESC "node executable bundled as externpro devel package to build angular (ng) projects"
  LICENSE "[MIT](https://raw.githubusercontent.com/nodejs/node/v${ver}/LICENSE 'MIT License')"
  )
install(PROGRAMS ${pth}/${bin}node${CMAKE_EXECUTABLE_SUFFIX} DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES ${pth}/CHANGELOG.md ${pth}/LICENSE ${pth}/README.md DESTINATION ${CMAKE_INSTALL_DATADIR})
