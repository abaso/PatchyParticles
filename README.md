# PatchyParticles

`PatchyParticles` is a code for perfoming Monte Carlo simulations of hard particles decorated with four patches modelled through the [Kern-Frenkel](http://www.sklogwiki.org/SklogWiki/index.php/Kern_and_Frenkel_patchy_model) pair interaction potential. The code is *educational* in the sense that it is meant to be read as much as meant to be used.  

## Compilation

`PatchyParticles` does not require any dependencies but a C compiler, the standard C library and GNU's `make`. The default compiler, specified in the `makefile` file, is gcc, but the code can be compiled with icc as well. The compilation has been tested with gcc >= 4.8 and icc >= 13.

In order to compile the code it is sufficient to launch make within the main folder: 

	$ make 

Depending on the platform, it might be necessary to explicitly pass the makefile to `make`. This can be done by issuing

	$ make -f makefile

The code can be compiled with icc with the command 

	$ make -f makefile.icc
	
The default behaviour is to compile `PatchyParticles` with full optimisations enabled (that is, with the options `-O3 -ffast-math -DNDEBUG` or `-fast -DNDEBUG` if the code is compiled with gcc or icc, respectively)). The code can also be compiled with optimisations and debug flags (`make g=1`) or with debug flags only (`make dbg=1`).

At the end of the compilation stage there will be two executables: `PatchyParticles` and `generator`.

## Usage

The `PatchyParticles` program takes one mandatory argument, the input file. The syntax of the file is quite simple, as it is just a list of "key = value" lines. The [next section](#input-file) has a list of mandatory options. The Examples folder contains some simple input files that can be used as starting points. 

The `generator` binary can be used to generate initial configurations of N particles at density ρ, with ρ < 0.7. It requires two arguments (an integer for N and a decimal number for ρ). The program prints the configuration in a new file named `generated.rrr`.

## Input file

Here is a list of mandatory options. Please refer to the input files in the `Examples` folder for common values. 

### Simulation options

* `Dynamics = <int>`: use 0 for rototranslations, 1 for AVB and 2 for VMMC.
* `Ensemble = <int>`: use 0 for NVT, 1 for Grand Canonical (muVT), 3 for Successive 
Umbrella Sampling.
* `Temperature = <float>`: temperature of the simulation, in units of the patch-patch 
bond.
* `Steps = <int>`: length of the simulation, in Monte Carlo steps.
* `GC_N_max = <int>`: maximum number of particles, above which the simulation will be 
stopped. Meaningful only for Grand Canonical (or SUS) simulations

### Input/output options

* `Initial_conditions_file`: initial configuration.
* `Print_every = <int>`: output frequency for energy, density and acceptances.
* `Save_every = <int>`: output frequency for configurations.
* `Energy_file = <string>`: name of the output file for the energy. 
* `Density_file = <string>`: name of the output file for the density.
* `Configuration_last = <string>`: name of the last configuration file (printed with frequency `Save_every` and at the end of the simulation)
* `Confguration_folder = <string>`: name of the folder where configuration files will be stored.

### Kern-Frenkel options

* `KF_delta = <float>`: Radial width of the Kern-Frenkel patches.
* `KF_cosmax = <float>`: Angular width of the Kern-Frenkel patches.

### Monte Carlo moves options

* `Disp_max = <float>`: maximum trial displacement for translations.
* `Theta_max = <float>`: maximum trial angular displacement for rotations, in radians.
* `vmmc_max_move = <float>`: maximum allowed displacement for VMMC moves: if a vmmc move attempts to move a particle for more than this value, the move will be rejected.
* `vmmc_max_cluster = <int>`: maximum cluster size for VMMC moves: if a vmmc move attempts to move more than this number of particles, the move will be rejected.

### Some useful non-mandatory options

* `Restart_step_counter = <int>`: true by default, if set to false will restart the simulation from the time step found in the initial configuration file and append the energy, density and acceptance output to their respective files.
* `Log_file = <string>`: by default PatchyParticles writes the output to the standard error. If the input file contains this option, PatchyParticles will redirect its output to the specified file.
* `Acceptance_file = <string>`: by default acceptance probabilities are written to the `acceptance.dat` file. If given, the value found in this option will be used instead.

## Output files

A simulation will produce at least two files, which by default are `energy.dat` and `acceptance.dat`. If Grand Canonical or SUS simulations are run, there will also be a `density.dat` file. Each line of these files contain the time step and the instantaneous energy, acceptance probabilities and density.

The moves to which acceptance probabilities refer depend on the chosen ensemble and dynamics:

* `dynamics = 0`: one column for the rototranslation acceptance
* `dynamics = 1`: one column for the VMMC acceptance
* `dynamics = 2`: two columns for the rototranslations and AVB acceptances

If `ensemble = 1` or `ensemble = 3` the next-to-last and last columns contain the acceptance probabilities of particle additions and deletions, respectively.

## Configuration files

PatchyParticles uses plain-text configuration files to store some information about the system and the positions and orientations of each particle. The first (header) line contains the time step of the configuration, the number of particles and the size of the box along the x, y and z directions.

The remaining part of the file contain the orientations and positions of the particles. The information about each particle takes up three lines. The first two store the two upper rows of the orientation matrix, while the third is the position of a particle's centre of mass.   

## Code organisation

* The `main.c` file contains calls to the initialisation functions, the calculation of the initial energy, the main loop and the calls to the cleanup functions.
* The general data structures used throughout the code, as well as some useful macros, are stored in the `defs.h` file.
* The `MC.c, MC.h` pair contains the main logic of the code. It manages the calculation of the energy, the particle rototranslations, the simulation of the different ensembles, *etc.* 
* The `avb.c, avb.h` and `vmmc.c, vmmc.h` pairs contain the data structures and functions pertaining to the Aggregation-Volume-Bias and Virtual-Move-Monte-Carlo moves that can be optionally enabled to speed up the simulation efficiencies.
* The `cells.c, cells.h` pair contain the data structures and functions pertaining to the linked-lists used to keep track of the list of neighbours of each particle.
* The `utils.c, utils.h` pair contains commonly-used functions to work with vectors and matrices.
* The `system.c, system.h` pair contains the functions used to initialise and cleanup the main data structure (`System`).
* The `output.c, output.h` pair contains the functions used to initialise and cleanup the data structure responsible for printing the output (`Output`), as well as the functions that actually print most of the output.
* The `parse_input.c, parse_input.h` pair contains the data structures, logic and functions used by the code to parse the input file passed to `PatchyParticles`  

## Acknowledgements

The code has been developed by Lorenzo Rovigatti (CNR-ISC), John Russo (School of Mathematics, University of Bristol) and Flavio Romano (Dipartimento di Scienze Molecolari e Nanosistemi, Università Ca' Foscari di Venezia)
