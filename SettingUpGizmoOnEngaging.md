The official Gizmo documentation can be found [here](http://www.tapir.caltech.edu/~phopkins/Site/GIZMO_files/gizmo_documentation.html#codeoverview) 

and more detail about the code can be found [here](http://www.tapir.caltech.edu/~phopkins/Site/GIZMO.html)

#### 1. You will find the code in [this repository](https://github.com/pfhopkins/gizmo-public) git clone it to download Gizmo. 

**!!!! More work to be done here: check with Abdelaziz about the code he fixed last year**


Some snippets of messages about this matter:

I have a few updates and fixes to share. Everything I will mention is referring to edits made to the gizmo stored in ```/work2/09020/tg883191/frontera/gizmo-apr2```

For eta ( limits the timestep for a particle with a large acceleration value):
I have added a print statement so that the value is written in  gizmo.out , the print statement should read something like this:
The Value of Eta or ```ErrTolIntAccuracy=0.001``` I have further commented out the lines rewriting the eta value we discussed which are:
``` #if !defined(SINGLE_STAR_AND_SSP_HYBRID_MODEL)
//     if (All.ComovingIntegrationOn)
//     {
//         All.ErrTolForceAcc = 0.005;
//         All.ErrTolIntAccuracy = 0.05;
//     }
// #endif 
```

For Kappa (limit for the probability of scattering for a given time step):
In essence, there are two checks being done to ensure that the probability calculated doesn't exceed a certain threshold in two places timestep.c and sidm_core_flux_calculation.h. Not only are the kappa values, or upper bounds on the probability of scattering, different in these two files, but further the probabilities checked against are calculated differently. In ```timestep.c``` we have ```double p_dt = prob_of_interaction(P[p].Mass, 0., PPP[p].AGS_Hsml, dV, dt)```;  while in ```sidm_core_flux_calculation.h``` we have the following ```double prob = prob_of_interaction(m_si, kernel.r, h_si, kernel.dv, local.dtime)```;

Where ```prob_of_interaction``` is defined in ```sidm_core.c``` as follows:
```double prob_of_interaction(double mass, double r, double h_si, double dV[3], double dt)
{
    double dVmag = sqrt(dV[0]*dV[0]+dV[1]*dV[1]+dV[2]*dV[2]) / All.cf_atime; // velocity in physical
    double rho_eff = mass / (h_si*h_si*h_si) * All.cf_a3inv; // density in physical
    double cx_eff = All.DM_InteractionCrossSection * g_geo(r/h_si); // effective cross section (physical) scaled to cgs
    double units = UNIT_SURFDEN_IN_CGS; // needed to convert everything to cgs
    if(All.DM_InteractionVelocityScale>0) {double x=dVmag/All.DM_InteractionVelocityScale; cx_eff/=1+x*x*x*x;} // take velocity dependence
    return rho_eff * cx_eff * dVmag * dt * units; // dimensionless probability
}

double g_geo(double r)
{
    double f, u; int i; u = r / 2.0 * GEOFACTOR_TABLE_LENGTH; i = (int) u;
    if(i >= GEOFACTOR_TABLE_LENGTH) {i = GEOFACTOR_TABLE_LENGTH - 1;}
    if(i <= 1) {f = 0.992318  + (GeoFactorTable[0] - 0.992318)*u;} else {f = GeoFactorTable[i - 1] + (GeoFactorTable[i] - GeoFactorTable[i - 1]) * (u - i);}
    return f;
}
```

Here is the message from Phil explaining the difference:
first off, the initial input "dt" being rescaled is different. in timestep.c, its the predicted timestep you will need to take for the next timestep (which, at the end of timestep.c, becomes the timestep, after passing all its checks and taking the minimum). In sidm_core_flux, it's the current timestep. Effectively sidm_core_flux is occuring midway through a timestep, while timestep.c occurs at the very end of one and beginning of the next. Second, the probability is calculated differently. you call the same "prob_of_interaction" function, but the inputs are all different: in timestep.c, you only know information local to a single cell/particle, so it uses the single-particle mass, distance = 0, effective SIDM kernel size AGS_Hsml, predicted timestep dt, and signal velocity AGS_vsig for dV oriented at an arbitrary angle. In sidm_core_flux, the probability is calculated separately for every pair of interacting neighbor particles that can causally have a chance of interaction with particle "i" in that timestep. For each pair, you calculate the -pairwise- probability of an interaction, which means you call the function with the mean particle mass, an average value of their kernel sizes, the separation equal to the actual center-of-mass separation of the cells, velocity term equal to the particle-particle relative velocity, and timestep equal to the current timestep using the minimum of the two interacting particle timesteps. Then you take the minimum over all interacting neighbors, and pass that to timestep.c to use it to help guess the -next- timestep. So in a smooth, reasonably well-behaved, well-particle-ordered system, they'll give similar answers, but still not identical, and if e.g. you have a single high-speed particle moving through a group (for velocity-independent cross-sections), they will get smaller steps (for the same 'target' probability), or other single-particle deviations, from the sidm_core_flux routine.In short, in one file you calculate the pairwise probability, while in the other, you calculate the probability using only information local to a single particle. The latter , which is in  timestep.c  makes less physical sense to me in the core collapse regime? Since it is only a proxy and the intuition I have for it is " the probability that if this particle were immersed in a medium of particles like itself, with density implied by its kernel, it would scatter in this time step".  Whether we need both checks or not is a good question that I am still trying to wrap my head around, I asked Phil but he hasn't responded yet, if anyone has ideas please let me know.

To consolidate the probability upper bounds, I have defined a global variable as follows:
in ```allvars.h``` I added inside of ```global_data_all_process``` and the ```if SIDM``` statement:
```
MyDouble DM_scattering_prob_upper_bound which is kappa and then I define its value in begrun.c. So now all one needs to do is define kappa and eta once in begrun.c in the following lines:
    All.ErrTolIntAccuracy = 0.001; // eta
    All.DM_scattering_prob_upper_bound = 0.2; //Kappa
```


#### 2. Copy Template-Config.sh to Config.sh and Template-Makefile.systype to Makefile.systype

#### 3. In the file named Makefile.systype check there is this line:
```
SYSTYPE="engaging"
```
and keep all the other lines commented out (starting with a #) 


#### 4. Modify the following lines like this in the Makefile:

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

Then set ```FFTW3_HOME=/orcd/software/core/001/spack/pkg/fftw/3.3.10/2qziucy``` and all other paths in Makefile


#### 5. Load the modules:

```module load openmpi/4.1.4 hdf5/1.14.3 gsl/2.7.1 fftw/3.3.10```


#### 6. Check all libs are available:

in your terminal type:  ```ldd GIZMO```

you should get something like this: 

```
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
```


#### 7. Build the program:

in the directory run "make clean" then "make"

Warnings are okay as long as the code compiles! 
