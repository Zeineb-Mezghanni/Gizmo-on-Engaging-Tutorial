The official Gizmo documentation can be found [here](http://www.tapir.caltech.edu/~phopkins/Site/GIZMO_files/gizmo_documentation.html#codeoverview) 

and more detail about the code can be found [here](http://www.tapir.caltech.edu/~phopkins/Site/GIZMO.html)

## You will find the code in [this repository](https://github.com/pfhopkins/gizmo-public) git clone it to download Gizmo. 


## Copy Template-Config.sh to Config.sh and Template-Makefile.systype to Makefile.systype

## In the file named Makefile.systype check there is this line:
```
SYSTYPE="engaging"
```
and keep all the other lines commented out (starting with a #) 


## Modify the following lines like this in the Makefile:

```
ifeq ($(SYSTYPE),"engaging")
CC = mpicc #instead ofmpicc
CXX = mpiccxx #instead of mpipc
FC = $(CC)
GSL_INCL = -I/orcd/software/community/001/spack/pkg/gsl/2.7.1/y74b3m5/include
GSL_LIBS = -L/orcd/software/community/001/spack/pkg/gsl/2.7.1/y74b3m5/lib
FFTW_INCL= -I/orcd/software/core/001/spack/pkg/fftw/3.3.10/2qziucy/include
FFTW_LIBS= -L/orcd/software/core/001/spack/pkg/fftw/3.3.10/2qziucy/lib
MPICHINCL= -I/orcd/software/core/001/spack/pkg/openmpi/4.1.4/zuyo6jx/include
MPICHLIB = -L/orcd/software/core/001/spack/pkg/openmpi/4.1.4/zuyo6jx/lib
HDF5INCL = -I/orcd/software/community/001/spack/pkg/hdf5/1.14.3/suffu3v/include
HDF5LIB = -L/orcd/software/community/001/spack/pkg/hdf5/1.14.3/suffu3v/lib -lhdf5 -lz
OPT += -DH5_USE_16_API
endif
```

Use "module show <module-name>" to check the paths, for example, "module show fftw/3.3.10"

You can set FFTW3_HOME=/orcd/software/core/001/spack/pkg/fftw/3.3.10/2qziucy and all other paths in Makefile


## Load the modules:

module load openmpi/4.1.4 hdf5/1.14.3 gsl/2.7.1 fftw/3.3.10


## Check all libs are available:

in your terminal type:  ldd GIZMO

you should find something like this: 

linux-vdso.so.1 (0x00007ffd9f6ed000)
libhdf5.so.310 => /orcd/software/community/001/spack/pkg/hdf5/1.14.3/suffu3v/lib/libhdf5.so.310 (0x00007f5b49f75000)
libz.so.1 => /lib64/libz.so.1 (0x00007f5b49d5d000)
libgsl.so.27 => /orcd/software/community/001/spack/pkg/gsl/2.7.1/y74b3m5/lib/libgsl.so.27 (0x00007f5b49890000)
libgslcblas.so.0 => /orcd/software/community/001/spack/pkg/gsl/2.7.1/y74b3m5/lib/libgslcblas.so.0 (0x00007f5b4964f000)
libm.so.6 => /lib64/libm.so.6 (0x00007f5b492cd000)
libmpi.so.40 => /orcd/software/core/001/spack/pkg/openmpi/4.1.4/zuyo6jx/lib/libmpi.so.40 (0x00007f5b48f90000)
libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5b48d70000)
libc.so.6 => /lib64/libc.so.6 (0x00007f5b4899a000)
libdl.so.2 => /lib64/libdl.so.2 (0x00007f5b48796000)
/lib64/ld-linux-x86-64.so.2 (0x00007f5b4a5fc000)
libopen-rte.so.40 => /orcd/software/core/001/spack/pkg/openmpi/4.1.4/zuyo6jx/lib/libopen-rte.so.40 (0x00007f5b484dc000)
libopen-pal.so.40 => /orcd/software/core/001/spack/pkg/openmpi/4.1.4/zuyo6jx/lib/libopen-pal.so.40 (0x00007f5b48227000)
librt.so.1 => /lib64/librt.so.1 (0x00007f5b4801f000)
libutil.so.1 => /lib64/libutil.so.1 (0x00007f5b47e1b000)
libhwloc.so.15 => /orcd/software/core/001/spack/pkg/hwloc/2.9.1/miyvzyy/lib/libhwloc.so.15 (0x00007f5b47bbf000)
libevent_core-2.1.so.7 => /orcd/software/core/001/spack/pkg/libevent/2.1.12/rjddy4q/lib/libevent_core-2.1.so.7 (0x00007f5b4798a000)
libevent_pthreads-2.1.so.7 =>/orcd/software/core/001/spack/pkg/libevent/2.1.12/rjddy4q/lib/libevent_pthreads-2.1.so.7 (0x00007f5b47787000)
libpciaccess.so.0 => /orcd/software/core/001/spack/pkg/libpciaccess/0.17/urkcbku/lib/libpciaccess.so.0 (0x00007f5b4757e000)
libxml2.so.2 => /orcd/software/core/001/spack/pkg/libxml2/2.10.3/hi7cu4u/lib/libxml2.so.2 (0x00007f5b4721b000)
liblzma.so.5 => /orcd/software/core/001/spack/pkg/xz/5.4.6/kketj7w/lib/liblzma.so.5 (0x00007f5b46fed000)
libiconv.so.2 => /orcd/software/core/001/spack/pkg/libiconv/1.17/hgnhhb7/lib/libiconv.so.2 (0x00007f5b46ce2000)



## build the program:

in the directory run "make clean" then "make"

Warnings are okay as long as the code compiles! 







Once installed, cd into the directory of *gizmo-public* and module load the following from the terminal: 
#### 
