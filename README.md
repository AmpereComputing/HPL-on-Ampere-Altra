# HPL
HPL is a software package that solves a (random) dense linear system in double precision (64 bits) arithmetic on distributed-memory computers. It can thus be regarded as a portable as well as freely available implementation of the High Performance Computing Linpack Benchmark.

The algorithm used by HPL can be summarized by the following keywords: Two-dimensional block-cyclic data distribution – Right-looking variant of the LU factorization with row partial pivoting featuring multiple look-ahead depths – Recursive panel factorization with pivot search and column broadcast combined – Various virtual panel broadcast topologies – bandwidth reducing swap-broadcast algorithm – backward substitution with look-ahead of depth 1.

Official website for HPL : https://www.netlib.org/benchmark/hpl/

# Building HPL using Ampere Oracle BLIS Libraries.

Building HPL with Ampere-Oracle BLIS libraries is very easy and should not take a lot of time. It’s a 2-step process:
Step 1: Where we build the Math libraries found on the Ampere branch of Oracle BLIS libraries and Step 2 where we build the HPL binaries.

A detailed guide is below: 

## Step 1: Prerequisites: 

System Config :

OS : Ubuntu 20.04

GCC : 12.2.0

Kernel : 5.4.0-148-generic

Also, it's a good idea if running Ubuntu to do a  ```sudo apt install build-essential```  and take care of some basic dependencies.

To ensure a seamless build process, both, the math libraries and the benchmark are built inside the /opt directory.


a.	Downloading and installing Ampere Oracle BLIS Libraries:

```
pushd /opt
git clone https://github.com/flame/blis.git MyBlisDir
pushd MyBlisDir
#Switch to the new ampere branch 
git checkout ampere
./QuickStart.sh altramax
```

* Ensure that the test bench contains Ampere Oracle BLIS exported to PATH and LD_LIBRARY_PATH appropriately.

```
source ./blis_build_altramax.sh
source blis_setenv.sh
export LD_LIBRARY_PATH=/usr/local/lib:/opt/MyBlisDir/lib/altramax:$LD_LIBRARY_PATH
popd
popd
```

b.	OpenMPI: Along with Ampere Oracle BLIS, we will also need openmpi. We have used openmpi 4.1.4. Install it in the default path:

```
popd /opt
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.4.tar.gz
tar -xvf openmpi-4.1.4.tar.gz
cd openmpi-4.1.4
sudo apt install -y gfortran
./configure
make all install
export PATH=/usr/local/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
``` 

If OpenMPI is installed in a non-default location. Add the <bin> directory location to PATH and the <lib> directory location to LD_LIBRARY_PATH using the following commands

```
export PATH=<PATH_TO_OPENMPI_BIN_DIR>:$PATH
export LD_LIBRARY_PATH=< PATH_TO_OPENMPI_LIB_DIR>:$LD_LIBRARY_PATH
```

* Ensure successful installation of openmpi by executing the following commands.
	
```
mpirun --version  #(That should bring up the openmpi version. 4.1.4 in this case)
mpicc --version #(That should bring up the installed gcc version)
mpic++ --version #(That should bring up the installed g++ version)
mpifort --version #(That should bring up the installed gfortran version)
```
	
* If any of the above 3 commands do not return the version for gcc/g++/gfortran, install the missing gcc/g++/gfortran for your distro using

```
“sudo apt/yum/dnf install <package_name>”
```

## Step 2: Building HPL Benchmark

a.	Downloading and Installing HPL 2.3

```
pushd /opt
wget https://netlib.org/benchmark/hpl/hpl-2.3.tar.gz
tar -xzf hpl-2.3.tar.gz
popd
```
	
* Copy the Makefile from this repository into the `/opt/hpl-2.3` folder.

```
wget -P /opt/hpl-2.3 https://raw.githubusercontent.com/AmpereComputing/HPL-on-Ampere-Altra/main/Make.Altramax_oracleblis
```

* Compile the HPL binary

```
make arch=Altramax_oracleblis -j
```

* Upon success, a bin folder will be created. This folder should contain 2 files: xhpl (which is the HPL #binary) and HPL.dat (which is the standard input file).

```
pushd /opt/hpl-2.3/bin/Altramax_oracleblis 
```


## Step 3: Creating the HPL input file

Copy the HPL.dat file from this repository into `/opt/hpl-2.3/bin/Altramax_oracleblis`:

```
rm -f /opt/hpl-2.3/bin/Altramax_oracleblis/HPL.dat
wget -P /opt/hpl-2.3/bin/Altramax_oracleblis https://raw.githubusercontent.com/AmpereComputing/HPL-on-Ampere-Altra/main/HPL.dat
```

Please note that this HPL.dat file is designed to run on 96 cores at 64 GB RAM. If you have access to more RAM please refer to Step 5 on how to maximise the values for Ns

## Step 4: Run the benchmark

```
mpirun -np 96 --bind-to core --map-by core ./xhpl &> out.log
```

## Step 5: Performance Expectations

If your system differs from our testbench, the HPL.dat file will need to be modified (line #6) to match your respective Altra Max config.


### Table 1: Showcasing the line which needs to be modified in HPL.dat 	

|Line Number|Value|Description|
|---|---|---|
|6|150000|Ns|
	
The value of N when changed to 150K should take approximately 180 gigs of memory and would run on a machine having 256 GB memory. Table2 with differing values of Ns is shown below with our reference numbers.


### Table 2: Varying sizes on Ns w.r.t available system memory.

Our observed Results for AltraMax 96 cores @ 2.8GHz:
|Input Param|Input Param|
|---|---|	
|NB=256|P=8 Q=12|
	
|Problem Size (Ns)|Memory Used|Results (Gflops)|
|---|---|---|
|70K|46 gig|1151|
|100K|83 gig|1250|
|105K|91 gig|1253|
	
Our observed Results for AltraMax 128 cores @ 3.0GHz:
|Input Param|Input Param|
|---|---|	
|NB=256|P=8 Q=16|
	
|Problem Size (Ns)|Memory Used|Results (Gflops)|
|---|---|---|
|150K|177 gig|1528|
|200k|312 gig|1552|
|250k|480 gig|1597|

### You can find our list of additional HPC benchmarks here : https://github.com/AmpereComputing/HPC
