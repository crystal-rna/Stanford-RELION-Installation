# RELION 5 Beta Installation on Stanford University Sherlock
Installation of RELION 5 beta on Stanford University Sherlock. This protocol is based off of Dong-Hua Chen's installation (Kornberg lab) here at Stanford, and Chris Haley's miniforge 3 installation process (chaley@stanford.edu).

## Sherlock login and access to directories

Setup sherlock account. Email srcc-support@stanford.edu stating you want access. Your PI will need to approve via email reply.

Login to sherlock: Open Terminal (Mac) or Windows Powershell (Windows) and input the following command:
```
ssh <sunetid>@login.sherlock.stanford.edu
```
RELION 5 beta requires Anaconda installation for its new machine learning algorithims. This will exceed 15GB, so it is recommended you install these packages on your GROUP_HOME.


Access your lab group's ($GROUP_HOME) directory:
```
cd /home/groups/<PI_group_name>
```
or simply execute:
```
cd $GROUP_HOME
```
Access your home directory:
```
cd /home/users/<sunetid>
```
or simply execute:
```
cd
```

## Edit your .bashrc file
This is a file that runs any commands you input everytime you login to Sherlock. For RELION, we will want to have several modules loaded before running. Having these commands in our .bashrc will eliminate the need for you to have run each command yourself in the ocmmand line manually everytime you restart Sherlock.

To edit:
vim .bashrc

This will place you in the text editing interface. Press "i" to insert text. Use arrow keys to navigate to bottom of this file. CMD+C (Mac) or CTRL+C (Windows) to copy, then CMD+V (Mac) or right click (Windows) to paste.

```
ml devel openmpi/4.1.2
ml devel cmake/3.8.1
ml devel cuda/11.5.0
ml devel math fftw/3.3.10
ml devel system fltk/1.3.4
ml devel devel x11/7.7
ml devel libtiff/4.5.0
ml system gtk+/2.24.30

export PATH=/home/users/cstack/relion/bin:${PATH}
export LD_LIBRARY_PATH=/home/users/cstack/relion/lib:$LD_LIBRARY_PATH

```
To refresh your .bashrc after you have made the respective changes:
```
source .bashrc
```



## Download and Install Relion
Installation based off of: https://relion.readthedocs.io/en/latest/Installation.html

Go to your user folder and clone the RELION repository:
```
cd
git clone https://github.com/3dem/relion.git
```
This will create a local Git repository. All subsequent git-commands should be run inside this directory.

Then run:
```
git checkout ver5.0
```
to access latest updates for RELION 5.0X.
To incoporate these changes run:
```
git pull
```
You will run the previous two commands if you wish to update RELION.


### Miniforge Installation
Install this in $GROUP_HOME under a directory name of your choice. For example:
```
cd $GROUP_HOME
mkdir <your_name>
cd <your_name>
```
In this directory, create the miniforge 3 installation script by running the code below:
```
cat <<EOF >  install.miniforge3.sh
#!/bin/bash

# Script for installing miniforge3.
DATE=$(date +"%F-%Hh%Mm%Ss")
LOGFILE=miniforge3-install-$DATE.log
exec &> >(tee $LOGFILE)

echo "Date of install: $DATE"

# Purge modules and load python.
module purge
module load python/3.9.0

# Set directory variables.
DIR_PREFIX=$(pwd)
SRC_MINIFORGE=$DIR_PREFIX/programs/src/miniforge3
echo $SRC_MINIFORGE
mkdir -p $SRC_MINIFORGE
RUN_MINIFORGE=$DIR_PREFIX/programs/run/miniforge3
echo $RUN_MINIFORGE

# Download install script to src
cd $SRC_MINIFORGE
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"

# Run install script. (-b sets batch automation, -p sets install location, -t runs test scripts after install)
bash $SRC_MINIFORGE/Miniforge3-$(uname)-$(uname -m).sh -p $RUN_MINIFORGE

# After install, refresh the terminal for installation to take effect
source ~/.bashrc

# Keep conda base environment deactivated by default
$RUN_MINIFORGE/condabin/conda config --set auto_activate_base False


EOF
```
Make this script executable by running the following:
```
chmod u+x install_miniforge3.sh
```
Install miniforge by executing the script you created:
```
./install_miniforge3.sh 
```
After installation, run:
```
which conda
```
Display of miniforge3 will indicate successful installation.

 
Conda environment code can be updated by running the following (SEE WARNING BELOW):
```
conda env update -f environment.yml
```
As per Relion read the docs:
You should NOT activate this relion-5.0 conda environment when compiling and using RELION. In other words, if you want to update conda, do it when you first start up Sherlock and not when you have had RELION running. Otherwise, you will get run time errors and may need to reinstall everything.

## Relion Installation
RELION 5 beta requires Anaconda to run its new machine learning algorithims. The training dataset is large (>15GB), so this should all be installed on your $GROUP_HOME. The RELION files will be installed in your user home.

First, go to your $GROUP_HOME and create a directory for PyTorch Machine Learning files:
```
cd $GROUP_HOME
mkdir torch
```
Next, make a directory called RELION in your user folder where the RELION build will be installed:
```
cd
mkdir relion
cd relion
mkdir build
cd build
```
Then run the following:
```
cmake -DCMAKE_INSTALL_PREFIX=/home/users/<sunetid>/relion/ -DFORCE_OWN_FTLK=ON -DTORCH_HOME_PATH=/home/groups/<group_name>/<your_name>/torch ..
```
Importantly, this specifies that PyTorch associated downloads will be in your $GROUP_HOME! They are too large to be installed on your user space.

Once this is complete, run the make command to install the program:
```
make -j 8
```
Then run:
```
make install
```
RELION is now installed! The last steps are to install two external programs.

### Installing CTFFIND and MotionCor2
CTF estimation is not a part of RELION. Install it separately here: https://grigoriefflab.umassmed.edu/ctf_estimation_ctffind_ctftilt

For MotionCor2, install it here:

Now, both files will be where your downloads, i.e: your downloads folder. We will need to transfer these to your Sherlock user directory so we can link them to RELION.

To do this, open an additional instance of Terminal/Powershell, while keeping your terminal logged into Sherlock open. In this new terminal, locate the downloads folder:
```
cd Downloads
cd MotionCor2_1
```

Execute the transfer with the command below. If necessary, change the MotionCor2 filename to what you installed.
```
rsync -av MotionCor2_1.6.4_Cuda115_Mar312023 <sunetid>@dtn.sherlock.stanford.edu:/home/users/<sunetid>
```
This will prompt you to login again with your sunet password.

Go back to downloads directory (previous directory):
```
cd ..
```
And transfer CTFFIND with the command below. Again, If necessary, change the MotionCor2 filename to what you installed.
```
rsync -av ctffind-5.0.2.tar.gz cstack@dtn.sherlock.stanford.edu:/home/users/cstack
```
With these transferred to your user Sherlock home, you may now close out of the second terminal we had opened. Go back to the terminal that is logged into Sherlock.
Now, we will create pointers so that we can tell RELION where to look (without the name being super long!). These pointers are like "shorcuts" on your PC. 

Navigate to your home directory. MotionCor2 is ready to go. Make a "MotionCor2" pointer by executing the following:
```
cd
ln -s MotionCor2_1.6.4_Cuda115_Mar312023 MotionCor2 
```

For CTFFIND, change the .tar file name in the first line to match the name that you installed.
```
gunzip ctffind-5.0.2.tar.gz 
tar -xf ctffind-5.0.2.tar 
cd cisTEM
cd bin
ln ctffind /home/users/<sunetid>/ctffind
```

With both packages installed on Sherlock with easier directory pointers it will be much easier to point to these when RELION is open.

## Opening and running RELION

### Troubleshooting
If Torch stuff installs into your own user folder, it will most likely end up in the hidden .cache directory. To delete this:
```
cd
rm -rf .cache
```



## Create slurm job submission file
This is the file that indicates how many resources you will request when submitting to Sherlock.
Edit the time to 48:00:00 if you plan to work on normal partition. Change the number of GPUs and memory according to your lab's resources. Run the script below to create this file.
```
cat <<EOF >  slurm_8gpu.bash
#!/bin/bash
# Number of nodes to request, 16 CPUS per node
#number of jobs, max of 500 total CPUS in use for one job (n*c<501)
#SBATCH -n XXXmpinodesXXX
#Number of threads per job
#SBATCH -c XXXthreadsXXX
#Time Limit, max of 48 hours
#SBATCH --time=168:00:00
#Define Que normal, dev, owners, gpu 
#SBATCH -p XXXqueueXXX
#SBATCH --gres=gpu:8
#error files
#SBATCH --mem=512000
#SBATCH -e XXXerrfileXXX
#SBATCH -o XXXoutfileXXX
##SBATCH --exclusive
##SBATCH --no-requeue
##SBATCH --exclude=gpu-27-21,gpu-27-35

#SBATCH --gpu_cmode=shared

module load devel cuda/11.5.0
mpirun -n XXXmpinodesXXX XXXcommandXXX
EOF
```


### Helpful Unix commands
Check history of commands you have run:

```
history |grep <command>
```


```
cat <<EOF >  slurm_8gpu.bash


EOF
```


# Extras 
Modify your .bashrc file to adjust your terminal environment by editing and sourcing .bashrc to make sure conda commands from Miniforge3 are recognized.

Go to your user directory to modify your .bashrc file:
```
cd
vi .bashrc
```
At end of .bashrc, paste the following. Ensure you change all text in '< >' to proper directory names:
```
# >>> miniforge initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__miniforge_setup="$('/home/groups/<group_name>/<your_name>/programs/run/miniforge3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__miniforge_setup"
else
    if [ -f "/home/groups/<group_name>/<your_name>/programs/run/miniforge3/etc/profile.d/conda.sh" ]; then
        . "/home/groups/<group_name>/<your_name>/programs/run/miniforge3/etc/profile.d/conda.sh"
    else
        export PATH="/home/groups/<group_name>/<your_name>/programs/run/miniforge3/bin:$PATH"
    fi
fi
unset __miniforge_setup
# <<< miniforge initialize <<<

```

