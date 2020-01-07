# NOTE:  I decided against using meson.  It seemed too complicated.
# I tried to build openblas as a cmake subproject instead.
# That also failed, even though I could build it with cmake and 
# ninja outside of meson.  Appeared to be a bug in Meson 0.52.0.
# Adding this in case I ever decide to try again.
# Note that I didn't realize when drafting this that OpenBLAS
# requires FORTRAN to compile.

project('openblas', 'c',
        version:'0.3.7',
        license:'BSD-3-Clause',
        default_options : ['warning_level=3', 'c_std=c11'])

openblas_incdir = include_directories('.', is_system : true)
openblas_sources = files('src/gtest-all.cc')

# IF MSVC - set NO_LAPACK 1 and set NO_LAPACKE 1

# IF Linux - set NO_AFFINITY ON; else set NO_AFFINITY 1 ???
SUBDIRS = ['interface', 'driver/level2', 'driver/level3', 'driver/others']

# IF no lapack
# SUBDIRS += ['lapack']
# Except that this requires fortran?

# OpenBLAS wants a CPU target
# OpenBLAS wants an OS target

# LIB_COMPONENTS:
# BLAS (optional - fortran?)
# CBLAS (optional)
# LAPACK (optional)
# LAPACKE (optional, if LAPACK true)
# RELAPACK (optional, if LAPACK true)

# BLAS or CBLAS must be set

# NUM_PARALLEL can be set...
# NUM_THREADS can be set...

# For Darwin, expects MACOSX_DEPLOYMENT_TARGET; defaults to 10.8

# For Linux, adds '-lm' to link in math library; also sets NO_EXPRECISION=1

# For WINNT, sets NEED_PIC=0; NO_EXPRECISION=1, EXTRALIB += -defaultlib:advapi32, 
# For WINNT, depending on compiler and version, sets -DMS_ABI
# dependency('threads') ?

# For x86, sets -mincoming-stack-boundary=2

# For all OS except WINNT, CYGWIN_NT, Interix, Android:  if NUM_THREAD>1, add -lpthread

# SO MANY OTHER OPTIONS

# INTERFACE64?  Something that can be passed in?

# Run make inside each member of SUBDIRS

### SUBDIRS: interface 



openblas_deps = []

openblas_cpp_arg = []

xtensor_blas_dep = declare_dependency(include_directories:include_directories('.', is_system: true),
                                      sources : openblas_sources,
                                      dependencies:openblas_deps,
                                      compile_args:openblas_cpp_arg)


