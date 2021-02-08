# Description
This repository includes tests related to CoreNEURON and NEURON intergration.

More information about `NEURON` and `CoreNEURON` can be found here:
* [NEURON](https://github.com/neuronsimulator/nrn)
* [CoreNEURON](https://github.com/BlueBrain/CoreNeuron)

# How to install `NEURON` with `CoreNEURON`

More information and details how to do this you can find in the above links. A typical way to install them to run the tests is the following:

```
git clone https://github.com/neuronsimulator/nrn.git
cd nrn
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=./install -DNRN_ENABLE_CORENEURON=ON
make -j8
make install
```

For `GPU` execution set the following `CMake` variable:
```
-DCORENRN_ENABLE_GPU=ON
```
> Note: For `GPU` execution make sure that you use a compiler that supports `OpenACC` compilation and `CUDA` is also available in your system. See more details [here](https://neuronsimulator.github.io/nrn/coreneuron/how-to/coreneuron.html#installation). 

# How to run the tests

All the tests can be run with `NEURON` or `CoreNEURON` and with multiple configurations depending on the provided options to `special` or `special-core`.
The available options are described bellow:

* `coreneuron`: Enables the execution of the test with `CoreNEURON` using the on-line mode
* `gpu`: Enables the execution of the test with `CoreNEURON` on-line mode on `GPU`. 
> Note: for this option you need to make sure that `NEURON` is installed with the `CMake` option `CORENRN_ENABLE_GPU` enabled.
* `filemode`: Runs `CoreNEURON` from `NEURON` in off-line mode. `NEURON` automatically generates the `CoreNEURON` dataset and runs the simulation with `CoreNEURON`. To use this `coreneuron` option must be enabled.
* `dumpmodel`: Creates the `CoreNEURON` dataset. This enables `CoreNEURON` to run separately after the initial simulation.

## Example

### On-line mode
The following commands runs one of the included test with `CoreNEURON` and with the `GPU` backend enabled.

```
nrnivmodl -coreneuron mod
mpirun -n 2 ./x86_64/special -mpi -c sim_time=100 -c coreneuron=1 -c gpu=1 testkin.hoc
```
### Off-line mode
The following commands runs one of the included tests with `NEURON`, generates the `CoreNEURON` dataset, runs the `CoreNEURON` simulation and compares the spikes between the two.
```
nrnivmodl -coreneuron mod
mpirun -n 2 ./x86_64/special -mpi -c sim_time=100 -c dumpmodel=1 -c gpu=1 testkin.hoc
cat outkin.dat | sort -k 1n,1n -k 2n,2n > out_nrn_kin.spk
mpirun -n 2 ./x86_64/special-core --mpi --tstop 100 --datpath coredat
cat out.dat | sort -k 1n,1n -k 2n,2n > out_cn_kin.spk
diff -w -q out_nrn_kin.spk out_cn_kin.spk
```

To run all the tests you can use the `run.sh` file which runs all the tests included in this repository with `NEURON` using some default options and then compares the generated spikes with the `CoreNEURON` simulation.
You can select which tests to run by setting the `spike_comparison_tests`, `direct_tests` and `gpu_tests`.
> Note: Make sure that you compiled the files before running the script.

There are 3 kinds of tests:
* `spike_comparison_tests`: Those tests are used only to compare the generated spikes between `NEURON` and `CoreNEURON`
  * bbcore
  * conc
  * deriv
  * gf
  * kin
  * patstim
  * vecplay
  * watch
  * vecevent
* `direct_tests`: Those test compare internally the voltage and currents between `NEURON` and `CoreNEURON`
  * netstimdirect
* `gpu_tests`: Same as `spike_comparison_tests` but if set they compare the spikes between `NEURON` and `CoreNEURON` using the `GPU` backend.