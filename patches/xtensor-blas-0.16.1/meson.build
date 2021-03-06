project('xtensor-blas', 'cpp',
    version:'0.16.1',
    license:'BSD-3-Clause',
    default_options : ['warning_level=3', 'cpp_std=c++14'])


xtensor_dep = dependency('xtensor',
    version: ['>=0.20.4', '<1.0.0'],
    fallback: ['xtensor', 'xtensor_dep'],
    required: false)

xtensor_blas_cpp_arg = []

xtensor_blas_deps = []

# xtensor-blas is an interface between xtensor and BLAS/LAPACK.  It enables use of xlinalg.hpp functions in xtensor.
# BLAS is a collection of low-level matrix and vector arithmetic operations.  Basic Linear Algebra Subprograms
# LAPACK is a collection of higher-level linear algebra operations.  Linear Algebra PACKage
# netlib maintains a reference implementation of each (www.netlib.org). Their APIs have been used to create
# better performing libraries over the years.

# Here we begin the long journey of attempting to find said BLAS and LAPACK implementations.
# This optimistic initial value will be updated by the foreach below if we can't find the symbols we need.
blas_lapack_found = true
cpp_compiler = meson.get_compiler('cpp')

# We need these symbols defined, and have a rough guess of which library might define them, but
# there are a wide variety of possible dependency combinations we may need depending on what the
# system's packages provide where. This dict is in the form of {function_name: [libraries, that,
# might, provide, it]}
#
# Our first choice is always OpenBLAS, which sometimes provides a complete LAPACK implementation,
# then Accelerate, which can provide these on Mac.
#
# TODO: Adjust these lists to include whatever Intel's MKL (Math Kernel Library) is called in
# pkg-config.
looking_for = {
    'cblas_dgemv': ['openblas', 'Accelerate', 'blas', 'cblas'],
    'dgesv_': ['openblas', 'Accelerate', 'lapack'],
}
detected_blas_deps = []
dependency_names = []

foreach symbol, depnames : looking_for
    found_function = cpp_compiler.has_function(symbol, dependencies: detected_blas_deps)
    if not found_function
        foreach possible_dep_name : depnames
            possible_dep = dependency(possible_dep_name, required: false)
            if not possible_dep.found()
                continue
            endif
            if cpp_compiler.has_function(symbol, dependencies: detected_blas_deps + [possible_dep])
                found_function = true
                detected_blas_deps += [possible_dep]
                dependency_names += [possible_dep.name()]
                break
            endif
        endforeach
    endif
    if not found_function
        blas_lapack_found = false
        break
    endif
endforeach
if blas_lapack_found
    xtensor_blas_deps += detected_blas_deps
else
    dependency_names = []
endif

# Our next choice for Windows are the prebuilt OpenBLAS libraries that we included with the Meson wrap.
if (not blas_lapack_found) and (host_machine.system() == 'windows')

    prebuilt_openblas_dep = subproject('openblas-dll').get_variable('openblas_dll_dep')

    if prebuilt_openblas_dep.found()
        blas_lapack_found = true
        xtensor_blas_deps += [prebuilt_openblas_dep]
    endif

endif

# Our next choice is to try to build a library included as a subproject.
if not blas_lapack_found

    # TODO:  Add subproject dependency; set the following to true if found
    blas_lapack_found = false

    # Top contenders include:
    # - OpenBLAS
    #   * Provides an optimized BLAS
    #   * Although not advertised, provdies a complete LAPACK implementation, partially optimized.
    #   * Requires a FORTRAN compiler
    # - BLIS - https://github.com/flame/blis
    #   * Provides BLAS compatibility.  Claims to be faster than OpenBLAS.
    #   * Unclear if it provides any LAPACK functions.
    # - Flame - https://github.com/flame/libflame
    #   * Provides some optimized LAPACK functions
    #   * For LAPACK functions that aren't optimized, they also have a complete reference implementation.
    #   * Provides some kind of BLAS, but should be able to interoperate with BLIS
    #   * Requires a FORTRAN compiler

endif

# A last resort is to try to use FLENS BLAS and FLENS LAPACK that come with xtensor-blas.
if not blas_lapack_found
    # FLENS BLAS is a complete implementation of BLAS, but FLENS LAPACK is not a complete implementation of LAPACK.
    # But if you'd like to give it a try, uncomment the following line and set blas_lapack_found to true.
    # xtensor_blas_cpp_arg += ['-DXTENSOR_USE_FLENS_BLAS']
    blas_lapack_found = false
endif

include_directories('include', is_system: true)

if blas_lapack_found and xtensor_dep.found()
    xtensor_blas_dep = declare_dependency(
        include_directories: include_directories('include', is_system: true),
        dependencies: [xtensor_dep, xtensor_blas_deps],
        compile_args: xtensor_blas_cpp_arg)
else
    xtensor_blas_dep = dependency('', required: false) # aka "not found"
endif

extra_cflags = []

# Workaround from mesonbuild/meson github issue 2550
if meson.is_subproject()
    install_dir = get_option('subproj_includedir')
    extra_cflags += '-I' + get_option('prefix') / install_dir
else
    install_dir = get_option('includedir')
endif

install_subdir('include/xtensor-blas/', install_dir: install_dir)

install_subdir('include/xflens', install_dir: install_dir)

dependency_names += 'xtensor'

pkgconfig = import('pkgconfig')
pkgconfig.generate(name: meson.project_name(),
    version: meson.project_version(),
    description: 'an extension to the xtensor library, offering bindings to BLAS and LAPACK libraries through cxxblas and cxxlapack from the FLENS project',
    requires: dependency_names,
    extra_cflags: extra_cflags)
