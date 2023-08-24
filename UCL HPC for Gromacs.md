# UCL HPC module
### 2021.2
```
module -f unload compilers mpi gcc-libs
module unload gromacs
module load beta-modules
module load gcc-libs/7.3.0
module load compilers/gnu/7.3.0
module load mpi/openmpi/3.1.4/gnu-7.3.0
module load python3
module load gromacs/2021.2/gnu-7.3.0
```
 
### 2020.4
```
module -f unload compilers mpi gcc-libs
module load beta-modules
module load gcc-libs/7.3.0
module load compilers/intel/2020/release
module load mpi/intel/2019/update6/intel
module load gromacs/2020.4/intel-2020
```
 
### 2019.3
```
module load gcc-libs
module load compilers/intel/2018/update3
module load mpi/intel/2018/update3/intel
module load gromacs/2019.3/intel-2018
```
 
### 2016.3
```
module load gcc-libs
module load compilers/intel
module load mpi/intel
module unload gromacs
module load gromacs/2016.3/intel-2017-update1
```

### job_conti.sh 
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

# This script is for MD production job submission
 
module load gcc-libs
module load compilers/intel/2018/update3
module load mpi/intel/2018/update3/intel
module load gromacs/2019.3/intel-2018
 
gerun mdrun_mpi -deffnm md_0_1 -cpi -append
```


### job_conti_batch.sh 
```
#!/bin/bash

# This script is for submitting a series of jobs simultaneously to the cluster
 
# v-- stop if there's a problem with any of the submits
set -o errexit
 
echo "Submitting set 1..."
 
chain_1_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat1; qsub -terse job_conti.sh)
chain_2_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat2; qsub -terse job_conti.sh)
chain_3_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat3; qsub -terse job_conti.sh)
chain_4_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat4; qsub -terse job_conti.sh)
chain_5_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat5; qsub -terse job_conti.sh)
chain_6_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat6; qsub -terse job_conti.sh)
 
 
for i in 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 
do
  echo "Submitting set $i..."
  chain_1_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat1; qsub -terse -hold_jid $chain_1_jid job_conti.sh)
  chain_2_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat2; qsub -terse -hold_jid $chain_2_jid job_conti.sh)
  chain_3_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat3; qsub -terse -hold_jid $chain_3_jid job_conti.sh)
  chain_4_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat4; qsub -terse -hold_jid $chain_4_jid job_conti.sh)
  chain_5_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat5; qsub -terse -hold_jid $chain_5_jid job_conti.sh)
  chain_6_jid=$(cd /lustre/home/ucbexxx/Scratch/xxxxxx/repeat6; qsub -terse -hold_jid $chain_6_jid job_conti.sh)
  
done
 
echo "Done."
```
 
