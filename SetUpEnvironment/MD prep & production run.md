# Note

This page aims to provide examples for MD preparation and production run commands. Different users would have slight variations in their commands. New users can find the most suitable one to proceed.

# Cheng Zhang's version
* Note
  
The command is mostly adapted from Justin Lemkul's [lysozyme tutorial](http://www.mdtutorials.com/gmx/lysozyme/index.html)
 
The command below is used for this [paper](https://www.sciencedirect.com/science/article/pii/S2001037021001860)

 
### Run the command below on Ubuntu
Use https://server.poissonboltzmann.org/pdb2pqr to determine the protonation states

```
gmx pdb2gmx  -f Fab.pdb -o Fab_processed.gro -water spce  -inter  -ignh -merge interactive
gmx editconf -f Fab_processed.gro -o Fab_newbox.gro -c -d 1.0 -bt cubic
gmx solvate -cp Fab_newbox.gro -cs spc216.gro -o Fab_solv.gro -p topol.top
gmx grompp -f ions.mdp -c Fab_solv.gro -p topol.top -o ions.tpr -maxwarn 1 
gmx genion -s ions.tpr -o Fab_solv_ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.05
```
 
 
* From now onwards, all the following are done on Myriad

  Get the mdp files from Justin's tutorial:
- nvt.mdp (change temperature, gen_temp, ref_t))
- npt.mdp (change temperature, ref_t)
- md.mdp (change temperature, ref_t)
 
 
### md.mdp (modification)
nsteps        = 1000000000000    ;  just set it to be quite long, and it can be stopped if e.g. 100 ns is achieved

nstxout                = 50000        ; save coordinates every 100 ps

nstvout                = 0            ; NOT save velocities

compressed-x-grps   = Protein   ; replaces xtc-grps, only save the protein coordinates, as I am not interested in the water/ions coordinates, which could save disc space
 
 
 
### job_em_nvt_npt_prod_grompp.sh
```
#!/bin/bash -l
#$ -S /bin/bash
#$ -l h_rt=4:00:0
#$ -l mem=4G
#$ -l tmpfs=15G
#$ -N em_nvt_npt_production-grompp
#$ -pe mpi 4
#$ -cwd 
 
module load gcc-libs
module load compilers/intel/2018/update3
module load mpi/intel/2018/update3/intel
module load gromacs/2019.3/intel-2018
 
gmx grompp -f minim.mdp -c Fab_solv_ions.gro -p topol.top -o em.tpr
gerun mdrun_mpi -v -deffnm em
 
gmx grompp -f nvt.mdp -c em.gro -p topol.top -o nvt.tpr -r em.gro 
gerun mdrun_mpi -deffnm nvt
 
gmx grompp -f npt.mdp -c nvt.gro -t nvt.cpt -p topol.top -o npt.tpr -r nvt.gro
gerun mdrun_mpi -deffnm npt
 
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr -r npt.gro
```
 
 
### job_conti.sh (Gromacs 2019 version. Used to append jobs one after another)
```
#!/bin/bash -l
#$ -S /bin/bash
#$ -l h_rt=01:00:0
#$ -l mem=2G
#$ -l tmpfs=15G
#$ -N md_0_1
#$ -pe mpi 12
#$ -cwd 
#$ -j y
#$ -o output.log
 
module load gcc-libs
module load compilers/intel/2018/update3
module load mpi/intel/2018/update3/intel
module load gromacs/2019.3/intel-2018
 
gerun mdrun_mpi -deffnm md_0_1 -cpi -append
```
 
 
### job_conti.sh (Gromacs 2021 version. Used to append jobs one after another)
```
#!/bin/bash -l
#$ -S /bin/bash
#$ -l h_rt=02:00:0
#$ -l mem=2G
#$ -l tmpfs=15G
#$ -N md_0_1
#$ -pe mpi 12
#$ -cwd 
#$ -j y
#$ -o output.log
 
module -f unload compilers mpi gcc-libs
module unload gromacs
module load beta-modules
module load gcc-libs/7.3.0
module load compilers/gnu/7.3.0
module load mpi/openmpi/3.1.4/gnu-7.3.0
module load python3
module load gromacs/2021.2/gnu-7.3.0
 
FILE=md_0_1.cpt
 
if test -f "$FILE"
then
   gerun mdrun_mpi -v -deffnm md_0_1 -cpi -append -maxh 2
else
   gerun mdrun_mpi -v -deffnm md_0_1 -maxh 2
fi
```
 
 
 
### job_conti_batch.sh (type "./job_conti_batch.sh" without quote on putty)
```
#!/bin/bash
 
# v-- stop if there's a problem with any of the submits
set -o errexit
 
echo "Submitting set 1..."
 
chain_1_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat1/; qsub -terse job_conti.sh)
chain_2_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat2/; qsub -terse job_conti.sh)
chain_3_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat3/; qsub -terse job_conti.sh)
 

for i in 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 
do
  echo "Submitting set $i..."
  
  chain_1_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat1/; qsub -terse -hold_jid $chain_1_jid job_conti.sh)
  chain_2_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat2/; qsub -terse -hold_jid $chain_2_jid job_conti.sh)
  chain_3_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat3/; qsub -terse -hold_jid $chain_3_jid job_conti.sh)
 
  done
 
echo "Done."
``` 
 
 
### job_analysis.sh
```
#!/bin/bash -l
#$ -S /bin/bash
#$ -l h_rt=01:00:0
#$ -l mem=2G
#$ -l tmpfs=15G
#$ -N md_0_1
#$ -pe mpi 1
#$ -cwd 
#$ -j y
#$ -o output.log
 
module load gcc-libs
module load compilers/intel/2018/update3
module load mpi/intel/2018/update3/intel
module load gromacs/2019.3/intel-2018
 
echo 1 1 | gmx trjconv -s ../md_0_1.tpr -f ../md_0_1.xtc -o md_0_1_fit-rot_trans.xtc -ur compact -fit rot+trans -e 100000
echo 1 | gmx gyrate -s ../md_0_1.tpr -f md_0_1_fit-rot_trans.xtc -o gyrate.xvg
echo 1 1 | gmx rms -s ../md_0_1.tpr -f md_0_1_fit-rot_trans.xtc -o rmsd.xvg -tu ns
echo 1 | gmx rmsf -s ../md_0_1.tpr -f md_0_1_fit-rot_trans.xtc -o rmsf.xvg -oq bfac.pdb -res
```
 
