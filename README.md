# PyTopol

Reza Salari - [Brannigan Lab](http://branniganlab.org)

### Introduction 

**PyTopol** provides utilities to convert certain molecular topologies to 
each other. Currently it supports converting CHARMM `psf` files (`xplor` format) 
to GROMACS topology format through the `psf2top.py` utility. If you'd like 
to use GROMACS topologies in NAMD, plese see 
[here](http://www.ks.uiuc.edu/Research/namd/2.9/ug/node14.html).

PyTopol follows a different approach than the force-field conversion tools and 
is intended to convert the full-topology of a molecule from one format to
another. This allows conversion of custom-parameterized topologies across
MD packages.


### Current stage
PyTopol is currently in *alpha stage*. The results for several 
systems are shown below which are encouraging. However, before using it for
production simulations, more testing is needed. 

Current version is 0.1.0. All 0.1.x versions will be considered alpha.


### Feedback
If you have a question, found a bug or have a suggestion, please submit it
[here](http://github.com/resal81/pytopol/issues). 

### License
PyTopol is licensed under [GNU GPLv3](http://www.gnu.org/licenses/gpl.html). 



## Quickstart

#### Method 1 - using `pip` (python package installer)

    $ pip install pytopol 
    $ psf2top.py -p psffile -c charmm_prm1 [charmm_prm2 ...] [-v]

Note that in this method `psf2top.py` will be installed in the `bin` directory 
of your Python distribution. Make sure this directory is in your path.

#### Method 2 - clone the git repo 

    $ git clone https://github.com/resal81/PyTopol.git
    $ cd pytopol 
    $ export PYTHONPATH=`pwd`:$PYTHONPATH
    $ cd scripts 
    $ python psf2top.py -p psffile -c charmm_prm1 [charmm_prm2 ...] [-v]


## Example run
For a protein+lipid `psf` system, the GROMACS topology can be generated by the
  follwing command. This assumes that name of the `psf` file is `system.psf` and
  you have the necessary parameter files in the same directory.

    $ psf2top.py -p system.psf -c par_all36_prot.prm par_all36_lipid.prm

You can add `-v` for debug information.


## Notes 

* `psf2top.py` converts the `psf` file to GROMACS topology format. It creates one
 `top.top` plus one or more `itp` files. Each segment in the `psf` file is converted 
 to one `itp` file. 

* `psf2top.py` only accepts `xplor` formatted `psf` files (used for NAMD), where the columns in the
  atoms section of the `psf` file are separated by at least one space. If this
  is the case for your `psf` file, make sure there is a `NAMD` keyword in the
  first line of the `psf` file.

* You should provide all of CHARMM parameter files for the atoms in your `psf`
  file. These correspond to the parameter files that you use for your system
  when running the NAMD simulation. 

* The CHARMM stream files (with `str` suffix) are not supported. These are files that typically
  include both topology and parameters for a molecule. For these cases put the
  parameter section in a separate file before giving it to `psf2top.py`.

* For the CHARMM parameter files, the `NBFIX` and `HBOND` sections are
  ignored.

* If you use [CHARMM-GUI](http://www.charmm-gui.org/?doc=input/membrane) to generate inputs,
  make sure to use the generated `xplor` format for the `psf` file.

* Remove water molecules from the `psf` file before conversion. The tests for water topology
  conversion do not quite pass yet (see below). You can use `editconf` and `genbox` to add
  water to the system later on in GROMACS.



## Test systems 

I used `psf2top.py` to convert several `psf` files to GROMACS format 
  and then compared the resulting single-point energies using NAMD 2.9 and 
  GROMACS 4.5.7. 


#### Tests setup

GROMACS simulation was run using:

    $ grompp -f mdpfile -c pdbfile -p top.top -o topol.tpr
    $ mdrun -nt 1 -s topol.tpr -rerun pdbfile -g gromacs.log

NAMD simulation was run using:

    $ namd2 +p1 conf

*GROMACS MDP file*
```
integrator    = md
nsteps        = 0
nstlog        = 1
nstlist       = 0
ns_type       = simple
rlist         = 0
coulombtype   = cut-off
rcoulomb      = 0
rvdw          = 0
pbc           = no
```

*NAMD configuration file*
```
structure          [psffile]
coordinates        [pdbfile]

paratypecharmm      on
parameters         [parameter files]
exclude             scaled1-4
1-4scaling          1.0

switching           off
cutoff              1000
pairlistdist        1000
timestep            1.0
outputenergies      1
outputtiming        1
binaryoutput        no

outputname          namd
dcdfreq             1
temperature         300
run                 0
```


#### Results of select systems

Notes:

*  For these tests, first the `psf` file was created and then converted to
  GROMACS format.
*  The `psf` files for these systems are in `test/systems`.
*  For more info on automation of these tests, see `test/systems/README.md`.
*  Energies are in kcal/mol.
*  `Diff` is the absolute difference (i.e. `gromacs-namd`) and `%Diff` is 
   `(gromacs-namd)/namd * 100`.

POPC membrane in vaccum (CHARMM 36, 74 POPC, 9916 atoms)

    21                 NAMD      GROMACS        Diff      %Diff
          bond        134.2        134.2        -0.0        0.0
         angle       1611.9       1611.9        -0.0        0.0
      dihedral       3737.3       3737.3         0.0        0.0
      improper         10.3         10.3        -0.0        0.0
          coul      -2377.2      -2377.1         0.0        0.0
           vdw      -2663.9      -2663.9        -0.0        0.0
 
DOPC membrane in vaccum (CHARMM 36, 76 DOPC, 10488 atoms)

    22                 NAMD      GROMACS        Diff      %Diff
          bond        143.9        143.9        -0.0        0.0
         angle       1591.2       1591.2        -0.0        0.0
      dihedral       4376.7       4376.7        -0.0        0.0
      improper         11.7         11.7        -0.0        0.0
          coul      -2254.6      -2254.6         0.0        0.0
           vdw      -2896.1      -2896.0         0.0        0.0
 
1LYZ in vacuum (CHARMM 27+CMAP, 129 residues, 1966 atoms)

    31                 NAMD      GROMACS        Diff      %Diff
          bond      11807.4      11807.3        -0.0        0.0
         angle       5946.3       5946.3        -0.0        0.0
      dihedral        781.8        781.9         0.1        0.0
      improper        578.2        578.2         0.0        0.0
          coul      -3221.8      -3221.8         0.0        0.0
           vdw    1741437.8    1741435.9        -1.9       -0.0
 
Cholestrol in vacuum (CHARMM 36, 74 atoms)

    41                 NAMD      GROMACS        Diff      %Diff
          bond          7.9          7.9        -0.0        0.0
         angle         58.6         58.6         0.0        0.0
      dihedral         22.9         22.9         0.0        0.0
      improper          0.0          0.0         0.0        0.0
          coul        -57.9        -57.9        -0.0        0.0
           vdw          5.0          4.6        -0.3       -7.0
 
10 water molecule (CHARMM 27, 30 atoms)

    43                 NAMD      GROMACS        Diff      %Diff
          bond          7.8          7.9         0.1        0.9
         angle          1.2          1.2        -0.0        0.0
      dihedral          0.0          0.0         0.0        0.0
      improper          0.0          0.0         0.0        0.0
          coul          0.4          0.4        -0.0        0.0
           vdw         -0.2         -0.2         0.0        0.0
 
Water box + 4 ions (CHARMM 27, 3736 atoms)

    44                 NAMD      GROMACS        Diff      %Diff
          bond       1161.5       1168.8         7.3        0.6
         angle        262.1        262.1        -0.0        0.0
      dihedral          0.0          0.0         0.0        0.0
      improper          0.0          0.0         0.0        0.0
          coul     -11830.1     -11830.0         0.0       -0.0
           vdw       1096.2       1096.2        -0.0        0.0
 

## Contribution
There are many ways you can help to improve **PyTopol**:

* Convert your `psf` files to GROMACS format and compare NAMD and GROMACS energies.
  Use the issues page to let me know of the potential discrepancies.

* Run simulations using the generated topologies and see if the results make sense.

* Fork this repo, implement improvements and send me a pull request.


## ToDo 
* Why the absolute difference in bonding energies of water box is rather big?
* More tests

## Acknowledgement 
* Energy conversion factors are from `charmm2gromacs-pvm.py` script by Par Bjelkmar, 
Per Larsson, Michel Cuendet, Berk Hess and Erik Lindahl.

## Similar tools
* [charmm2gromacs](http://www.gromacs.org/@api/deki/files/185/=charmm2gromacs-pvm.py) 
  is a tool for converting CHARMM force field to GROMACS.
* [SwissParam](http://www.swissparam.ch/) converts `mol2` format to CHARMM and GROMACS
  formats.



