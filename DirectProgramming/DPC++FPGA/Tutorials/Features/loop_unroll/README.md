# Unrolling Loops
This tutorial demonstrates a simple example of unrolling loops to improve throughput for a SYCL*-compliant FPGA program.

| Optimized for                     | Description
|:---                                 |:---
| OS                                | Linux* Ubuntu* 18.04/20.04 <br> RHEL*/CentOS* 8 <br> SUSE* 15 <br> Windows* 10
| Hardware                          | Intel&reg; Programmable Acceleration Card (PAC) with Intel Arria&reg; 10 GX FPGA <br> Intel&reg; FPGA Programmable Acceleration Card (PAC) D5005 (with Intel Stratix&reg; 10 SX) <br> Intel&reg; FPGA 3rd party / custom platforms with oneAPI support <br> **Note**: Intel&reg; FPGA PAC hardware is only compatible with Ubuntu 18.04*
| Software                          | Intel&reg; oneAPI DPC++/C++ Compiler <br> Intel&reg; FPGA Add-On for oneAPI Base Toolkit
| What you will learn               |  Basics of loop unrolling. <br> How to unroll loops in your program. <br> Determining the optimal unroll factor for your program.
| Time to complete                  | 15 minutes

## Purpose

The loop unrolling mechanism is used to increase program parallelism by duplicating the compute logic within a loop. The number of times the loop logic is duplicated is called the *unroll factor*. Depending on whether the *unroll factor* is equal to the number of loop iterations or not, loop unroll methods can be categorized as *full-loop unrolling* and *partial-loop unrolling*.

### Example: Full-Loop Unrolling
```c++
// Before unrolling loop
#pragma unroll
for(i = 0 ; i < 5; i++){
  a[i] += 1;
}

// Equivalent code after unrolling
// There is no longer any loop
a[0] += 1;
a[1] += 1;
a[2] += 1;
a[3] += 1;
a[4] += 1;
```
A full unroll is a special case where the unroll factor is equal to the number of loop iterations. Here, the compiler instantiates five adders instead of one adder.

### Example: Partial-Loop Unrolling

```c++
// Before unrolling loop
#pragma unroll 4
for(i = 0 ; i < 20; i++){
  a[i] += 1;
}

// Equivalent code after unrolling by a factor of 4
// The resulting loop has five (20 / 4) iterations
for(i = 0 ; i < 5; i++){
  a[i * 4] += 1;
  a[i * 4 + 1] += 1;
  a[i * 4 + 2] += 1;
  a[i * 4 + 3] += 1;
}
```
Each loop iteration in the "equivalent code" contains four unrolled invocations of the first. The compiler instantiates four adders instead of one adder. Because there is no data dependency between iterations in the loop, the compiler schedules all four adds in parallel.

### Determining the optimal unroll factor
In an FPGA design, unrolling loops is a common strategy to directly trade off on-chip resources for increased throughput. When selecting the unroll factor for a specific loop, the intent is to improve throughput while minimizing resource utilization. It is also important to be mindful of other throughput constraints in your system, such as memory bandwidth.

### Tutorial design
This tutorial demonstrates this trade-off with a simple vector add kernel. The tutorial shows how increasing the unroll factor on a loop increases throughput... until another bottleneck is encountered. This example is constructed to run up against global memory bandwidth constraints.

The memory bandwidth on an Intel&reg; Programmable Acceleration Card with Intel Arria&reg;, 10 GX FPGA system, is about 6 GB/s. The tutorial design will likely run at around 300 MHz. In this design, the FPGA design processes a new iteration every cycle in a pipeline-parallel fashion. The theoretical computation limit for one adder is:

**GFlops**: 300 MHz \* 1 float = 0.3 GFlops

**Computation Bandwidth**: 300 MHz \* 1 float * 4 Bytes   = 1.2 GB/s

You repeat this back-of-the-envelope calculation for different unroll factors:

|Unroll Factor  | GFlops (GB/s) | Computation Bandwidth (GB/s)
|:---  |:--- |:---
|1   | 0.3 | 1.2
|2   | 0.6 | 2.4
|4   | 1.2 | 4.8
|8   | 2.4 | 9.6
|16  | 4.8 | 19.2

On an Intel&reg; Programmable Acceleration Card with Intel Arria&reg; 10 GX FPGA, it is reasonable to predict that this program will become memory-bandwidth limited when the unroll factor grows from 4 to 8. Check this prediction by running the design following the instructions below.


### Additional Documentation
- [Explore SYCL* Through Intel&reg; FPGA Code Samples](https://software.intel.com/content/www/us/en/develop/articles/explore-dpcpp-through-intel-fpga-code-samples.html) helps you to navigate the samples and build your knowledge of FPGAs and SYCL.
- [FPGA Optimization Guide for Intel&reg; oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide) helps you understand how to target FPGAs using SYCL and Intel&reg; oneAPI Toolkits.
- [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide) helps you understand target-independent, SYCL-compliant programming using Intel&reg; oneAPI Toolkits.

## Key Concepts
* Basics of loop unrolling.
* How to unroll loops in your program.
* Determining the optimal unroll factor for your program.


## Building the `loop_unroll` Tutorial

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux*:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: `. ~/intel/oneapi/setvars.sh`
>
> Windows*:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
>
>For more information on environment variables, see **Use the setvars Script** for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).



### Running Samples in Intel&reg; DevCloud
If running a sample in the Intel&reg; DevCloud, remember that you must specify the type of compute node and whether to run in batch or interactive mode. Compiles to FPGA are only supported on fpga_compile nodes. Executing programs on FPGA hardware is only supported on fpga_runtime nodes of the appropriate type, such as fpga_runtime:arria10 or fpga_runtime:stratix10.  Neither compiling nor executing programs on FPGA hardware are supported on the login nodes. For more information, see the Intel&reg; oneAPI Base Toolkit Get Started Guide ([https://devcloud.intel.com/oneapi/documentation/base-toolkit/](https://devcloud.intel.com/oneapi/documentation/base-toolkit/)).

When compiling for FPGA hardware, it is recommended to increase the job timeout to 12h.


### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment, create launch configurations,
and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel&reg; oneAPI Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel&reg; oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.
 - (Linux only) Debug your GPU application with GDB for Intel&reg; oneAPI toolkits using the Generate Launch Configurations extension.

To learn more about the extensions and how to configure the oneAPI environment, see the 
[Using Visual Studio Code with Intel&reg; oneAPI Toolkits User Guide](https://software.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

### On a Linux* System

1. Generate the `Makefile` by running `cmake`.
     ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake .. -DFPGA_DEVICE=intel_s10sx_pac:pac_s10
   ```
   You can also compile for a custom FPGA platform. Ensure that the board support package is installed on your system. Then run `cmake` using the command:
   ```
   cmake .. -DFPGA_DEVICE=<board-support-package>:<board-variant>
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
     ```
     make fpga_emu
     ```
   * Compile for simulation (fast compile time, targets simulator FPGA device):
     ```
     make fpga_sim
     ```
   * Generate the optimization report:
     ```
     make report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     make fpga
     ```
3. (Optional) As the above hardware compile may take several hours to complete, FPGA precompiled binaries (compatible with Linux* Ubuntu* 18.04) can be downloaded <a href="https://iotdk.intel.com/fpga-precompiled-binaries/latest/loop_unroll.fpga.tar.gz" download>here</a>.

### On a Windows* System

1. Generate the `Makefile` by running `cmake`.
     ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake -G "NMake Makefiles" ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake -G "NMake Makefiles" .. -DFPGA_DEVICE=intel_s10sx_pac:pac_s10
   ```
   You can also compile for a custom FPGA platform. Ensure that the board support package is installed on your system. Then run `cmake` using the command:
   ```
   cmake -G "NMake Makefiles" .. -DFPGA_DEVICE=<board-support-package>:<board-variant>
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
     ```
     nmake fpga_emu
     ```
   * Compile for simulation (fast compile time, targets simulator FPGA device):
     ```
     nmake fpga_sim
     ```
   * Generate the optimization report:
     ```
     nmake report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     nmake fpga
     ```

> **Note**: The Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA and Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX) do not support Windows*. Compiling to FPGA hardware on Windows* requires a third-party or custom Board Support Package (BSP) with Windows* support.

> **Note**: If you encounter any issues with long paths when compiling under Windows*, you may have to create your ‘build’ directory in a shorter path, for example c:\samples\build.  You can then run cmake from that directory, and provide cmake with the full path to your sample directory.

### Troubleshooting
If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
``make VERBOSE=1``
For more comprehensive troubleshooting, use the Diagnostics Utility for
Intel&reg; oneAPI Toolkits, which provides system checks to find missing
dependencies and permissions errors.
[Learn more](https://software.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).

 ### In Third-Party Integrated Development Environments (IDEs)

You can compile and run this tutorial in the Eclipse* IDE (in Linux*) and the Visual Studio* IDE (in Windows*). For instructions, refer to the following link: [FPGA Workflows on Third-Party IDEs for Intel&reg; oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-oneapi-dpcpp-fpga-workflow-on-ide.html).

## Examining the Reports
Locate `report.html` in the `loop_unroll_report.prj/reports/` directory. Open the report in Chrome*, Firefox*, Edge*, or Internet Explorer*.

Navigate to the Area Report and compare the kernels' FPGA resource utilization with unroll factors of 1, 2, 4, 8, and 16. In particular, check the number of DSP resources consumed. You should see the area grows roughly linearly with the unroll factor.

You can also check the achieved system f<sub>MAX</sub> to verify the earlier calculations.

## Running the Sample

1. Run the sample on the FPGA emulator (the kernel executes on the CPU):
     ```
     ./loop_unroll.fpga_emu     (Linux)
     loop_unroll.fpga_emu.exe   (Windows)
     ```
2. Run the sample on the FPGA simulator device:
     ```
     ./loop_unroll.fpga_sim     (Linux)
     loop_unroll.fpga_sim.exe   (Windows)
     ```
3. Run the sample on the FPGA device:
     ```
     ./loop_unroll.fpga         (Linux)
     loop_unroll.fpga.exe       (Windows)
     ```

### Example of Output
```
Input Array Size:  67108864
UnrollFactor 1 kernel time : 255.749 ms
Throughput for kernel with UnrollFactor 1: 0.262 GFlops
UnrollFactor 2 kernel time : 140.285 ms
Throughput for kernel with UnrollFactor 2: 0.478 GFlops
UnrollFactor 4 kernel time : 68.296 ms
Throughput for kernel with UnrollFactor 4: 0.983 GFlops
UnrollFactor 8 kernel time : 44.567 ms
Throughput for kernel with UnrollFactor 8: 1.506 GFlops
UnrollFactor 16 kernel time : 39.175 ms
Throughput for kernel with UnrollFactor 16: 1.713 GFlops
PASSED: The results are correct
```

### Discussion of Results
The following table summarizes the execution time (in ms), throughput (in GFlops), and number of DSPs used for unroll factors of 1, 2, 4, 8, and 16 for a default input array size of 64M floats (2 ^ 26 floats) on Intel&reg; Programmable Acceleration Card with Intel&reg; Arria&reg; 10 GX FPGA:

Unroll Factor  | Kernel Time (ms) | Throughput (GFlops) | Num of DSPs
|:--- |:--- |:--- |:---
|1   | 242 | 0.277 | 1
|2   | 127 | 0.528 | 2
|4   | 63  | 1.065 | 4
|8   | 46  | 1.459 | 8
|16  | 44  | 1.525 | 16

Notice that when the unroll factor increases from 1 to 2 and from 2 to 4, the kernel execution time decreases by a factor of two. Correspondingly, the kernel throughput doubles. However, when the unroll factor is increased from 4 to 8 or from 8 to 16, the throughput no longer scales by a factor of two at each step. The design is now bound by memory bandwidth limitations instead of compute unit limitations, even though the hardware is replicated.

These performance differences will be apparent only when running on FPGA hardware. The emulator, while useful for verifying functionality, will generally not reflect differences in performance.

## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).
