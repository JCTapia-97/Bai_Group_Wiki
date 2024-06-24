## Tutorials

- A step by step introduction with [LJ
  fluid](https://lammpstutorials.github.io/sphinx/build/html/tutorials/level1/lennard-jones-fluid.html)
  (Simon Gravelle) as a good starting point and has additional example
  simulations for more complex systems.
- LAMMPS tutorial ran in [Jupyter
  Notebook](https://github.com/mrkllntschpp/lammps-tutorials) (Mark
  Tschopp).
- In general, the [LAMMPS Manual
  Page](https://docs.lammps.org/Manual.html) has very good scientific
  description of what each command does.
- Papers in the group Zotero library, inside folders labeled classics.
- Classic texts listed under [Computational
  modeling](Books_and_Courses#Computational_modeling "wikilink").

## Compiling LAMMPS with CMake

While many HPCs have images/modules for LAMMPS, they may not contain all
the packages required for our simulations. A simple way to check this is
by running a simulation on the HPC and seeing if an error occurs. For
our simulations, the most basic packages are `KSPACE` for k-space
calculations, `MOLECULE` for general MD simulations, `RIGID` for rigid
body mechanics, and `EXTRA-MOLECULE` for Fourier style dihedrals.
Additionally, optional packages which may be useful are `PYTHON` for
interfacing LAMMPS with Python (useful for ASE), `EXTRA-DUMP` for
generating `.xtc` files for trajectories, and `COLVARS` for
meta-dynamics. If an image/binary does not have these packages compiled
then LAMMPS must be compiled yourself. In general, the following steps
should be used for compiling LAMMPS but more optimization flags can be
used and found in [Compilation &
Tuning#LAMMPS](Compilation_&_Tuning#LAMMPS "wikilink").

1.  Download the source code from git following the [LAMMPS
    documentation](https://docs.lammps.org/Install_git.html)
2.  `cd` into the directory and `mkdir build && cd build`.
3.  Find the `C`, `C++`, `Fortran` compilers and their respective MPI
    versions. This can be found from the respective HPC's documentation
    (e.g. on
    [Expanse's](https://www.sdsc.edu/support/user_guides/expanse.html)
    documenation, the corresponding compilers can be found in the
    Compiling Codes section and the Intel Compilers sub-section.
4.  Determine which Fast Fourier Transform (FFT) algorithm you're going
    to be using. Generally, `FFTW3` works best with GNU compilers, `MKL`
    works best with Intel compilers, and `AOCL-FFTW` works best with AMD
    compilers. If none is chosen, LAMMPS uses their own built-in FFT
    algorithm called `KISS` which is not very performant.
5.  Start an interactive job and load the appropriate compiler modules.
    The following modules are for Expanse and will need to be changed
    depending on which HPC is used or if Expanse settings change.
        module rm intel cpu
        module add cpu/0.15.4
        module add intel/19.1.1.217 intel-mkl/2019.1.144 intel-mpi/2019.8.254
6.  Generate the cmake configuration file. The following flags are for
    Expanse and will need to be changed depending on which HPC is used
    or if Expanse settings change
        cmake -DCMAKE_CXX_COMPILER=mpicxx -DCMAKE_C_COMPILER=mpicc -DCMAKE_Fortran_COMPILER=mpif90 -DCMAKE_CXX_FLAGS='-O3 -march=core-avx2 -mtune=core-avx2 -axCORE-AVX512 -no-prec-div' -DBUILD_MPI=yes -DBUILD_OMP=yes -DFFT=MKL -DLAMMPS_MEMALIGN=64 -DBUILD_SHARED_LIBS=on -DLAMMPS_EXCEPTIONS=yes -DPKG_KSPACE=yes -DPKG_MOLECULE=yes -DPKG_RIGID=yes -DPKG_EXTRA-DUMP=yes -DPKG_COLVARS=yes -DPKG_EXTRA-MOLECULE=yes -DCMAKE_INSTALL_PREFIX=$HOME/tools/lammps/intel-impi-mkl ../cmake

    The important flags are described below

    - `DCMAKE_*_COMPILER` is the name of the compiler for the respective
      language. `CXX` is for C++, `C` is for C, and `Fortran` is for
      Fortran.
    - `DCMAKE_*_FLAGS` described what optimization flags are used when
      compiling C++ code
    - `DFFT` is which FFT algorithm used (if unset, will use `KISS`)
    - `DLAMMPS*` flags are for LAMMPS optimizations and descriptions for
      each can be found [on their
      Documentation](https://docs.lammps.org/Build_settings.html)
    - `DPKG_*` describe which packages will be installed and all
      packages can be found
      [here](https://docs.lammps.org/Packages_list.html)
    - `DCMAKE_INSTALL_PREFIX` describes the path the binary and library
      files will be copied to when installed
7.  Compile the code using `make -j N` where N being the total number of
    cores in the interactive session. With this configuration, with 4
    cores, it takes about 10 minutes to compile.
8.  Install cmkae into the `DCMAKE_INSTALL_PREFIX` directory by using
    `make install`.
9.  This should properly compile LAMMPS but make sure to make the
    necessary changes to `sub-lammps-*.script` to call the correct
    LAMMPS binary.

## Example simulations

### NVT Liquid-phase UA Butane

- Familiarisation with the PDB file format
- Generating basic systems with PACKMOL
- Generating LAMMPS topology file with VMD
- Generating LAMMPS input file and its important functions

#### Packing a Simulation Box with Packmol

1.  First we need to generate a PDB file of a single butane molecule
    using Schrodinger, Material Studio, or any other preferred software.
    Alternatively, a PDB file can be found on the internet. Remove any
    hydrogens from the PDB file and change the "Atom Name" the remaining
    carbon atoms so that the self-similar atoms are renamed identically
    (e.g. the 2 central carbons are CH2R and the outer 2 carbons are
    CH3R). This can be done in software or directly from the PDB using a
    text editor. Keep in mind, if you're using a text editor to modify
    the PDB file, the properties in the PDB file are associated to their
    column location, not based on spaced or tabs. See [this
    guide](https://www.cgl.ucsf.edu/chimera/docs/UsersGuide/tutorials/pdbintro.html)
    for more information.
      
      
    It's always good practice to visualize your PDB in VMD to ensure
    that the naming and bonding is correct before packing the entire
    system.

    - Verifying Atom Names: Open VMD's visualizer, press `0` and click
      the atom in question. In VMD's console, you should see the
      information of that atom including its name
    - Verifying Bonding: This is unlikely in this scenario, but if atoms
      are too close to each other, VMD will assume there is a bond
      between the two atoms. For small molecules/systems, you can
      manually remove the bonds using the top menu in VMD main with
      `Mouse > Add/remove bonds`. This can be very arduous for larger
      systems. A better method is to use
      `set sel [atomselect top $ATOMNAME]` where `$ATOMNAME` is one of
      the atom's name in the erroneous bond, then use
      `$sel set radius $R` where `$R` is the Van der Waals radius VMD
      uses to consider for bonds. Then use `mol bondsrecalc top` to
      recalculate bonds. This is more guess and check to get the correct
      radii, but it will be applied to all atoms with that name in your
      system.
2.  Next, we'll solvate a box using Packmol. Packmol has many good
    tutorials and documentation on it's
    [website](https://m3g.github.io/packmol/). Currently, the Julia
    version is recommended as it's cross-platform, allows for scripting
    through Julia, and doesn't require compilation. Installation
    instructions for Packmol.jl can be found
    [here](https://github.com/m3g/Packmol.jl?tab=readme-ov-file#packmol).
    In this simulation, we'll be using Packmol to pack a box with the
    appropriate number of butane molecules.
      
      
    To find the correct simulation box, we need to consider the cut-off
    distance for our force field. In TraPPE the cut-off distance is 14
    angstroms. This means that, at minimum, the box lengths should be
    twice. Here, we'll use a box of 30x30x30 angstroms. We can next
    calculate the number of butane molecules required to attain the
    correct bulk density which can be found from the [NIST
    website](https://webbook.nist.gov/cgi/cbook.cgi?ID=106-97-8). With
    the correct density at 300 K and 1 ATM and the simulation box
    volume, we find that the total number of butane molecules needed is
    160 molecules. Next, we can make the following Packmol input file
    called `bulk-butane.inp`

        tolerance 2.0

        output bulk-butane.pdb
        seed -1
        filetype pdb
        avoid_overlap yes
        nloop 1000
        add_box_sides 2
        randominitialpoint

        structure butane.pdb
            number 160
            inside box 0. 0. 0. 28. 28. 28.
        end structure

    The important commands in this input file is the following

    - output: designates the output file name
    - add_box_sides: Length added to each side of the box
    - structure: The file location of the molecule being packed into the
      box
    - number: Number of molecules packed into the box
    - inside box: The box the molecules are being packed into. First 3
      values are xlo, ylo, and zlo and the final 3 are xhi, yhi, and
      zhi.

    It is important to note that Packmol does not consider periodic
    boundary conditions when packing the box. This can cause close
    contact issues when packing the entire 30x30x30 box. This is why we
    only pack a box of 28x28x28 and then add an additional 2 angstroms
    to the unit cell afterwards
3.  Pack the box with the input script in the Julia console by running
    the two commands
    ``` bash
    using Packmol
    run_packmol()
    ```
4.  Again, with this PDB, check the structure and ensure that there are
    no erroneous bonds. While it may look visually correct, the
    following section will describe other methods aside from visual to
    determine the accuracy of the system set up

#### Generating a LAMMPS topology file from a PDB

1.  To generate the LAMMPS topology file, we will be using VMD which can
    be downloaded at its
    [website](https://ks.uiuc.edu/Development/Download/download.cgi?PackageName=VMD).
    Load the PDB generated by Packmol into VMD by dragging and dropping
    the file into the visualization window or going through File \> New
    Molecule.
2.  In VMD's console, use the following commands to generate the
    topology file
        mol bondsrecalc top
        topo retypebonds
        topo guessangles
        topo guessdihedrals
        mol reanalyze top
        topo writelammpsdata topology.in

    `mol bondsrecalc top` recalculates bonds, `topo retypebonds` renames
    bond types based on the LAMMPS topology file naming scheme,
    `topo guessangles` and `topo guessdihedrals` generate the angle and
    dihedral types based on the bonding in LAMMPS, `mol reanalayze top`
    will reanalyze the topology based on the new bonds, angles, and
    dihedrals, and lastly `topo writelammpsdata topology.in` will
    generate the LAMMPS topology file with `topology.in` as its output
    name
3.  Opening the topology file, we see a header section describing the
    total number of atoms, bonds, angles, dihedrals, how many types of
    each, and box parameters. Below that we see the Coeffs sections,
    which describe the coefficients for each of the parameters. And
    below all those sections is the Bonds, Angles, and Dihedral
    sections, which says which atoms are connected and what type they
    are. Again, we should verify that the topology generated is indeed
    correct. We can now do this numerically. This is a system solely
    comprising of butane which is 4 atoms, 3 bonds, 2 angles, and 1
    dihedral. Thus we can verify that there are no erroneous bonds by
    calculating the expected number of bonds in the entire system (160
    \* 3 = 480). This can be repeated with the remaining geometries.
    Additionally, we can look at the bonds section in LAMMPS. We expect
    2 bond types, CH2R-CH2R and CH2R-CH3R. If we see CH3R-CH3R, we know
    that two terminal carbons are bonding when they should not.
4.  Next, we can add the pair, bond, angle, and dihedral coefficients
    from the appropriate [TraPPE papers](http://trappe.oit.umn.edu/).
    Keep in mind that for TraPPE, values are given in units of K, so
    conversion to kcal/mol is required. Additionally, TraPPE uses fixed
    bond lengths but no harmonic potential is given. The equilibrium
    bond length can be used from TraPPE and the harmonic potential is
    acquired from a compatible force field (usually OPLS, AMBER, or
    CHARMM). TraPPE uses harmonic potentials for
    [bond](https://docs.lammps.org/bond_harmonic.html) and
    [angle](https://docs.lammps.org/angle_harmonic.html) styles and
    [fourier](https://docs.lammps.org/dihedral_fourier.html) for
    dihedrals. For non-bonded interactions, TraPPE uses
    [lj/cut/coul/long](https://docs.lammps.org/pair_lj_long.html). These
    styles will be touched upon more in the input file section, but the
    functional forms are necessary to know how to input the appropriate
    coefficients. The coefficients sections should look like the
    following:
         Pair Coeffs

         1 0.382 3.95 # CH2R
         2 0.195 3.75 # CH3R

         Bond Coeffs

         1 260 1.54 # CH2R-CH2R
         2 260 1.54 # CH2R-CH3R

         Angle Coeffs

         1 124.199 114 # CH3R-CH2R-CH2R

         Dihedral Coeffs

         1 4 0 0 0 0.7055 1 0 0.1355 2 180 1.5725 3 0 # CH3R-CH2R-CH2R-CH3R

         Masses

         1 14.02658 # CH2R
         2 15.03452 # CH3R

    Note, that the masses of CH2R and CH3R are different than the mass
    of a single carbon atom. This is due to the united atom approach
    TraPPPE uses. The CH2R atom's mass is the mass of 1 carbon atom and
    2 hydrogen atoms and likewise with CH3R with 3 hydrogen atoms. With
    this, the topology file is ready to start the simulation.

#### Generating the LAMMPS input file

1.  The LAMMPS input file, usually named `main.lmp` , are the commands
    LAMMPS runs to perform the MD simulations. At its most basic, an
    input file looks like the following but can become more complex
    depending on the need of your system.
        units real

        boundary p p p
        dielectric 1.0
        atom_style full
        pair_style lj/cut/coul/long 14
        pair_modify mix arithmetic
        special_bonds lj 0.0 0.0 0.0 coul 0.0 0.0 0.5
        bond_style harmonic
        angle_style harmonic
        dihedral_style fourier

        read_data topology.in

        thermo 1000
        thermo_style custom step temp etotal ke pe econserve epair emol ebond eangle edihed eimp evdwl ecoul elong etail press

        restart 5000 restart.1 restart.2

        velocity all create 300 2939
        fix NVT all nvt temp 300 300 $(100*dt)

        kspace_style pppm 1e-6
        neighbor 2.0 bin
        neigh_modify delay 0 every 1 check yes once no

        dump movie all atom 500 movie.lammpstrj

        timestep 10
        min_style sd
        minimize 1.0e-9 1.0e-9 200 2000

        run_style verlet

        timestep 1

        run 1000000

    The important functions are as follows by order of occurrence

    - `units real` sets the simulations units
    - `*_style` describes the functional form used for that interaction
      type
    - `thermo` will output the thermodynamic information every N steps
    - `restart` will generate restart files every 5000 steps
    - `velocity all create 300 2939` will initialize the velocity at 300
      K with 2939 as the random seed
    - `fix NVT all nvt temp 300 300 $(100*dt)` will do time integration
      on all atoms and keep thermostated at 300K with a damping time of
      100\*dt time span
    - `minimize` will run a steepest descent minimization on the system
      with the appropriate tolerances
    - `run` will run the simulation for 1,000,000 time steps.
2.  If LAMMPS is compiled with DUMP_EXTRA, it is more space efficient to
    save the movie file as an XTC file rather than a LAMMPS trajectory
    file (.lammpstrj). To do this replace the `dump` command with
    `dump movie all xtc 500 movie.lammpstrj.xtc`. This should be done
    whenever available
3.  If we are beginning to a simulation from a restart file, we'd need
    to make the following changes:
    - Read the restart file instead of the topology file

    <!-- -->

        #read_data topology.in
        read_restart restart.equilN

    - Do not initialize velocities

    <!-- -->

        #velocity all create 300 2939

    - Remove minimization steps

    <!-- -->

        #timestep 10
        #min_style sd
        #minimize 1.0e-9 1.0e-9 200 2000

### NPT TIP4P Water-CO2 Simulations

- Adding charges to atoms in VMD
- SHAKE and RIGID fixes in LAMMPS
- Groups in LAMMPS
- Setting up NPT simulations

#### Setting up the System

In LAMMPS the location of the virtual site for TIP4P water can be
implicitly calculated from the atoms in the water molecule using TIP4P
specific pair styles. Alternatively, an explicit 4-site model can be
used (more similar to GROMACS) but is more computationally expense. In
this tutorial, we will focus on the former implementation, and, as such,
a 3-site water model should be packed into box. More information on the
TIP4P implementation in LAMMPS can be found
[here](https://docs.lammps.org/Howto_tip4p.html).

1.  Acquire a PDB file for both H2O and CO2. Make sure the geometry for
    both molecules are correct by checking the the current bond and
    angle values with the respective equilibrium bond values. As we're
    using fix rigid for CO2, the CO2 geometry will be held as the input
    geometry and large deviations from the equilibrium values can cause
    bad principal moments error in LAMMPS. Be sure to name the similar
    atoms the same as this is very important in LAMMPS. In this case,
    name the atoms OW, HW, CO, and OC for oxygen in water, hydrogen in
    water, carbon in CO2, and oxygen in CO2 respectively. Again, keep in
    mind that for PDBs spacing matters. Lastly, for TIP4P water, ensure
    that the PDB is set up such that the oxygen atom is first and the
    hydrogen atoms are after it.
2.  With the methods described in the previous tutorial, generate a
    40x40x40 box containing 1,700 water molecules and 1 CO2 molecule
    using packmol. For this simulation, we will be purposefully setting
    the density slightly less than the expected bulk density for the NPT
    simulations (~0.8 g/ml). Visually check for any erroneous bonds in
    VMD, though it may be difficult to see with a system this dense.
3.  To generate the LAMMPS topology file with VMD, rather than inputting
    the commands in the VMD command line as we did previously, we will
    generate a `.tcl` file. This is helpful if you're generating
    multiple of the same system or troubleshooting a system. With your
    favorite text editor, generate a new file called `H2OCO2.tcl` with
    the following lines
        set sel [atomselect top "name OW"]
        $sel set radius 1.4 
        $sel set charge -1.040

        set sel [atomselect top "name HW"]
        $sel set charge 0.520

        set sel [atomselect top "name CO"]
        $sel set charge 0.700

        set sel [atomselect top "name OC"]
        $sel set charge -0.350

        mol bondsrecalc top
        topo retypebonds
        topo guessangles
        topo guessdihedrals
        mol reanalyze top
        topo writelammpsdata topology.in

    To break this down, the `set sel` sections set the variable `sel` to
    the desired atoms based off the `atomselect` conditions. For OW, I
    found that I had a couple erroneous bonds between other OW atoms. To
    correct this, I reduced the Van der Waals radius to 1.4 using
    `$sel set radius` (You may need to vary this number until you get
    the correct bonding in your system). Then, the charge is set of OW
    based on the TIP4P force field using `$sel set charge`. The final
    section of the script is the same as in the butane tutorial.
4.  In the VMD console, change directories to the location of
    `H2OCO2.tcl` and use the following command to run the script
    `source H2OCO2.tcl`. Once the topology file is generated, double
    check the total number of bonds and angles to verify there are
    indeed no erroneous bonds.
5.  Add in the force field information for [TIP4P
    water](https://docs.lammps.org/Howto_tip4p.html) and
    [CO2](https://aiche.onlinelibrary.wiley.com/doi/10.1002/aic.690470719).
    It should look like the following:
         Pair Coeffs

         1 0.0537 2.8 # CO
         2 0 0 # HW
         3 0.157 3.050 # OC
         4 0.1550 3.1536 # OW

         Bond Coeffs

         1 5000 1.16 # CO-OC
         2 0 0.9572 # HW-OW

         Angle Coeffs

         1 0 104.52 # HW-OW-HW
         2 500 180 # OC-CO-OC

    In this case, we see a couple things different between CO2 and H2O
    even though they're both being held rigid. This is due to how each
    one is kept rigid. CO2 will be kept rigid using
    [rigid/small](https://docs.lammps.org/fix_rigid.html) while H2O uses
    the [shake algorithm](https://docs.lammps.org/fix_shake.html). While
    the harmonic potentials are unnecessary for both during the
    simulation, during minimization, shake will place a large harmonic
    potential on the bond/angle in question while rigid will resort to
    the input harmonic potential.

#### NVT and NPT Input Files

Though we plan to do NPT simulations, it's best to run NVT simulations
first to help relax the system and ensure that there are no close
contacts in due to how the system is packed. Close contacts would
generate a very large force, which would lead to a large volume change.
This quick volume change could lead to a 2-phase system which would be
extremely difficult/impossible to fix.

1.  Generate a new `main.lmp.NVT` file to initialize the system. It
    should look like the following
        units real

        boundary p p p
        dielectric 1.0
        atom_style full
        pair_style lj/cut/coul/tip4p 4 2 2 1 0.15 14
        pair_modify mix arithmetic
        special_bonds lj 0.0 0.0 0.0 coul 0.0 0.0 0.5
        bond_style harmonic
        angle_style harmonic
        dihedral_style None

        read_data topology.in

        group water type 4 2
        group CO2 type 1 3

        thermo 1000
        thermo_style custom step temp etotal ke pe econserve epair emol ebond eangle edihed eimp evdwl ecoul elong etail press

        restart 5000 restart.1 restart.2

        variable systemp equal 300

        velocity all create ${systemp} 2939
        fix NVT water nvt temp ${systemp} ${systemp} $(100*dt)
        fix rigidCO2 CO2 rigid/nvt/small molecule temp ${systemp} ${systemp} $(100*dt)

        fix rigidwater water shake 0.0001 20 1000 b 39 a 58

        kspace_style pppm/tip4p 1e-6
        neighbor 2.0 bin
        neigh_modify delay 0 every 1 check yes once no

        dump movie all atom 500 movie.lammpstrj

        timestep 10
        min_style sd
        # fix noForce CO2 setforce 0 0 0
        minimize 1.0e-9 1.0e-9 200 2000

        # unfix noForce

        run_style verlet

        timestep 1

        run 1000000

    You'll see a few minor differences here. The first is that the
    `pair_style` has more inputs which are the OW atom type, HW atom
    type, HW-OW bond type, HW-OW-HW angle type, OW-M distance, and the
    cutoff distance. Next, we have groups for water and CO2 which have
    been chosen through the atom types. We have set a new variable
    `systemp` as we have it in multiple places now. For the velocity
    initalization, all atoms are still considered, but the time
    integration is different. The nvt fix only time integrates water
    while the rigid/nvt/small fix time integrates CO2. This is because
    the rigid fixes in LAMMPS does the time integration while shake does
    not. If we had `fix NVT all nvt` then CO2 would be time integrated
    twice. The shake fix does not do time integration, so the time
    integration for water needs to be handled by the nvt fix. The shake
    fix afterwards keeps water rigid by returning the bonds/angles to
    their equilibrium distance at the start of each frame. Lastly, the
    `kspace_style` needs to be modified to account for the virtual sites
    on water.
2.  Another method to keep the CO2 rigid during minimization is by
    setting the forces on CO2 to 0 during the minimization and then
    removing the fix afterwards. These are shown above but are commented
    out. This may not be useful in this case as it will not move CO2 if
    it's configuration is bad, but is useful for rigid frameworks.
3.  Create a symbolic link for `main.lmp` to `main.lmp.NVT` by using the
    command `ln -sf main.lmp.NVT main.lmp`. This ensures the input files
    can run the correct script and we don't have to worry about renaming
    the `main.lmp.NVT` file. Afterwards, you can run the job.
4.  Make an NPT input file by doing `cp main.lmp.NVT main.lmp.NPT` and
    edit it to the following
        units real

        boundary p p p
        dielectric 1.0
        atom_style full
        pair_style lj/cut/coul/tip4p 4 2 2 1 0.15 14
        pair_modify mix arithmetic
        special_bonds lj 0.0 0.0 0.0 coul 0.0 0.0 0.5
        bond_style harmonic
        angle_style harmonic
        dihedral_style None

        read_restart restart.X

        group water type 4 2
        group CO2 type 1 3

        thermo 1000
        thermo_style custom step temp etotal ke pe econserve epair emol ebond eangle edihed eimp evdwl ecoul elong etail press

        restart 5000 restart.1 restart.2

        variable systemp equal 300
        variable syspress equal 1

        # velocity all create ${systemp} 2939
        fix NPT water npt temp ${systemp} ${systemp} $(100*dt) iso ${syspress} ${syspress} $(1000*dt)
        fix rigidCO2 CO2 rigid/npt/small molecule temp ${systemp} ${systemp} $(100*dt) iso ${syspress} ${syspress} $(1000*dt)

        fix rigidwater water shake 0.0001 20 1000 b 39 a 58

        kspace_style pppm/tip4p 1e-6
        neighbor 2.0 bin
        neigh_modify delay 0 every 1 check yes once no

        dump movie all atom 500 movie.lammpstrj

        # timestep 10
        # min_style sd
        # fix noForce CO2 setforce 0 0 0
        # minimize 1.0e-9 1.0e-9 200 2000

        # unfix noForce

        run_style verlet

        timestep 1

        run 1000000

    Many of the changes are similar to the changes described in the
    previous tutorial, but one main change is the npt fix, applying the
    pressure on all 3 faces of the simulation cell (`iso`).
5.  Again create a symbolic link for `main.lmp` to `main.lmp.NPT` by
    using the command `ln -sf main.lmp.NPT main.lmp` and run the job.

## Visualizing Trajectories with VMD

1.  Open VMD and visualize the PDB of your system.
2.  In the VMD Main window, right click you system and select 'Load Data
    into Molecule'.
3.  A new window will pop up and you can drag and drop the file into the
    'filename' text box and load the trajectory. It's important to note
    that for the most current stable release (ver. 1.9.3 as of
    13-03-2024), windows only has a 32-bit version. This constrains the
    maximum file size for a trajectory to ~4Gb. If your trajectory is
    much larger than that, use the 'stride: N' option to take every N
    frames.

## Troubleshooting Methods/Tips

- A common error is `Out of range atoms - cannot compute PPPM` or
  `Lost atoms: ...`. This usually comes from close contacts between
  atoms sometime during the simulation, leading to large forces between
  atoms, then the atoms moving too quickly.

:\* Double check the single-point energy of the initial system by doing
`run 0` and look at the potential/total energies. If they are
unrealistically large it may be due to bad packing leading to close
contacts. Try decreasing stopping tolerance for relative energy and/or
increasing the number of steps. Alternatively, you can remove the
stopping tolerance so that it will always run the full number of steps
`minimize 0 0 200 2000`.

:\* Alternatively, two atoms in the simulation may be getting too close
to each other during the simulation. A good way to see which atoms are
coming too close is to add the line `delete_atoms overlap 0.1 all all`
in your \<main.lmp\> file. This will delete any atoms that are closer
than 0.1 angstroms away. Note, that usually this command deletes the
atom in the first group, but as both groups are all, it will select an
atom at random. You may want to run this a couple of times so you can
find the atom indices of both atoms if it unclear in the trajectory.

- While it is more space efficient to save movie files as `.xtc` files,
  `.xtc` files only contain atom positions. As close contact errors lead
  to large velocities, it is often useful to save the atom velocities as
  well. This can be done by using the lammpstrj file with the line
  `dump movie all custom 500 movie.lammpstrj id mol type xu yu zu vx vy vz`.
  When loading the trajectory in VMD, you can choose to color the atoms
  based on velocity to find which atom/atoms are moving too quickly

<!-- -->

- A common method for troubleshooting is to set `thermo 1`,
  `dump movie all custom 1 movie.lammpstrj id mol type xu yu zu vx vy vz`,
  `run 1000`. This is useful for when the simulation fails in the first
  couple hundred/thousand steps as it shows the energy at every frame
  and the visualization has very high resolution. However, if the error
  is occurring on frame 100,000, it becomes storage inefficient to save
  movie frames at every step. Instead, you can save the last N frames of
  you simulation by adding the following lines

<!-- -->

    dump movie all custom 1 movie.*.lammpstrj id mol type xu yu zu vx vy vz
    dump_modify movie maxfiles N

  
Then run `cat movie.*.lammpstrj > movie.all.lammpstrj` to combine the
movie files into a single file and then visualize as you normally would

\<!--
