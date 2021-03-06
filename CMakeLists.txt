cmake_minimum_required(VERSION 3.8)
project(aubio C)
include (CheckIncludeFile)

if (WIN32)
	if (MSVC)
		add_definitions(
			-D_CRT_SECURE_NO_WARNINGS=1
		)
	endif()
	add_definitions(
		-DHAVE_WIN_HACKS
	)
endif()

if (APPLE)
	set(BLA_VENDOR Apple)
	find_package(BLAS REQUIRED MODULE)
	set(BLAS_FLAG -DHAVE_ACCELERATE=1)
else()
	set(BLA_VENDOR OpenBLAS)
	find_package(BLAS MODULE)
	if (BLAS_FOUND)
		CHECK_INCLUDE_FILE(openblas/cblas.h STANDARD_OPENBLAS)
		CHECK_INCLUDE_FILE(cblas-openblas.h UBUNTU_OPENBLAS)
		if (UBUNTU_OPENBLAS AND NOT STANDARD_OPENBLAS)
			set(BLAS_FLAG -DHAVE_BLAS=1 -DHAVE_CBLAS_H=1)
		else()
			set(BLAS_FLAG -DHAVE_BLAS=1 -DHAVE_OPENBLAS_CBLAS_H=1)
		endif()
	else()
		set(BLA_VENDOR ATLAS)
		# put as required as it is mandatory to have a blas library and
		# this is the last chance to get one
		find_package(BLAS REQUIRED MODULE)
		if (BLAS_FOUND)
			set(BLAS_FLAG -DHAVE_BLAS=1 -DHAVE_ATLAS_CBLAS_H=1)
		endif()
	endif()
endif()

find_package(FFTW3 COMPONENTS single REQUIRED MODULE)

message("Detected Blas library: " ${BLA_VENDOR})

set(CMAKE_DEBUG_POSTFIX d)

file(GLOB_RECURSE AUBIO_SOURCES src/*.c)

add_library(aubio STATIC ${AUBIO_SOURCES})

target_include_directories(aubio PUBLIC
	src
	${FFTW3_INCLUDE_DIR}
)

target_link_libraries(aubio PUBLIC
	${BLAS_LIBRARIES}
	${FFTW3F_LIBRARY}
)

# We need both HAVE_FFTW3 and HAVE_FFTW3F; using only the latter does nothing, using only the former is double-precision mode.
target_compile_definitions(aubio
     PUBLIC HAVE_FFTW3=1
            HAVE_FFTW3F=1
            ${BLAS_FLAG}
            HAVE_STDLIB_H=1
            HAVE_STDIO_H=1
            HAVE_MATH_H=1
            HAVE_STRING_H=1
            HAVE_LIMITS_H=1
            HAVE_STDARG_H=1
            HAVE_C99_VARARGS_MACROS=1
)
