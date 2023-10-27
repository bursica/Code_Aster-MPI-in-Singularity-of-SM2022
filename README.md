# Code_Aster-MPI-in-Singularity-of-SM2022
## Prerequisites
Assuming you have Singularity up and running, otherwise check
- https://github.com/sylabs/singularity/blob/main/INSTALL.md 
- https://docs.sylabs.io/guides/3.0/user-guide/quick_start.html

After sucessful Singularity installation:
```
mkdir $HOME/containers && cd $HOME/containers
```

Download and put the container image (.sif) in $HOME/containers as described in https://gitlab.com/codeaster-opensource-documentation/

### Example:
```
wget -c https://www.code-aster.org/FICHIERS/singularity/salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif

singularity run --app install salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif
```

## Preparing the access to the container

Follow https://gitlab.com/codeaster-opensource-documentation/opensource-installation-development/-/blob/main/devel/compile.md#preparing-the-access-to-the-container.

### Add an overlay to the Singularity container, so data can be written to it:
```
dd if=/dev/zero of=overlay.img bs=1M count=1500 && mkfs.ext3 overlay.img

singularity sif add --datatype 4 --partfs 2 --parttype 4 --partarch 2 --groupid 1 salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif overlay.img
```

The overlay.img is not needed anymore: 
```
rm overlay.img
```
Your container file salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif should now be +1.5G larger

## Working in Container 
Open the container in a shell and bind:
```
sudo singularity run --bind ${HOME}/containers:${HOME}/containers -w ${HOME}/containers/salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif shell
```
Move to:
```
cd ${HOME}/dev/codeaster/src/code_aster
```
and create:
```
nano pkginfo.py
```
with (change the version/date as needed):
```
pkginfo = [(16, 4, 0), 'n/a', 'n/a', '30/06/2023', 'n/a', 1, ['no source repository']]
```
Save and exit.

Now execute the following commands, one by one, to build and install the MPI-Version of Code-Aster (e.g.16.4.0) within the container (denoted by Singularity> prefix):
```
cd ${HOME}/dev/codeaster/src

git checkout tags/16.4.0

export TOOLS="/opt/salome_meca/V2022.1.0_scibian_univ/tools"

export ASTER_TESTING="${TOOLS}/Code_aster_testing-1640"


./waf_mpi configure --prefix=${ASTER_TESTING} --install-tests --jobs=8

./waf_mpi build --jobs=8

./waf_mpi install


echo "vers : testing_mpi:/opt/salome_meca/V2022.1.0_scibian_univ/tools/Code_aster_testing-1640/share/aster" >> ${TOOLS}/Code_aster_frontend-202200/etc/codeaster/aster
```
The last command makes the MPI-version accessible within Salome-Meca. 

Now you should be able to launch Salome-Meca within the container, first exit the container shell with:
```
exit
```
And run Salome-Meca with --soft flag if you don't have Nvidia GPU:
```
salome_meca-lgpl-2022.1.0-1-20221225-scibian-9 --soft
```
Happy FEM-ing.
