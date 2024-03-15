
# Build Systems

Transitioning to the theme of build systems, it's essential to recognize a fundamental truth: no build system is perfect, and our choice is essentially about selecting the type of imperfection we're willing to work with. Currently, our project utilizes three different build systems. It began with HDLmake, to which I added a custom build system named fmake (for simulation purposes only), and now we're transitioning to Ruckus. The shift to Ruckus was motivated by our desire to incorporate the SURF libraries from SLAC into our system, though this migration has proven more challenging than expected.

## HDLmake

HDLmake, developed by CERN, has been an adequate build system for our projects. Interestingly, there seems to be a prevalent trend among build system developers to integrate "make" into the development process. This approach essentially involves using one programming language to invoke another, purportedly simplifying the process - or so it seems.

I'm not keen on the idea of employing one programming language to invoke another, adding unnecessary complexity to the development process. Nonetheless, this is the design approach that has been adopted.


![](attachments/Pasted%20image%2020240315161103.png)

The image depicts our previous file structure before transitioning to Ruckus, with a top folder housing the makefile. This folder accumulates all temporary build files and the final bitfile, though surprisingly, it doesn't utilize an "out of source" build directory as typically expected. The reasons remain unexplored due to lack of investigation on my part. Notably, our repository includes HDLmake as a submodule, a decision I reluctantly accept. Given the fragile and unstable nature of firmware development, using a consistent build system version across the board does offer some rationale.


![](attachments/Pasted%20image%2020240315161131.png)

The "Manifest.py" files function analogously to modules in Python, with each folder recognized as a module. These Manifest files detail the components of the module, allowing for the setting of variables, inclusion of other modules, and addition of files. Unlike CMake, which assigns variables to a target object, the Manifest approach opts for direct variable declarations at the top level of the file. Which of course comes with all the drawbacks that forced CMake to adopt the target model.

The setup is practical and user-friendly. Using a target-based method like CMake could simplify managing multiple targets. There's a chance we're not utilizing all the capabilities of HDLmake. Plus, the version we're using is a few years old, so there might be updates and enhancements we haven't explored.

![](attachments/Pasted%20image%2020240315161159.png)

The displayed Manifest.py file mainly involves adding modules and files, typical of most Manifest.py structures. Being a Python script, it allows for executing extra Python code, such as generating a version file with a VHDL constant matching the Git commit hash, a feature utilized in the KLMTRG firmware. However, this function only activates when using HDLmake for ISE, not when synthesizing directly through the ISE GUI, where the version file won't get updated.

![](attachments/Pasted%20image%2020240315161209.png)

The makefile serves as an extension to the HDLMake library, incorporating code that didn't fit within the HDLMake framework. It begins by checking if the XILINX environment variable is set, a necessary step for internally generating a TCL script. This script, in turn, is executed by ISE suite programs to produce a .xise project file. The makefile also defines the HDLMAKEFILE, which is created upon the initial execution of "make" and includes target-specific information like file paths. This setup enables commands like "project," "synthesize," and "bitstream," and allows for opening the ISE GUI with "ise_gui" once the project file is established.

After setting it up, this build system operates smoothly and even supports generating Vivado project files. Essentially, it fulfills all our requirements.

## Build System: fmake

[sim_subdetector_top_axi4_wrap_poll_tb](../Simulation/sim_subdetector_top_axi4_wrap_poll_tb.md)

The fmake build system streamlines simulation processes for both ISE and Vivado, requiring no build files. To prepare for simulation, one initiates a build folder at the project's root using "fmake make-build." Simulating, for example, the KLMTRG's main test bench necessitates just a couple of commands:

To prepare the simulation environment:

fmake make-simulation --entity subdetector_top_axi4_wrap_poll_tb

To launch the simulation in ISE with a graphical interface:

fmake run-ise --entity subdetector_top_axi4_wrap_poll_tb --gui

This system adequately serves its intended simulation purposes without plans for further expansion.

The fmake build system is available for installation via PyPI and can be easily added to your environment using the pip command: pip install fmake.

## The Ruckus Build system

The Ruckus build system, developed by SLAC, integrates into their broader suite of tools. It uses TCL, MAKE, and some Python, paralleling Manifest.py by utilizing ruckus.tcl files to manage file inclusions. Despite efforts, successful firmware synthesis with Ruckus has yet to be achieved.

![](attachments/Pasted%20image%2020240315161230.png)

The updated layout is neatly divided into firmware and software sections. In the Firmware directory, a build folder is immediately noticeable, where Ruckus enables an "out of source" build process. The shared folder houses firmware files common across various targets, while the submodules folder contains essential components like the Ruckus system and SURF library, critical for Ruckus's operation. Usage of tagged submodule releases is strongly advised. The targets folder includes the top-level HDL files and the Ruckus entry point for specific targets.

![](attachments/Pasted%20image%2020240315161255.png)

The makefile serving as the entry point for Ruckus is streamlined, containing only the essential information needed to initiate Ruckus, such as the FPGA type. For more detailed configuration, it employs TCL files (ruckus.tcl), which aligns well with VIVADO's use of TCL as its primary scripting language.