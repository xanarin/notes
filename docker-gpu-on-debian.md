# How to get Docker GPU working on Debian

As of 2025-09-14, you must use Debian 12 because NVIDIA doesn't have their proprietary drivers ready for Debian 13 yet. You can check to see what Debian releases are supported by NVIDIA here:
https://developer.download.nvidia.com/compute/cuda/repos/

I chose to use the repository-based installation method from NVIDIA as opposed to the `.run` file installers, so that updates are easier to get in the future.

Now, how to get everything working? Here are the steps (at a high level). These were done on GAMING-PC to grant access to the NVIDIA RTX 3080 to Docker containers:

1. Install Debian dependencies. These appear to be:
   a. build-essential
   b. linux-headers-amd64
1. Follow the [CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#debian). At the current time, this includes:
   a. Enabling the `contrib` section for the Debian repos
   b. Downloading the CUDA keyring package (this adds the apt sources.list for the cuda repos, along with the repo GPG key): `wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb && sudo apt install ./cuda-keyring_1.1-1_all.deb`
   c. `sudo apt update`
   d. Install the CUDA toolkit: `sudo apt install cuda-toolkit`
   e. Add the CUDA toolkit utils to your PATH: `export PATH=${PATH}:/usr/local/cuda-13.0/bin`
      a. Don't be clever and symlink these utilities into `/usr/local/bin`; they rely on their location to find other files
1. Install the NVIDIA proprietary drivers
   a. `sudo apt install nvidia-driver-cuda`
   b. Run `sudo dkms status` and make sure the kernel modules were installed properly. If the output just says "ready", then DKMS has not built the kernel modules yet. You can run `sudo dkms autoinstall` to force DKMS to rebuild the driver
1. Follow the [installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) for the NVIDIA container toolkit
   a. Add the repository: 
   ```sh
   $ curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```
   b. `sudo apt update`
   c. Install NVIDIA Container Toolkit
   ```sh
   $ export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
     sudo apt-get install -y \
        nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
        nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
        libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
        libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
   ```
1. Reboot the system and ensure that the nvidia driver is in use (not nouveau): `lspci -v -s "$(lspci | grep -m1 -i nvidia | cut -d' ' -f1)" | grep -i 'driver in use'`
1. Run `nvidia-smi` to ensure that the GPU is detected
1. Run a Docker container to ensure that the GPU is passed through: `docker run --rm --gpus all nvidia/cuda:12.6.2-devel-ubuntu22.04 nvidia-smi`
