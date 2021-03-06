# OpenFold-amber

A set of scripts using open source softwares that can convert an amino acid sequence into a folded 3D structure using simplistic simulated annealing molecular dynamics simulations and user-defined distance and torsion restraints. Mainly just a python wrapper script that calls AmberTools20 sander to run MD simulations.

### Pre-Reqs:
1. [Python3](https://www.python.org) <br/>
Required non-standard packages: MDAnalysis (version 1.0.0)

2. [AmberTools20](http://ambermd.org/GetAmber.php) <br/>

For a simple-to-install, non-parallelized version of AmberTools, you can use conda ([Miniconda](https://docs.conda.io/en/latest/miniconda.html)):
```bash
# Download miniconda if you don't already have installed.
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# Install miniconda; initialize the miniconda install within your shell during installation
bash Miniconda3-latest-Linux-x86_64.sh
# Add conda-forge to the channel list (it may already be present, but worth checking). 
conda config --add channels conda-forge
# Update to conda-forge versions of packages
conda update --yes --all
# Create a new conda environment named OpenFold-amber
conda create -n OpenFold-amber python==3.8
# Activate the OpenFold-amber environment
conda activate OpenFold-amber
# Install AmberTools20 within the environment
conda install -c conda-forge ambertools=20
# Install MDAnalysis 
conda install MDAnalysis
```

3. Clone or download this git repository to a single location. 

### To Run:
1. Prepare a FASTA/txt file with the amino acid sequence in single letter formatting. 
2. Prepare the 8 column distance restraints file (format below) and 5 column torsion restraints file (format below).
3. Copy the fold_protein.json from the git repository and edit with your parameters (explaination below).
4. Run fold_protein.py:

```bash
export OpenFoldHome=~/Apps/OpenFold-amber	# edit this line with the global location for this cloned git repository
python3 $OpenFoldHome/fold_protein.py fold_protein.json
```

### Input to the Program: fold_protein.json 
1. name: string; an identifier string used in naming of output directory and files, so you can really use any string you want. 
2. fasta_file_path: string; directory path that points to the FASTA file with the to-be folded sequence in single letter format (i.e., "NLYIQWLKDGGPSSGRPPPS").
3. distance_restraints_file_path: string; directory path that points to the distance restraints file in 8 column format.
4. torsion_restraints_file_path: string; directory path that points to the torsion restraints file in 5 column format.
6. simulated_annealing_input_file_path: string; directory path that points to the input file to perform basic the simulated annealing MD sims. with the user defined restraints. General users shouldn't need to change this parameter's value. 
7. tordef_file_path: string; directory path that points to the tordef.lib file needed for AmberTools' makeANG_RST script to work. Users shouldn't need to change this paramter's value. 
8. forcefield: string; file name associated with the leaprc file to be used in AmberTools' tleap to generate the linear 3D structure and respective parameters. Only tested with "leaprc.protein.ff14SB".
9. distance_force_constants: python list of floats; contains the harmonic force constant applied to pairwise atom-atom distance restraints. Units: kcal/(mol·Angstrom)
10. torsion_force_constants: python list of floats; contains the harmonic force constant applied to dihedral atom groups. Units: kcal/(mol·rad)
11. temperatures: python list of floats; contains the maximum temperatures for simulated annealing cycles. Units: K
12. annealing_runs: integer; number of simulated annealing simulations that will be performed. 

>If the length of the force constants and temperatures lists (parameters #9-11) is 1, then the single value is used across all simulated annealing cycles.<br/>
>If the length of these lists is = number of cycles, then the first value is used for the first cycle, the second value for the second cycle, and so on.<br/>
>If the length is > cycles, the extra values are ignored.<br/>
>If the length is < cycles, the first value is used for all cycles.<br/>
>Any of the values in these lists may be 0.0, but it is recommended that temperature values >= 100.0 K.<br/>





>The given example file has what I have found to be good defaults.

**Distance Restraints Format:**

Distance Restraints File should be formatted as a list of restraints, with the following 8 columns:

>atom1_residue_number (int), atom1_residue_name, atom1_name, atom2_residue_number (int), atom2_residue_name, atom2_name, lower_bound_distance (float), upper_bound_distance (float)

For example:

    2   MET   CB    41    ALA   CB    3.81    5.81
    3   PHE   CB    15    GLY   CA    9.4     11.6
    etc

I used make_rst.py (details below) to make restraints. The file uses BioPython and should be easy to modify and use if you are making restraints from an original pdb file. Feel free to create your restraints list in other ways.

**Torsion Restraints Format:**

Similar to Above, but there are 5 columns:

>residue_number (int), residue_name, angle_name, lower_bound (float), upper_bound (float)

For example:
```
1    LYS    PSI    138.7    140.7
2    VAL    PHI    -103.7    -101.7
2    VAL    PSI    113.4    115.4
3    PHE    PHI    -76.1    -74.1
etc
```

**Output:** folded protein in pdb file; note the starting velocities are randomized, so the output is nondeterministic

**Optional Scripts:**
1. make_rst.py <br/>
Makes restraints from original pdb file.
```
python make_rst.py <name of pdb file without the .pdb extension> <distance range float in Angstroms> <angle range float in degrees>
```
2. secstruc.py <br/>
Appends secondary structure restraints to your original restraints file
```
python secstruc.py <sequence> <secondary structure preduction sequence> <distance file> <angle file>
```

2. scores.py <br/>
If you want RMSD and TMScores, the TMScore program (https://zhanglab.ccmb.med.umich.edu/TM-score/) will produce both; scores.py will run the program, parse the info and put it in a seperate file ("scores") for you.
```
python scores.py <TMScore executable> <original pdb file> <your new pdb file>
```
3. metrics.py <br/>
If you run it after you run scores.py, it will append the "scores" text file with data on the on the average difference between correct and new atom distances and torsions (as well as standard deviations). Could be easily modified for other metrics.
```
python metrics.py <original pdb file> <your new pdb file>
```

**Viewing Trajectory:**

The relevant files for viewing trajectory in VMD are "prmtop" (file type: AMBER7 Param) and "siman1.nc", "siman2.nc", etc, up to the number of cycles of simulated annealing (let VMD determine the file type automatically).


**Use:**

Please feel free to use/modify code. Everything is completely open-source. This work was done at the Center for Molecular Biophysics at Oak Ridge National Lab. Contact jesskwoods (at) gmail.com with corrections, questions, comments.
