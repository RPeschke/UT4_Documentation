# sim_subdetector_top_axi4_wrap_poll_tb


  The `subdetector_top_axi4_wrap_poll_tb` is the main simulation in this firmware project. It was first made to work with ISE, but now it can also run on the Vivado simulator. It includes several modules: `test_pattern_storage`, `trg_logic` with the old trigger system, `straightline_fit_full_array_interface_external_trigger` with the new straight line fitter, `trg_cntr`, and `cntr_mgr`. Inside the `straightline_fit_full_array_interface_external_trigger` module, there's the `straightline_fit_full_array_interface`, which is the top level for the Straight Line Fit (SLF).


## Running the simulation

To execute the simulation, the Python program "fmake" is utilized. It can be installed using the command `pip install fmake`. After installation, "fmake" requires initialization. This is done by running the command `fmake make-build` within the top-level directory of the project, in this instance, the "klm-trg" folder.

```bash
# installing fmake
pip install fmake


# initilizing fmake 
fmake make-build

```


Following the initialization, a new directory named "build" will appear in the project's main folder. "fmake" utilizes this directory to store temporary files, which should be excluded from any version control systems like Git.

With "fmake" now ready, you can proceed to create simulation projects using the command:



```bash
fmake make-simulation --entity subdetector_top_axi4_wrap_poll_tb
```


This step may require some time to complete. If it appears to be taking too long, you can obtain more detailed information about what "fmake" is doing by increasing the verbosity level with the following command:


```bash
fmake make-simulation --entity subdetector_top_axi4_wrap_poll_tb --verbosity 10
```

During this process, "fmake" scans all files to identify those associated with the specified entity. It is important to note a known issue where the presence of a symbolic link to another directory within the project can trigger infinite recursion. This problem can be circumvented by placing a ".fmakeignore" file in the problematic directory. The ".fmakeignore" file does not need to contain any specific content; its mere presence ensures the directory will be ignored by "fmake."


After the project has been set up, it can be executed by using the following command:

```
fmake run-vivado --entity subdetector_top_axi4_wrap_poll_tb --gui
```


This command launches Vivado and initiates the simulation, which may take some time to start. Alternatively, the project can also be run in ISE with the command:


```
fmake run-ise --entity subdetector_top_axi4_wrap_poll_tb --gui
```


Both commands open the respective simulation environments with a graphical user interface, allowing  to visually monitor and interact with the simulation process.




Once the simulation is up and running, interaction with it can be facilitated through a Python interface, with a Jupyter notebook being the most convenient tool for this purpose. Within the project folder, you'll find a notebook named:

`sim_subdetector_top_axi4_wrap_poll_tb`

This notebook, while basic and lacking in comprehensive features, provides the capability to interact with the simulation. It introduces a class named `subdetector_top_axi4_wrap_poll_tb`, designed for top-level interaction with the simulation's register handler. This class enables users to set register values and retrieve debug data.

For instance, using the command:


`hex(reg.slf_debug_data())`


will fetch the debug data, which defaults to `"0xbe11e"` if no specific setting is applied. Then, by altering register 504 to 10:

`reg.set_register(504, 10)`

the expected output would be `0`.

Although there are additional Python scripts that could simplify interaction with the simulation, they have not been integrated into this branch yet.


## straightline_fit_full_array_interface

```VHDL
entity straightline_fit_full_array_interface is
  generic (
    this_subdetector : in subdetector_t
  );
  port (
    clk            : in  STD_LOGIC;
    rst            : in  STD_LOGIC;
    reg            : in  register_32T;

    debug_data_out : out std_logic_vector(31 downto 0);
    hit_1d_arr     : in  Trg1DHitArrayType;
    veto           : in  std_logic     := '0';
 

    legacy_trigger : in  legacy_trigger_type;

    trg_tf_vec     : out STD_LOGIC_VECTOR(
					    NUMBER_OF_LINKS - 1 downto 0) 
  

  );

end entity;
```

The simulation features several ports for its operation:
 **Generic**:
1. **this_subdetector**: Determines if the setup is for barrel or encap.


**ports:** 
1. **Clock and Reset Ports**: Used for synchronization and initialization.
2. **Register Input**: Sends register values to all subcomponents, common across nearly all components.

Additionally:

4. **Debug Data Out Port**: Connects the "slf_event_debugger" module to the register handler, facilitating debug data output.
5. **Trg1DHitArrayType**: An input data stream consisting of an array of 16 KLMDIGITS.
6. **Veto Signal**: Currently unused in the module.
7. **Legacy Trigger Input**: Feeds legacy trigger information into the module.
8. **Trg Tf Vec**: Outputs trigger information generated by the module.


Overview of Functionality: The module begins by merging data streams due to the current FPGA's limitation in handling parallel data processing. Initially, it consolidates the data streams. Then, based on the "this_subdetector" generic, it selects either the "geometryConverter_full" or "geometryConverter_full_EKLM". The choice depends on whether the data pertains to the barrel or the EKLM, with the latter's geometry being simpler and requiring less complex processing. Following this, the "linerFitter_full" takes over to perform the fitting process. Finally, the "slf_event_debugger" module is employed to provide debugging capabilities.



In the ISE version, the data stream merger utilized FIFOs created by ISE. However, transitioning these FIFOs to Vivado encountered compatibility issues. As a workaround, older FIFOs coded in pure VHDL were employed. These VHDL-based FIFOs do not match the timing performance of the original ISE-generated ones, potentially leading to timing violations during compilation. Nevertheless, for simulation purposes, they are adequate. It may be advisable to consider replacing them with FIFOs from the "SURF" library for improved reliability and performance.


Within this segment, the KLMDIGITS undergo serialization and deserialization, employing a compact serializer. This means not all attributes of the KLMDIGIT record ("Trg1DHitFormat") are included in the serialization process. Specifically, only the following attributes are serialized: subdetector, Sector, Layer, Channel, Axis, and Timestamp.



The timestamp data is utilized solely by the "slf_event_debugger," and its presence is not strictly necessary. Thus, it presents an opportunity for further streamlining by potentially eliminating this signal.

The module contains a specific process block dedicated to merging the data streams. This block is connected to the data FIFOs, it handles the reading out of these FIFOs. To interact with the FIFOs, the AXI stream protocol is employed, a standard approach for managing data streams across different modules within this setup. The handling of the AXI stream interface is managed through pseudo-classes, with the "in_fifo_rx" variable overseeing access to the data stream. This variable features two key methods: "isReceivingData" and "read_data." When the AXI stream is active, "isReceivingData" returns true, allowing the "read_data" method to be executed, thereby encapsulating the entire data stream management process.

Data from the KLMDIGITS is dispatched via "fifo_hit1d" to either the "geometryConverter_full" or "geometryConverter_full_EKLM" module. At this juncture, the data undergoes conversion from Digits to 1D hits ("geo_hit1d"), which includes details such as subdetector, sector, axis, and the coordinates x and y.

These 1D hits are then relayed to both the "linerFitter_full" and the "slf_event_debugger." Additionally, the "linerFitter_full" receives data from the legacy trigger, which dictates the overall timing. The generation of the SLF trigger is synchronized with the legacy trigger, occurring within the same clock cycle. This synchronization ensures consistent timing behavior, which is critical for seamless integration with other components like the  the GDL/GDR.

## geometryConverter_full


### Ports


```VHDL
entity geometryConverter_full is
  port (
      clk                      : in std_logic;
      rst                      : in std_logic;
      reg                      : in register_32T;
      
      this_subdetector         : in  subdetector_t;
-- Data Stream  Input
      hit_1d_i                 : in Trg1DHitFormat;
-- Debug output      
      hit_1d_i_missed_events   : out std_logic_vector(15 downto 0);

-- data Stream output
      hit1d_out                : out hit1d := hit1d_null;
      hit1d_out_ready          : in std_logic := '0'
  );
end entity;
```



The "geometryConverter_full" module is responsible for converting the geometry for the BKLM, incorporating the three conventional ports: clock (clk), reset (rst), and register input (reg). Additionally, it includes a "this_subdetector" port, which remains unused. The module receives KLMDIGITS through the "hit_1d_i" input port. Without the capability to apply backpressure, the module risks data loss if a KLMDIGIT arrives while its input FIFO is full. To monitor such occurrences, there's an "hit_1d_i_missed_events" output port, linked to the "slf_event_debugger," which ideally should register zero events but is crucial for operational monitoring. The module outputs data through the "hit1d_out" and "hit1d_out_ready" ports, adhering to AXI stream synchronization principles, requiring both 'valid' and 'ready' signals to be high during the same clock cycle for data transmission.


### Input Process

```VHDL 
process(clk) is 
variable buff  : std_logic_vector(31 downto 0) := (others => '0');
variable v_in_fifo_RX : axi_stream_32_m := axi_stream_32_m_null;
begin
if rising_edge(clk) then
    pull(v_in_fifo_RX, in_fifo_RX.s2m);
    if rst = '1' then
        i_missed_event_counder <= (others =>'0');
    end if;
    hit_1d_i_missed_events <= std_logic_vector(
                                i_missed_event_counder
                                );
    
    if hit_1d_i.WEN = '1' and ready_to_send(v_in_fifo_RX)   then
        buff := Trg1DHitFormat_Compact_serialize(hit_1d_i);
        send_data(v_in_fifo_RX , buff);
    elsif hit_1d_i.WEN = '1' then
        i_missed_event_counder <= i_missed_event_counder +1;
    end if;
    
    push(v_in_fifo_RX, in_fifo_RX.m2s);
end if;

end process;
```

The process for handling input is direct: it verifies whether a KLMDIGIT is present at the hit_1d input. If a hit is detected and the corresponding FIFO is not full—indicating that by AXI stream interface is prepared to transmit—the data is serialized and dispatched to the FIFO using the AXI stream protocol. This step utilizes pseudo-classes specifically designed to manage the AXI stream interface connection.

It's important to note that during this process, the timestamp component of the data is omitted.

Should the process encounter a KLMDIGIT while the FIFO lacks readiness to accept new data, the incoming hit is discarded, and the missed_event_counter is incremented to reflect the lost data.


### Main Process


  


The central process involves retrieving data from the FIFO and subsequently fetching the relevant conversion factors from the RAM block. For each unique combination of Sector, Layer, and Axis, there exists a specific pair of slope and offset values used to convert KLMDIGITs inputs into x and y coordinates. This conversion utilizes a linear equation, where the x-coordinate is calculated as

$x = x_{\text{slope}} \times \text{layer} + x_{\text{offset}}$

and y-coordinate is calculated as: 

$y = y_{\text{slope}}(\text{sector}, \text{layer}, \text{axis}) \times \text{channel} + y_{\text{offset}}(\text{Sector}, \text{layer}, \text{axis})$


Each address in the RAM block matches a specific combination of sector, layer, and axis. The function `convert_ram_addr` is utilized to map between these combinations and their corresponding RAM addresses. Access to the RAM block is facilitated by a pseudo-class named `mem_handler_t`, which offers several member functions for managing memory operations.

The `write_data` function allows writing information to a specified address within the memory, completing this action within a single clock cycle. To initiate a read operation, the `set_read_addr` function is used to designate a specific address for reading. Once the address is correctly set, the `is_read_addr` function will return true, indicating the readiness to access the requested data through the `read_data` function. Given the deterministic duration of data retrieval from the memory block, this step offers potential for pipelining as an enhancement for future implementation.

After successfully retrieving the slope and offset parameters, they are applied to the KLMDIGITS, transforming them into 1dHITS. These 1dHITS are subsequently transmitted to the next block via the AXI stream interface.


## geometryConverter_full_EKLM


The "geometryConverter_full_EKLM" operates similarly, with the primary difference being the absence of RAM blocks for slope and offset values. Instead, the determination of slope and offset depends solely on whether it is for the Forward or Backward endcap, resulting in just four sets of slopes and offsets. Consequently, storing this information does not necessitate a dedicated memory module.