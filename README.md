# Guide to Install and Run Slurm
------------------------
- [x] Install Slurm
- [x] Running Nwchem Using Slurm
- [x] Running xtb Using Slurm
- [ ] Running DFTB+ Using Slurm

## Install
1. Install slurmd slurmctld
```
sudo apt install -y slurmd slurmctld
```
2. Make slurm.conf
```
sudo nano /etc/slurm-llnl/slurm.conf
```
3. Copy and Paste into slurm.conf
```
ControlMachine=//USER #<YOUR-HOST-NAME>
MpiDefault=none
ProctrackType=proctrack/pgid
ReturnToService=1
SlurmctldPidFile=/run/slurmctld.pid
SlurmdPidFile=/run/slurmd.pid
SlurmdSpoolDir=/var/lib/slurm-llnl/slurmd
SlurmUser=slurm
StateSaveLocation=/var/lib/slurm-llnl/slurmctld
SwitchType=switch/none
TaskPlugin=task/none
FastSchedule=1
SchedulerType=sched/builtin
SelectType=select/cons_res
SelectTypeParameters=CR_Core
AccountingStorageType=accounting_storage/none
ClusterName=//USER #<YOUR-HOST-NAME>
JobAcctGatherType=jobacct_gather/none
SlurmctldLogFile=/var/log/slurm-llnl/slurmctld.log
SlurmdLogFile=/var/log/slurm-llnl/slurmd.log
NodeName=//USER CPUs=24 Sockets=1 CoresPerSocket=12 ThreadsPerCore=2 State=UNKNOWN
PartitionName=long Nodes=//USER Default=YES MaxTime=INFINITE State=UP
```
4. Edit the following:
```
ControlMachine=//USER
ClusterName=//USER
NodeName=//USER
Nodes=//USER
CPUs=24
Sockets=1
CoresPerSocket=12
ThreadsPerCore=2
```
5. Delete and change //USER with your username, username can be checked by command:
```
hostname
```
6. CPUs, Sockets, CoresPerSocket, ThreadsPerCore can be checked by command:
```
lscpu
```
7. Save and Exit
```
ctrl + o
```
```
enter
```
```
ctrl + x
```
8. Start Slurm
```
sudo systemctl start slurmctld && sudo systemctl start slurmd
```
```
sudo scontrol update nodename=//USER state=idle
```
```
change //USER with username
```

## Running Nwchem 7.0.2 using Slurm - Example H2O
----------------------------------
1. Make and go to nwchem_slurm directory
```
mkdir nwchem_slurm && cd nwchem_slurm
```
2. Make h2o.nw
```
nano h2o.nw
```
3. Edit h2o.nw - copy and paste
```
 start h2o 
 title "Water in 6-31g basis set" 

 geometry units au  
   O      0.00000000    0.00000000    0.00000000  
   H      0.00000000    1.43042809   -1.10715266  
   H      0.00000000   -1.43042809   -1.10715266 
 end  
 basis  
   H library 6-31g  
   O library 6-31g  
 end
 task scf
 ```
4. Save and Exit
```
ctrl + o
```
```
enter
```
```
ctrl + x
```
5. Make run.sh
```
nano run.sh
```
6. Copy and Paste script into run.sh
```
#!/bin/bash
#
#SBATCH --comment=nwchem
#SBATCH --nodes=1
#SBATCH --ntasks=8 
#SBATCH --job-name=nwchem
#SBATCH --export=ARMCI_DEFAULT_SHMMAX=8192
#SBATCH --output=h2o.out

ulimit -l unlimited 

module purge
module load apps/nwchem/7.0.2

mpirun nwchem h2o.nw
```
7. Save and Exit
```
ctrl + o
```
```
enter
```
```
ctrl + x
```
8. Run calculation
```
sbatch run.sh
```
To check if calculation is still runnig use command ```squeue```. To cancel calculation use command ```scancel jobid```

9. Open output
```
nano h2o.out
```

## Running xtb using Slurm - Example ethane
----------------------------------
1. Install Openbabel
```
sudo apt install openbabel -y
```
2. Make and edit .smi file using openbabel in xtb_ethane - for example ohess ethane
```
mkdir xtb_ethane_slurm && cd xtb_ethane_slurm/
```
```
nano geom.smi
```
Type:
```
CC
```
3. Save and Exit
```
ctrl + o
```
```
enter
```
```
ctrl + x
```
4. Convert .smi to .xyz
```
obabel geom.smi -O geom.xyz --gen3D
```
5. make run.sh
```
nano run.sh
```
6. edit run.sh
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=168:0:0
export OMP_NUM_THREADS=8
xtb geom.xyz --ohess -P $OMP_NUM_THREADS > xtb.out
```
Change export OMP_NUM_THREADS as needed, in this case we will use 8 threads.
7. Save and Exit
```
ctrl + o
```
```
enter
```
```
ctrl + x
```
8. Run XTB
```
sbatch run.sh
```
