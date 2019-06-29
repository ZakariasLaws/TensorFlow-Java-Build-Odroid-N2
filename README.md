# Build TensorFlow Java API for Odroid-N2

This guide will help you build the [TensorFlow Java API](https://www.tensorflow.org/install/lang_java) from source on Odroid-N2, using the latest stable r1.14 TensorFlow branch.

In order to use the TensorFlow Java API, the TensorFlow JAR and JNI files are required, at the time of writing, the official [TensorFlow website](https://www.tensorflow.org/install/lang_java) does not support aarch64 devices, hence the required files need to be built from source.

* See `/build/` for the already built files, if you are lucky, these files might work out of the box (as long as you have installed the dependencies). To use, simply copy the directory `bazel-bin` to your project, after which you can compile and run your code according to section [Test Installation](#test).

## Dependencies

OpenJDK 8 is necessary to download and install Bazel version 0.25.2, later when installing TensorFlow, a later Java version can be used.

### OpenJDK 8
OpenJDK version 8 is available from the PPA repository [OpenJDK builds](https://launchpad.net/~openjdk-r/+archive/ubuntu/ppa).

```bash
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
```

Install OpenJDK 8:
```bash
sudo apt-get install openjdk-8-jdk
```

Check that your Odroid is using Java1.8 by updating the alternatives, make sure to select openjdk-8. The output should look something like this:

```bash
odroid@odroid:~$ sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-arm64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-arm64/bin/java      1111      manual mode
* 2            /usr/lib/jvm/java-8-openjdk-arm64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
```

Repeat the previous operation for javac and verify them both.
```bash
odroid@odroid:~$ java -version
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-8u212-b03-0ubuntu1.18.04.1-b03)
OpenJDK 64-Bit Server VM (build 25.212-b03, mixed mode)
```

```bash
javac -version
javac 1.8.0_212
```

### Other Dependencies
Run the following command in your terminal:
```bash
sudo apt install pkg-config zip g++ zlib1g-dev unzip autoconf automake libtool
sudo apt update
sudo apt upgrade
```
### Pyhton
Python 2.7.15+
```bash
# For Python 2.7
sudo apt-get install python-pip python-numpy swig python-dev
sudo pip install future
sudo pip install wheel
```

Python 3.6+
```bash
sudo apt-get install python3-pip python3-numpy swig python3-dev
sudo pip3 install wheel
```

##### GCC and G++

GCC and G++ version 7.4.0 was used when writing this guide, you can check your versions by typing `gcc -v` and `g++ -v`, the result should look something like the following:
```bash
odroid@odroid:~$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/aarch64-linux-gnu/7/lto-wrapper
Target: aarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu/Linaro 7.4.0-1ubuntu1~18.04.1' --with-bugurl=file:///usr/share/doc/gcc-7/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++ --prefix=/usr --with-gcc-major-version-only --program-suffix=-7 --program-prefix=aarch64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --enable-multiarch --enable-fix-cortex-a53-843419 --disable-werror --enable-checking=release --build=aarch64-linux-gnu --host=aarch64-linux-gnu --target=aarch64-linux-gnu
Thread model: posix
gcc version 7.4.0 (Ubuntu/Linaro 7.4.0-1ubuntu1~18.04.1) 
```

```bash
odroid@odroid:~$ g++ -v
Using built-in specs.
COLLECT_GCC=g++
COLLECT_LTO_WRAPPER=/usr/lib/gcc/aarch64-linux-gnu/7/lto-wrapper
Target: aarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu/Linaro 7.4.0-1ubuntu1~18.04.1' --with-bugurl=file:///usr/share/doc/gcc-7/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++ --prefix=/usr --with-gcc-major-version-only --program-suffix=-7 --program-prefix=aarch64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --enable-multiarch --enable-fix-cortex-a53-843419 --disable-werror --enable-checking=release --build=aarch64-linux-gnu --host=aarch64-linux-gnu --target=aarch64-linux-gnu
Thread model: posix
gcc version 7.4.0 (Ubuntu/Linaro 7.4.0-1ubuntu1~18.04.1) 
```

## Swap Memory
Most likely you will require some extra swap memory in order to Compile TensorFlow, this can be added temporarily by following the next few steps.

1. Check if there already is swap installed, you should preferably only have one swap partion at the time. If the output is empty, you don't currently have any swap installed
   ```bash
   sudo swapon --show
   ```
2. Create a swap partions:
   ```bash
   sudo fallocate -l 10G /swapfile
   ```
3. Set root permission:
   ```bash
   sudo chmod 600 /swapfile
   ```
4. Turn on swap:
   ```bash
   sudo mkswap /swapfile
   ```
5. Activate swap:
   ```bash
   sudo swapon /swapfile
   ```
6. Check that the swap file was succesfully setup with `free -h`:
   ```bash
   odroid@odroid:~$ free -h
   total        used        free      shared  buff/cache   available
   Mem:           3.6G        162M        2.7G        120K        791M        3.4G
   Swap:           10G        125M         10G
   ```
If you wish to make the swap partition persist when rebooting the Odroid, add the following line `/swapfile swap swap defaults 0 0` to the `/etc/fstab` file.

## Bazel
In order to build TensorFlow we need to use [Bazel](https://bazel.build/), the official build tool for TensorFlow, developed and maintained by Google. It works to build [r1.14](https://www.tensorflow.org/api_docs/python/tf) TensorFlow with version 0.25.2, however, it might also work with later/earlier versions.

Download the bazel archive `bazel-0.25.2-dist.zip` from the [bazel releases github](https://github.com/bazelbuild/bazel/releases), unzip it to a location of choice. or download with wget:
Alternatively, use `wget` and `unzip`.
```bash
wget https://github.com/bazelbuild/bazel/releases/download/0.25.2/bazel-0.25.2-dist.zip
unzip -d bazel bazel-0.25.2-dist.zip
cd bazel
```

Compile bazel supplying the JDK location as a command line argument (make sure to be in the root of the bazel directory):
```bash
env EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh
```

This will take approximately 30 min and should be about `[... / 1,791] Actions`. When finished, the bazel binary will be placed in `output/bazel`, copy or move this binary to a location discoverable by PATH, so you can execute it by only typing `bazel` in the terminal (alternatively add this location to PATH).
```bash
sudo cp output/bazel /usr/local/bin/
```

To verify that Bazel is working correctly type `bazel` in your command line, it might take a few seconds to run and should output the following:

```bash
odroid@odroid:~$ bazel
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
                                              [bazel release 0.25.2- (@non-git)]
Usage: bazel <command> <options> ...

Available commands:
  analyze-profile     Analyzes build profile data.
  aquery              Analyzes the given targets and queries the action graph.
  build               Builds the specified targets.
  canonicalize-flags  Canonicalizes a list of bazel options.
  clean               Removes output files and optionally stops the server.
  coverage            Generates code coverage report for specified test targets.
  cquery              Loads, analyzes, and queries the specified targets w/ configurations.
  dump                Dumps the internal state of the bazel server process.
  fetch               Fetches external repositories that are prerequisites to the targets.
  help                Prints help for commands, or the index.
  info                Displays runtime info about the bazel server.
  license             Prints the license of this software.
  mobile-install      Installs targets to mobile devices.
  print_action        Prints the command line args for compiling a file.
  query               Executes a dependency graph query.
  run                 Runs the specified target.
  shutdown            Stops the bazel server.
  sync                Syncs all repositories specified in the workspace file
  test                Builds and runs the specified test targets.
  version             Prints version information for bazel.

Getting more help:
  bazel help <command>
                   Prints help and options for <command>.
  bazel help startup_options
                   Options for the JVM hosting bazel.
  bazel help target-syntax
                   Explains the syntax for specifying targets.
  bazel help info-keys
                   Displays a list of keys used by the info command.
```

## Install TensorFlow r1.14
Now we are ready to build TensorFlow from source, start with cloning the [Tensorflow GitHub page] (https://github.com/tensorflow/tensorflow), this might take a few minutes:

```bash
git clone https://github.com/tensorflow/tensorflow.git
```

When finished, enter the directory and checkout branch r1.14, which we want to install.
```bash
cd tensorflow
checkout r1.14
```

Before starting the build, configure the installation by using the default python path and selecting no (`n`) on everything:
```bash
odroid@odroid:~$ ./configure
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.google.protobuf.UnsafeUtil (file:/home/odroid/.cache/bazel/_bazel_odroid/install/0551bb8d4f083266f46f0e4b52f8846f/_embedded_binaries/A-server.jar) to field java.nio.Buffer.address
WARNING: Please consider reporting this to the maintainers of com.google.protobuf.UnsafeUtil
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
You have bazel 0.25.2- (@non-git) installed.
Please specify the location of python. [Default is /usr/bin/python]: 


Found possible Python library paths:
  /usr/local/lib/python2.7/dist-packages
  /usr/lib/python2.7/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python2.7/dist-packages]

Do you wish to build TensorFlow with XLA JIT support? [Y/n]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]: n
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: n
No CUDA support will be enabled for TensorFlow.

Do you wish to download a fresh release of clang? (Experimental) [y/N]: n
Clang will not be downloaded.

Do you wish to build TensorFlow with MPI support? [y/N]: n
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]:  


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
	--config=gdr         	# Build with GDR support.
	--config=verbs       	# Build with libverbs support.
	--config=ngraph      	# Build with Intel nGraph support.
	--config=numa        	# Build with NUMA support.
	--config=dynamic_kernels	# (Experimental) Build kernels into separate shared objects.
Preconfigured Bazel build configs to DISABLE default on features:
	--config=noaws       	# Disable AWS S3 filesystem support.
	--config=nogcp       	# Disable GCP support.
	--config=nohdfs      	# Disable HDFS support.
	--config=noignite    	# Disable Apache Ignite support.
	--config=nokafka     	# Disable Apache Kafka support.
	--config=nonccl      	# Disable NVIDIA NCCL support.
Configuration finished

```

Finally, build TensorFlow. Be patient, this will take up to 8 hours (depending on resource utilization... 

* In case you are connected to the Odroid with `ssh` and want to run the build in the background in orer to exit the session, use `nohup`, see the second command. 
* The flag `--local_resources 2048,2,1.0` tells Bazel we want to use 2048MB of memory and 2 cores and 1.0 in available I/O, not specifying this will result in too many threads spawning, creating an infinite compilation which will be killed by the operating system. This would look something like this:
   ```bash
   [2,359 / 2,470] Compiling tensorflow/core/kernels/matrix_square_root_op.cc; 754s local ... (6 actions, 2 running)
   [2,359 / 2,470] Compiling tensorflow/core/kernels/matrix_square_root_op.cc; 2364s local ... (6 actions, 2 running)
   ERROR: /home/odroid/Constellation/tensorflow/tensorflow/core/kernels/BUILD:3255:1: C++ compilation of rule '//tensorflow/core/kernels:matrix_square_root_op' failed (Exit 4)
   gcc: internal compiler error: Killed (program cc1plus)
   ```
* The flag `--host_javabase=@local_jdk//:jdk` tells bazel which compilation flag we wish to use, this is necessary in case bazel does not automatically pick up the location of the installed JDK.
* The `--config opt` specifies targets to compile, in our case this is the TensorFlow JAR archive and the native Java bindings for aarch64.

```bash
bazel build --host_javabase=@local_jdk//:jdk --local_resources 2048,2,1.0 --config opt //tensorflow/java:tensorflow //tensorflow/java:libtensorflow_jni
```
Using nohup
```bash
nohup bazel build --host_javabase=@local_jdk//:jdk --local_resources 2048,2,1.0 --config opt //tensorflow/java:tensorflow //tensorflow/java:libtensorflow_jni &
```

Watch the progress when running in the background, there should be about `[... / 5,292] actions` in total:
```bash
watch tail nohup.out
```

When it is done you will see similar output to this:
```bash
[5,292 / 5,292]
INFO: Elapsed time: 23339.351s, Critical Path: 911.43s
INFO: 1013 processes: 1011 local, 2 worker.
INFO: Build completed successfully, 1016 total actions
INFO: Build completed successfully, 1016 total actions
```

In order to use the Java bindings, compile your Java source file with`-cp bazel-bin/tensorflow/java/libtensorflow.jar` and run the generated class file with `-cp bazel-bin/tensorflow/java/libtensorflow.jar:. -Djava.library.path=bazel-bin/tensorflow/java/`.

## <a name="test"> Test Installation
You can test the installation by compiling the `TensorFlowExample.java` file provided in this repository. NOTE that `/path/to/` needs to be replaced with the path to the directory above your `bazel-bin` directory is. This will be your `tensorflow` root directory, unless you moved it.

```bash
odroid@odroid:~$ javac -cp /path/to/bazel-bin/tensorflow/java/libtensorflow.jar TensorfFlowExample.java
odroid@odroid:~$ java -cp /path/to/bazel-bin/tensorflow/java/libtensorflow.jar:. -Djava.library.path=/bazel-bin/tensorflow/java/ TensorFlowExample
TensorFlowExample using TensorFlow version: 1.14.0
```

You're done!
