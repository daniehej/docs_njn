# Installing OpenFace

[OpenFace](https://github.com/TadasBaltrusaitis/OpenFace) is a toolkit for facial behavior analysis such as detecting head pose and eye gaze tracking.

OpenFace is not pre-compiled for ARM, so we have to compile it manually. This involves a couple of steps, which combine the [Unix installation Guide](https://github.com/TadasBaltrusaitis/OpenFace/wiki/Unix-Installation) and [This issue](https://github.com/TadasBaltrusaitis/OpenFace/issues/714).

1. First follow the steps in the [Unix installation Guide](https://github.com/TadasBaltrusaitis/OpenFace/wiki/Unix-Installation)

```
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install g++-8
sudo apt-get install cmake
sudo apt-get install libopenblas-dev
sudo apt-get install libboost-all-dev
sudo apt-get install liblapack-dev liblapacke-dev
```

Install dlib version 19.22 or above. See the available versions at [The dlib website](http://dlib.net/files)

```
wget http://dlib.net/files/dlib-19.24.tar.bz2;
tar xf dlib-19.24.tar.bz2;
cd dlib-19.24;
mkdir build;
cd build;
cmake ..;
cmake --build . --config Release;
sudo make install;
sudo ldconfig;
cd ../..;    
```

2. Get OpenFace

```
git clone https://github.com/TadasBaltrusaitis/OpenFace.git
cd OpenFace
```

3. Edit scripts for ARM

In cmake/modules/FindOpenBLAS.cmake add the two lines

```
/usr/include/aarch64-linux-gnu/
```
and
```
/usr/lib/aarch64-linux-gnu/
```
in the include and lib search paths respectively.

4. Then comment out the section with x86 SIMD options in CMakeLists.txt:

```
#if (${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
#    execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
#    if (GCC_VERSION VERSION_LESS 4.7)
#        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -msse -msse2 -msse3")
#    else ()
#        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -msse -msse2 -msse3")
#    endif ()
#else ()
#    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -msse -msse2 -msse3")
#endif ()
```

5. Compile the code 

```
mkdir build
cd build
cmake -D CMAKE_CXX_COMPILER=g++-8 -D CMAKE_C_COMPILER=gcc-8 -D CMAKE_BUILD_TYPE=RELEASE ..
make
```

In order to run the samples it is necessary to donwload the models for the CE-CLM algorithm as described in [The documentation](https://github.com/TadasBaltrusaitis/OpenFace/wiki/Model-download). These files should be placed in build/bin/lib/local/LandmarkDetector/model/patch_experts.
