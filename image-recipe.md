# Nvidia Jetson Nano image recipe
Guide to creating an image with various preinstalled packages for use in AAU.

## Steps

1. Get [the newest Jetson Nano Image](https://developer.nvidia.com/embedded/downloads).

2. Use BalenaEtcher to flash the image, see e.g. these instructions [Getting Started with Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#write). Flash and verify card.

3. Add cables including Ethernet cable for installing. Insert into NJN and start-up. 

    After some time, you should see a USB device popping up. If not, remove and re-insert USB cable.

4. Setup the Nano in headless mode. Connect via TTY

    1. `screen /dev/TTYACM0 15200`

    2. Follow the setup. If the TTY does not connect, reboot by removing and reinserting power, and try to connect to the TTY as soon as the L4T-README USB device shows up.
    
        Enter the following information in the setup:

        ```text
        User: AAU Nano
        username: aaunano
        password: aaunano
        hostname: aaunano
        locale: Denmark, base on English (UK)
        network: Choose Ethernet or wifi depending on your setup. Not important for later.
        ```
    
5. Update. There is a default Ubuntu process that locks the package manager and does an update and upgrade, but after a bit of time it's possible to continue

    ```text
    sudo apt update
    sudo apt upgrade
    ```

7. Install nomachine

    Note: Replace version in commands with the most recent version.

    - Download the ARMv8 (arm64) version [Nomachine ARMv8 Download](https://www.nomachine.com/download/download&id=112&s=ARM)

    - (On local machine:) scp file to device: `scp nomachine_7.26_arm64.deb aaunano@192.168.55.1:.`

    - (On Jetson Nano:) Install: `sudo dpkg -i nomachine_7.26_arm64.deb`

    - Remove: `rm -f nomachine_7.26_arm64.deb`

8. Connect with nomachine

    - Install client on local computer [Nomachine Client Download](https://www.nomachine.com/download/linux&id=1)

    - Connect to the nano on default IP `192.168.55.1`

9. Add Danish keyboard and adjust time zone

    - Press EN in top right corner -> Text Entry Setting -> + -> Danish -> Close

    - Press clock in top right corner -> Time & Date Settings -> Click Copenhagen

10. Install jetson-inference

    - Follow the steps on [Hello AI World: jetson-inference](https://github.com/dusty-nv/jetson-inference/blob/master/docs/building-repo-2.md). Use default values but add pytorch for python3.


11. Prevent keyring daemon from starting:

    `sudo chmod 644 /usr/bin/gnome-keyring-daemon`

    The reason is that we should not store passwords on a system with known password.

12. Change USB Information.

    - Follow these steps.

    ```text
    cd /opt/nvidia/l4t-usb-device-mode/
    sudo ./nv-l4t-usb-device-mode-runtime-stop.sh
    sudo mkdir /media/aaunano/img
    sudo mount -o filesystem.img /media/aaunano/img
    ```

    Open "Disks", click on the cog, "Edit Filesystem", enter new Volume name "AAU-README".
    
    Edit the files
    ```
    sudo ./nv-l4t-usb-device-mode-start.sh
    ```
    Modify shortcut on desktop.

    Right-click on L4T-README, click "Properties"

    Change "command" to `nautilus /media/aaunano/AAU-README`

13. Add scripts for Nomachine configuration

    `vim headless_mode.sh`

    ```
    #!/bin/bash
    exec sed -i '/CreateDisplay/s/ .*/ 1/' /usr/NX/etc/server.cfg
    ```

    `vim screenshare_mode.sh`

    ```
    #!/bin/bash
    exec sed -i '/CreateDisplay/s/ .*/ 0/' /usr/NX/etc/server.cfg
    chmod +x headless_mode.sh
    chmod +x screenshare_mode.sh
    ```

14. Increase swap space.

    ```bash
    # Disable ZRAM:
    sudo systemctl disable nvzramconfig
    # Create 8GB swap file
    sudo fallocate -l 8G /var/tmp/swap.img
    sudo chmod 600 /var/tmp/swap.img
    sudo mkswap /var/tmp/swap.img
    
    sudo sh -c "echo /var/tmp/swap.img none swap defaults 0 0 >>/etc/fstab"
    ```

15. Install libraries and programs.

    (is it necessary to compile opencv?)

    1. Pytorch, Tensorflow from Nvidia

    ```bash
    wget https://nvidia.box.com/shared/static/h1z9sw4bb1ybi0rm3tu8qdj8hs05ljbm.whl -O torch-1.9.0-cp36-cp36m-linux_aarch64.whl
    sudo apt-get install python3-pip libopenblas-base libopenmpi-dev 
    pip3 install Cython
    pip3 install numpy torch-1.8.0-cp36-cp36m-linux_aarch64.whl
    ```
    
    2. Torchvision

    ```bash
    sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev libavcodec-dev libavformat-dev libswscale-dev
    git clone --branch release/0.10 https://github.com/pytorch/vision torchvision   # see below for version of torchvision to download
    cd torchvision
    export BUILD_VERSION=0.10.0
    python3 setup.py install --user
    cd ..  # attempting to load torchvision from build dir will result in import error
    ```

    3. Torchaudio:

    ```bash
    git clone --branch release/0.9 https://github.com/pytorch/audio torchaudio
    cd torchaudio
    pip3 install pkgtools
    sudo apt install ninja-build
    git submodule update --init --recursive
    export CUDACXX=/usr/local/cuda-10.2/bin/nvcc
    BUILD_SOX=1 python3 setup.py install --user
    ```
    
    4. Tensorflow

    ```bash
    sudo apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev liblapack-dev libblas-dev gfortran
    pip3 install -U pip testresources setuptools
    pip3 install wheel
    
    pip3 install six
    pip3 install protobuf
    pip3 install uff
    
    pip3 install future mock h5py==2.10.0 keras_preprocessing keras_applications gast futures # Tested with newest versions 2021-08-13, only h5py needs old version.
    pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v46 tensorflow
    ```

    5. Numba

        First install llvm, then llvmlite. Also compile and install the updated version of tbb. Then numba can be installed.

    ```bash
    wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.1/clang+llvm-10.0.1-aarch64-linux-gnu.tar.xz
    tar -xvf clang+llvm-10.0.1-aarch64-linux-gnu.tar.xz
    cd clang+llvm-10.0.1-aarch64-linux-gnu
    sudo cp -R * /usr/local/
    export LLVM_CONFIG=/usr/local/bin/llvm-config
    pip3 install llvmlite==0.36.0
    
    pip3 install --upgrade colorama

    git clone https://github.com/syoyo/tbb-aarch64
    cd tbb-aarch64
    ./scripts/bootstrap-aarch64-linux.sh
    cd build-aarch64/
    make && make install
    cd ..
    cd dist
    sudo cp -rv * /usr
    
    pip3 install numba
    pip3 install --upgrade scipy
    pip3 install --upgrade pandas
    ```
    
    6. Easy to install packages:

    ```bash
    pip3 install sklearn
    pip3 install librosa
    pip3 install onnx
    pip3 install cupy #(works but takes a long time)
    ```
    
    7. Install Jupyter Lab

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
    sudo apt install -y nodejs
    pip3 install jupyter jupyterlab
    ```

12. Shutdown: `sudo shutdown`

13. Remove power and insert SD card in desktop/laptop.

14. Copy SD card

    1. Open gnome-disk or Disks from search menu in Ubuntu
    2. Click menu in top right of window -> Create Disk Image

    or alternatively something like

    `sudo dd if=/dev/sda of=AAUNANO-B.img`
  

