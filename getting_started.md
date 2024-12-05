# 1. Prerequisite

In openEuelr environment, the oneAPI framework integration is based on openEuler 24.03 LTS SP1 and subsequent versions.

# 2. Install oneAPI low level dependencies

Intel Arch SIG has integrated oneAPI required low level packages into openEuler 24.03 LTS SP1, we only to install them directly.

```
$ sudo dnf -y install intel-gmmlib intel-gsc intel-igc-cm intel-igc-core \
        intel-igc-opencl intel-level-zero-gpu intel-ocloc intel-opencl \
        level-zero libmetee
```

After the installation of oenAPI low level packages, the oneAPI runtime environment will be
supported on openEuler for running oneAPI applications for both Intel CPUs, GPUs, etc.

# 3. Install oneAPI toolkits

For openEuler oneAPI toolkit installation, the oneAPI repo can be configurated to use Intel official released rpm packages.

```
$ tee > /tmp/oneAPI.repo << EOF
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF
```

```
$ sudo mv /tmp/oneAPI.repo /etc/yum.repos.d
$ sudo dnf update
```

Install below required oneAPI Base Toolkit for building oneAPI applications:
* Intel® oneAPI DPC++ Compiler (Placeholder DPCPPROOT as its installation path, e.g. /opt/intel/oneapi/compiler/latest)
* Intel® oneAPI Math Kernel Library (oneMKL) (Placeholder MKLROOT as its installation path, e.g. /opt/intel/oneapi/mkl/latest)
* Intel® oneAPI Collective Communications Library (oneCCL) (Placeholder CCLROOT as its installation path, e.g. /opt/intel/oneapi/ccl/latest)
* Intel® MPI Library (Placeholder MPIROOT as its installation path, e.g. /opt/intel/oneapi/mpi/latest)

```
$ sudo dnf install -y intel-oneapi-dpcpp-cpp-2024.2-2024.2.1-1079 intel-oneapi-mkl-devel-2024.2.1-103 intel-oneapi-ccl-devel-2021.13.1-31
```

Check the installation result and hardware information.
```
$ source /opt/intel/oneapi/setvars.sh
$ sycl-ls
[opencl:cpu][opencl:0] Intel(R) OpenCL, 11th Gen Intel(R) Core(TM) i9-11900K @ 3.50GHz OpenCL 3.0 (Build 0) [2024.18.7.0.11_160000]
[opencl:gpu][opencl:1] Intel(R) OpenCL Graphics, Intel(R) Arc(TM) A770 Graphics OpenCL 3.0 NEO  [23.30.26918.50]
[opencl:gpu][opencl:2] Intel(R) OpenCL Graphics, Intel(R) UHD Graphics 750 OpenCL 3.0 NEO  [23.30.26918.50]
[level_zero:gpu][level_zero:0] Intel(R) Level-Zero, Intel(R) Arc(TM) A770 Graphics 1.3 [1.3.26918]
[level_zero:gpu][level_zero:1] Intel(R) Level-Zero, Intel(R) UHD Graphics 750 1.3 [1.3.26918]

```

# 4. Build and Run oneAPI samples

Link the required toolchain and add below environment paths for building and running oneAPI applications

```
$ sudo ln -s /usr/lib/gcc/x86_64-openEuler-linux/12/* /usr/lib64/
$ export CPATH=$CPATH:/usr/include/c++/12/:/usr/include/c++/12/x86_64-openEuler-linux/
$ export LD_LIBRARY_PATH=/opt/intel/oneapi/2024.0/lib:$LD_LIBRARY_PATH
```

Build and run the oneAPI samples

```
$ git clone https://github.com/oneapi-src/oneAPI-samples.git
$ cd oneAPI-samples/DirectProgramming/C++SYCL/DenseLinearAlgebra/vector-add/
$ mkdir build && cd build && cmake ..
$ make
$ ./vector-add-buffers
Running on device: Intel(R) Arc(TM) A770 Graphics
Vector size: 10000
[0]: 0 + 0 = 0
[1]: 1 + 1 = 2
[2]: 2 + 2 = 4
...
[9999]: 9999 + 9999 = 19998
Vector add successfully completed on device.
```

# 5. Install Intel Extension for Pytorch

This step is for installing Intel Extension for Pytorch. The installation of TorchVision and TorchAudio is option, prebuilt wheel files are available for Python 3.8, 3.9, 3.10, 3.11.

```
$ python3 -m pip install torch==2.3.1+cxx11.abi torchvision==0.18.1+cxx11.abi torchaudio==2.3.1+cxx11.abi \
                         intel-extension-for-pytorch==2.3.110+xpu oneccl_bind_pt==2.3.100+xpu \
						 --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/
$ python3 -m pip install transformers==4.33.0
```
# 6. Sanity Test
You can run a simple sanity test to double check if the correct version and requirement are intalled. If the software stack can get correct hardware information onboard your ystem, the information of PyTorch and Intel Extension for PyTorch version, GPU card information will be deteced. You can skip some warning message for using the environment without issue.
```
$ export OCL_ICD_VENDORS=/etc/OpenCL/vendors
$ python3 -c "import torch; import intel_extension_for_pytorch as ipex; print(torch.__version__); print(ipex.__version__); [print(f'[{i}]: {torch.xpu.get_device_properties(i)}') for i in range(torch.xpu.device_count())];"
```

# 7. Run LLM Demo (FastChat+Vicuna7B)
You can simply run the FastChat+Vicuna7B as a demo for utilizing Intel oneAPI with Intel GPUs as below.

```
$ python3 -m pip install "fschat[model_worker,webui]"
```

The command and flag to predict on gpu/cpu with: *--device xpu/cpu*. For example, running one GPU. If you are behind the proxy, try to download the vicuna-7b-v1.5 model offline.

```
$ python3 -m fastchat.serve.cli --model-path lmsys/vicuna-7b-v1.5 --device xpu
```

# 8. Known Issues
N/A
