# An Introduction to SURF and Ruckus

## Disclaimer

This document is a personal summary of the libraries, and I am not affiliated with the authors of the libraries. The interpretations presented might not be accurate.


## Abstract

The Ruckus Build System is a comprehensive tool that streamlines the development process of firmware for FPGAs and other programmable devices. It acts as a bridge between version control systems and hardware description languages, embedding version information directly into the build process. The system utilizes a hybrid Makefile/TCL structure to manage builds, enabling both command-line operation for efficiency and a GUI for increased usability and error analysis. Ruckus simplifies the process of setting up and replicating Vivado project environments, mitigating the need to version control the entire project workspace, and provides tools for automating release tagging and documentation. Its integration with Git allows developers to track changes and maintain a clear history of their project’s evolution. With the ability to handle multiple hardware targets and support a wide range of Xilinx FPGAs, Ruckus stands out as a flexible and robust build  
system tailored for complex digital design tasks. Managed by the TID-ID Electronics Systems Department, it is a well-maintained solution that receives regular updates to support the latest toolchains and technologies


## Ruckus: Vivado Build System

Ruckus is an open-source tool that complements the SURF firmware library, designed to enhance FPGA development workflows with a hybrid Makefile/TCL system. It integrates the Git repository directly into the Xilinx Vivado build process, offering unique features such as embedding the Git hash of the build as a readable register within the project, facilitating traceability and version control. Ruckus provides scripts for tag releasing and generating release notes, along with TCL helper functions to simplify adding code into the Vivado project. It enables users to reproduce a Vivado project environment without the need to revision control the entire Vivado project, making it easier to recover from issues by allowing a simple project environment reset using commands like `$ make clean` followed by `$ make gui`. Managed and maintained by the TID-ID Electronics Systems Department, Ruckus is described as fairly mature and stable, requiring updates primarily for compatibility with new releases of Vivado.

### Ruckus Build System: File Structure: Overview/Definition



![](attachments/Pasted%20image%2020240315165601.png)


The "ProjectBase" directory acts as the core organizational structure for both firmware and software elements in a project. Within this framework, the "firmware" directory is structured according to a specific standard to ensure orderliness, while the "software" directory does not adhere to any particular file structure, given its independence from the ruckus build system. External git repository source codes are consolidated in the "submodules" directory. The "targets" directory is crucial as it includes the top-level project information and Makefiles, though it contains minimal source code itself. To facilitate efficiency in output management for both targets and simulations, the "build" directory serves as a common repository, typically linked to a server's hard drive for improved runtime performance. Lastly, the "common" directory is designated for source code that is universally applicable across all targets, promoting code reuse and consistency across the project.

### Ruckus Build System: File Structure: Submodules


The "submodules" directory is integral to the project structure, housing all external git repository source codes necessary for the project's build system. Specifically, for the ruckus build system to function correctly, only the "ruckus" and "surf" submodules are required. It is strongly advised that users refrain from developing directly within a submodule; instead, they should work exclusively with versions of submodules that have been officially tagged in git to ensure stability and reliability. For example, a stable project setup might utilize the "ruckus" submodule tagged as "v4.0.0" and the "surf" submodule tagged as "v2.24.2", indicating the specific versions of these submodules that the project depends on.

### Ruckus Build System: Example File Structure

The Ruckus Build System's example file structure showcases an organized approach to firmware development, featuring a `firmware` directory that contains a symbolic link to a `build` directory located at `/u1/ruckman/build`. Within `firmware`, there are subdirectories such as `common`, `ip`, and `rtl` for various code bases and components. The `submodules` directory includes essential components like `ruckus` and `surf`. Finally, the `targets` directory houses the project-specific files, in this case, `Simple10GbERudpKcu105Example`, encapsulating the top-level project's structure and makefiles for the build process.


### Ruckus Build System: What’s inside a target directory

The Ruckus Build System target directory is laid out to facilitate the organization and management of the build process. It contains a `hdl` subdirectory for local VHDL sources, an `images` directory where the output files are stored, and a `Makefile` which serves as the local Makefile. Additionally, there is a `tb` folder for test bench files and a `vivado` directory that includes TCL scripts such as `post_synthesis.tcl` for post-synthesis processes and `promgen.tcl` for PROM configurations. Key files like `ruckus.tcl` provide steering instructions for incorporating source code into Vivado projects. This structure exemplifies a clean and efficient approach to managing FPGA build environments within the Ruckus Build System.


![](attachments/Pasted%20image%2020240315165913.png)



The Ruckus Build System's target directory structure, as shown in the image, includes a `build.info` file generated at the start of each build, documenting the Git configuration. The `hdl` directory contains VHDL design files, while the `images` directory holds various output files such as bitstreams, PROM files, and compressed versions for Git repository commits, each labeled with version numbers, build timestamps, and shortened Git hashes. Additionally, there's a `Makefile` for build automation, `ruckus.tcl` for script execution, and a `vivado` directory containing TCL scripts like `promgen.tcl` for PROM configurations, demonstrating an organized and detailed approach to firmware development within the build system.

### Ruckus Build System: targets/DESIGN/Makefile

![](attachments/Pasted%20image%2020240315170044.png)


The Ruckus Build System's Makefile within the `targets/DESIGN` directory provides a structured approach for defining and managing the build process of a design project. It specifies the target output (such as PROM or bitstream), sets the firmware version manually by the user, and identifies the FPGA part number, ensuring it aligns with Vivado's part number convention. The Makefile also establishes the top directory path, relative to the firmware directory, and can include debugging capabilities through the XVC Debug Bridge. Release configurations are managed and named to correspond with the project, and the Makefile incorporates higher-level makefiles from the Ruckus system directory to streamline the build process. These settings are crucial for maintaining consistency, traceability, and control over the design and build lifecycle within the Ruckus Build System.


### Ruckus Build System: targets/DESIGN/ruckus.tcl


The `ruckus.tcl` file within the Ruckus Build System serves as a script for initializing and setting up the build environment for a design project. It starts by loading the RUCKUS TCL environment and library, ensuring all necessary procedures and configurations are in place. The script continues by loading common ruckus.tcl files, which include those from the `surf` submodule and the common directory, integrating essential submodules and shared resources. It also handles the loading of local source code and constraints from specified directories, which aligns with the project's structural requirements. Additionally, it manages the local simulation source code, facilitating the simulation-only build processes and setting the top-level properties for the project. This script acts as a crucial orchestrator, piecing together various components and configurations required for a successful build process in the Ruckus Build System.


### Ruckus Build System: loading source code TCL procedures

The Ruckus Build System incorporates several TCL procedures for loading source code into a Vivado project, which can be found in the `vivado\_proc.tcl` script. The `loadRuckusTcl()` function is pivotal for recursively loading source code from various `ruckus.tcl` files, creating a structured hierarchy of steering files. The `loadSource()` function handles the loading of various source code files (.vhd, .v, .vh, .sv, .dcp), with flags for simulation-only tagging and targeting either single files or all files within a directory in a non-recursive manner. The `loadIpCore()` function is specifically for loading Xilinx IP cores (.xci or .xcix files), while `loadBlockDesign()` deals with Xilinx IP integrator designs (.bd or .tcl files). Lastly, `loadConstraints()` is used for loading constraints (.xdc or .tcl files), following the same pattern for file targeting as the previous functions. These procedures collectively facilitate a modular and efficient build process within the Ruckus framework.


### ruckus.tcl examples: Creating a hierarchy of steering files

![](attachments/Pasted%20image%2020240315170332.png)

Figure illustrates the Ruckus Build System's process for creating a structured hierarchy of steering files using `ruckus.tcl` scripts. The initial `ruckus.tcl` file from the target directory loads additional `ruckus.tcl` files from the common and submodules/surf directories, establishing a flow of dependencies. These in turn point to specific `ruckus.tcl` files within the submodules/surf directory, which are responsible for loading various Ruckus files associated with different components such as axi, base, dsp, devices, ethernet, protocols, and xilinx. This structure effectively organizes and manages the incorporation of different modules and code segments, contributing to a modular and maintainable codebase within the firmware development environment.

### ruckus.tcl examples: Loading source code


The provided text details various `ruckus.tcl` procedures for loading source code into a project. The `loadSource` command can be used with the -path option to load a single file, or the `-dir` option to load all files from a directory, applicable to both synthesis and simulation. There's also a `-sim_only` flag to load only simulation files, either a single one with `-path` or all from a directory with `-dir`. Similarly, the `loadConstraints` command is available to load constraint files, also with `-path` and `-dir` options for single or directory-based loading. These commands provide flexibility for handling different types of source files and constraints during project setup.

The text outlines procedures for incorporating Xilinx IP core and block design files into a project using the `ruckus.tcl` script. The `loadIpCore` command can either load a single Xilinx IP core file using the `-path` option or all IP core files from a specified directory using the `-dir` option. Similarly, the `loadBlockDesign` command allows for the loading of a single Xilinx block design file with the `-path` flag or all block design files from a directory with the `-dir` flag. These commands streamline the process of adding Xilinx-specific components to a Vivado project, ensuring that all necessary design elements are included accurately.


### Build System: Design Flow: Both GUI and Batch mode


The build system described offers both a GUI and batch mode for design flow, with each mode presenting distinct advantages and drawbacks. The batch mode boasts high-performance building with minimal overhead but suffers from cryptic error messaging and challenges in design analysis from terminal outputs. On the other hand, the GUI mode allows for native Vivado interaction and user-friendly error messaging, though it is more demanding on X11 resources and has a longer runtime than batch mode. A typical design flow would involve entering the GUI mode to find and analyze synthesizing errors, exiting to run the batch mode for .bit and/or .mcs file generation, and then, if necessary, returning to GUI mode to troubleshoot any implementation errors. Once the batch mode is complete, users may revisit the GUI mode to review detailed implementation reports without having to clean the makefile, thus maintaining a seamless and efficient development process.

### Build System: Simulation


The build system supports two modes for simulation: the Vivado Native Simulator and the Synopsys VCS Simulator. The Vivado Native Simulator is advantageous for quickly prototyping and simulating firmware without the need for an external software interface. However, it lacks an external software interface, which can be a limitation. The Synopsys VCS Simulator, on the other hand, can generate VCS build scripts using the Vivado framework. It allows for the attachment of a software interface through a shared memory interface and can simulate mixed signals, incorporating both analog and digital signals. The downside of the VCS Simulator is that it requires additional setup time compared to the Vivado Native Simulator. Both simulation modes offer distinct benefits and challenges, catering to different needs in the firmware development process.


### Design Flow: IP Core Generation	

![](attachments/Pasted%20image%2020240315170749.png)

The typical design flow for IP core generation begins with opening the GUI mode using the "make gui" command. Within the GUI, you access the "IP Catalog" to select and configure the desired IP. Once the IP core is generated, you close the GUI mode and copy the resulting .xci file from the build directory to the firmware/common directory. The next step is to update the ruckus.tcl file to reference the new Xilinx IP core file. Any obsolete files in the build directory should be cleaned out with the "make clean" command. Afterward, you can re-open the GUI mode to confirm that the new IP is correctly included in the "IP Sources" section, completing the IP core integration process.


## Additional Libraries

### SLAC Ultimate RTL Framework (SURF) Firmware

The SURF library is an extensive open-source VHDL library designed for FPGA development, compatible with Xilinx FPGAs, Intel FPGAs, and ASIC digital designs. It offers a wide range of VHDL-based IP modules for commonly implemented functionalities across various categories such as Ethernet, AXI4, AXI4-Lite, AXI4 stream, device-specific libraries, synchronization, wrapped Xilinx IPs, and serial protocols. Key features include support for Ethernet protocols like 1000BASE-KX, 10Gbase-KR, XAUI, IPv4, ARP, DHCP, ICMP, UDP; comprehensive AXI4 libraries for crossbar, DMA, FIFO, etc.; device libraries for manufacturers like ADI, Micron, SiliconLabs, TI; synchronization libraries; wrapped Xilinx libraries for clock management, SEM, DNA, IPROG; and serial protocols including I2C, SPI, UART, line-code, JESD204B. Additionally, it includes SLAC Protocols with documentation available at a specified SLAC Confluence page. Managed and maintained by the TID-ID Electronics Systems Department, SURF undergoes regular updates with new features and bug fixes on a weekly basis.

### Rogue Software


Rogue Software is an open-source platform, found on GitHub, that complements the SURF firmware library and is designed to operate in either a Python/C++ hybrid mode or C++ only mode. It supports a variety of architectures including x86-64, ARM32, and ARM64, and is compatible with Linux, Mac, and Windows operating systems, although the latter may require additional layers like WSL2, Cygwin, or a Linux virtual machine. Rogue facilitates both rapid prototyping and the deployment of experiments with various interface options, such as non-GUI Python scripts, EPICS, and Python GUIs with libraries like PyQt or PyDM. It utilizes ZeroMQ for connecting multiple heterogeneous DAQ software clients to a single server, enhancing operational efficiency. The maintenance and management of Rogue fall under the responsibility of the TID-ID Electronics Systems Department.


### PCIe FW/SW Framework


The PCIe FW/SW Framework is an open-source firmware framework available on GitHub, designed to work with BARO AXI-Lite interface and supports up to 8 DMA lanes, with each lane capable of handling 256 TDEST per DMA (up to a total of 2048 destinations). It serves as a unified firmware framework and software kernel driver for any Xilinx PCIe card, offering extensive support for a range of Xilinx Development and Data Center Cards. The framework is capable of achieving high data transfer rates, demonstrating up to 103 Gb/s for large frames and over 4MHz frame rate for small frames without the need for frame batching. The list of supported cards is extensive and expandable as needed. This framework is managed and maintained by the TID-ID Electronics Systems Department, ensuring its reliability and up-to-date functionality.

### Zynq Ultrascale+ FW/SW Framework

The Zynq Ultrascale+ FW/SW Framework is an open-source firmware framework designed for AXI-Lite interfaces with support for up to 8 DMA lanes and the capability of handling 256 TDEST per DMA lane. This framework provides a standardized platform for both firmware frames and software kernel drivers tailored for any Zynq Ultrascale+ device. It offers extensive support for a range of common Xilinx Development boards, including various versions of RFSoC boards and SLAC's 2nd Generation NASA RFSoC Board. The framework is scalable, allowing for additional Zynq Ultrascale+ board support as needed. It's a relatively new development that draws inspiration from the work on axi-pcie-core and is maintained by the TID-ID Electronics Systems Department, ensuring its continuous evolution and reliability.

