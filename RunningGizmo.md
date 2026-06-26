Once the ics file is ready (output of GLueGalaxies.ipynb

#### 1. create a parameter file
an example for a ‘minimal’ parameterfile can be found in the [Gizmo Documentation](http://www.tapir.caltech.edu/~phopkins/Site/GIZMO_files/gizmo_documentation.html#params-generic)

Make sure to include the correct paths for the ics file and the output direcotry and keep track of these paths as you start running multiple sims

#### 2. create a slurm file to submit your bash job
a useful guide can be found in the [engaging documentation](https://orcd-docs.mit.edu/recipes/many-jobs/) 

some useful flags are:

`-N` for the numebr of nordes 

`--ntasks-per-node`

`-p` for the partition (on engaging the Necib group uses: `pi_lnecib`) 

`--mem-per-cpu` 

`-t` for max time in days-hours:minutes 

`-o` for the output file 

`-e` for the error file 


make sure to lad the packages you need 
example: 
`module load openmpi/4.1.4 hdf5/1.14.3 gsl/2.7.1 fftw/3.3.10`

for checks, I like to use:
`which mpirun`

you can add to the end of your slurm: 
`mpirun -oversubscribe --mca orte_base_help_aggregate 0 -np 140 $1 $2 $3` 

#### 3. Submit the job
from the terminal submit the job using:

`sbatch path/to/slurm/file path/to/excecutable path/to/param/file 0 (or 1 if you are restarting the sim)`
