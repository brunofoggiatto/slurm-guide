# SLURM Installation Guide 

This guide provides a step-by-step process for installing, configuring, and using **SLURM** on Ubuntu 22.04.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Hostname Configuration](#2-hostname-configuration)
3. [SLURM Installation](#3-slurm-installation)
4. [Create Users and Directories](#4-create-users-and-directories)
5. [SLURM Configuration](#5-slurm-configuration)
6. [Munge Key Configuration](#6-munge-key-configuration)
7. [Enable and Start SLURM Services](#7-enable-and-start-slurm-services)
8. [Testing SLURM Installation](#8-testing-slurm-installation)
9. [Troubleshooting](#9-troubleshooting)
10. [Submitting Your First SLURM Job](#10-submitting-your-first-slurm-job)

---

## 1. Prerequisites

Install common tools and build dependencies:

```bash
sudo apt update
sudo apt install -y build-essential manpages-dev vim git
```

---

## 2. Hostname Configuration

Set your hostname:

```bash
sudo hostnamectl set-hostname localhost
```

> **Tip:** Verify `/etc/hosts` contains an entry mapping your IP/127.0.0.1 to the hostname.

---

## 3. SLURM Installation

Update package lists and install SLURM packages, including the documentation and munge.

```bash
sudo apt update
sudo apt install -y slurm-wlm slurm-wlm-doc munge
```

---

## 4. Create Users and Directories

Create the `slurm` user and the directories required by SLURM:

```bash
sudo adduser slurm
sudo mkdir -p /var/spool/slurm-llnl
sudo mkdir -p /var/spool/slurm-llnl/slurmd
sudo mkdir -p /var/log/slurm
sudo chown -R slurm:slurm /var/spool/slurm-llnl /var/log/slurm
```

---

## 5. SLURM Configuration

On Ubuntu 22.04, SLURM configuration files are located under `/etc/slurm/` (not `/etc/slurm-llnl/`).

Create the configuration directory (if it doesn’t exist):

```bash
sudo mkdir -p /etc/slurm
```

Create and edit `slurm.conf`:

```bash
sudo nano /etc/slurm/slurm.conf
```

Paste the following configuration, adjusting `CPUs` and `RealMemory` to match your system:

```conf
ClusterName=localcluster
ControlMachine=localhost

SlurmUser=slurm
SlurmdUser=root

StateSaveLocation=/var/spool/slurm-llnl
SlurmdSpoolDir=/var/spool/slurm-llnl/slurmd
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid
SlurmctldLogFile=/var/log/slurm/slurmctld.log
SlurmdLogFile=/var/log/slurm/slurmd.log

SwitchType=switch/none
MpiDefault=none
SlurmctldPort=6817
SlurmdPort=6818
ProctrackType=proctrack/pgid
ReturnToService=2
SchedulerType=sched/backfill
SelectType=select/cons_res
SelectTypeParameters=CR_Core

NodeName=localhost CPUs=4 RealMemory=8000 State=UNKNOWN
PartitionName=debug Nodes=localhost Default=YES MaxTime=INFINITE State=UP
```

Determine CPU count and memory:

```bash
nproc
free -m
```

> **Example:** if `free -m` shows `total=16036`, that’s 16 GB.

---

## 6. Munge Key Configuration

On Ubuntu 22.04, create the Munge key with `mungekey --create`.

```bash
sudo /usr/sbin/mungekey --create
```

Set ownership and permissions:

```bash
sudo chown munge:munge /etc/munge/munge.key
sudo chmod 400 /etc/munge/munge.key
```

Enable and start the Munge service:

```bash
sudo systemctl enable munge
sudo systemctl start munge
```

Verify Munge by encoding/decoding a test credential:

```bash
munge -n | unmunge
```

Check the key file permissions:

```bash
ls -l /etc/munge/munge.key
# Expected: -r-------- 1 munge munge 1024 <date> /etc/munge/munge.key
```

---

## 7. Enable and Start SLURM Services

Enable SLURM to start on boot and start the daemons now:

```bash
sudo systemctl enable slurmctld
sudo systemctl enable slurmd
sudo systemctl start slurmctld
sudo systemctl start slurmd
```

Check status:

```bash
systemctl status slurmctld slurmd
```

---

## 8. Testing SLURM Installation

Basic cluster information:

```bash
sinfo
```

Run a simple task on localhost:

```bash
srun hostname
```

> Expected output: `localhost`

## 9. Troubleshooting

Tail the SLURM logs to diagnose issues:

```bash
tail -f /var/log/slurm/slurmctld.log
```

```bash
tail -f /var/log/slurm/slurmd.log
```

---

## 10. Submitting Your First SLURM Job

Create a simple job script that sets name, output, tasks, CPUs, memory, and time limit.

Create the file:

```bash
nano first-job.sh
```

Paste the contents:

```bash
#!/bin/bash
#SBATCH --job-name=first-job
#SBATCH --output=first-out.txt
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=500MB
#SBATCH --time=00:05:00

echo "Running on host: $(hostname)"
echo "Date: $(date)"
sleep 30
echo "Finish!"
```

Submit the job:

```bash
sbatch first-job.sh
```

Cancel a running job:

```bash
scancel <JOBID>
```

Check job queue:

```bash
squeue
```

View job output:

```bash
cat first-out.txt
```

List jobs for a specific user:

```bash
squeue -u <USER>
```

---

Done! You now have a minimal, single-node SLURM setup on Ubuntu 22.04.
