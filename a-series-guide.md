---
title: TinyFPGA A-Series User Guide
---

## Getting Started
The TinyFPGA A-Series boards use Lattice Semiconductor's MachXO2 FPGAs.  There are a number of existing software and hardware tools available as well as documentation from Lattice for these FPGAs.  This guide will help get you started with the A-Series boards and the tools and information specific to them as well as the tools and documentation available for the FPGA chips themselves.

### Hardware
Of course you will need to purchase one or more A-Series boards, but you will also need a few other things to get working.

1. TinyFPGA [A1](http://store.tinyfpga.com/product/tinyfpga-a1) or [A2](http://store.tinyfpga.com/product/tinyfpga-a2) Board.
2. [Pins](http://store.tinyfpga.com/product/a-series-pins) if you want to solder the board to another PCB, insert it into a socket, or use with a solderless breadboard.
3. 3.3 volt power supply.  You can use a [3.3 volt regulator](http://store.tinyfpga.com/product/a-series-3-3-volt-regulator-ld1117v33), a lab power supply, or use an existing 3.3 volt power supply from your project.
4. [JTAG programmer](https://www.ebay.com/sch/i.html?_nkw=lattice+fpga+jtag).  TinyFPGA currently doesn't have a JTAG programmer, but I hope to soon have an opensource Arduino sketch available soon and very low-cost programmer hardware later.
5. [Something](https://www.google.com/search?q=godzilla+robot&safe=active&tbm=isch) [interesting](https://www.google.com/search?q=quad+copter&safe=active&tbm=isch) [to](https://www.google.com/search?q=3d+printer+open+source&safe=active&tbm=isch) [control](https://www.google.com/search?q=vga+graphics&safe=active&tbm=isch) [or](https://www.google.com/search?q=retro+console&safe=active&tbm=isch) [interface](https://www.google.com/search?q=retro+computer&safe=active&tbm=isch) [with](https://www.google.com/search?q=tcp+ip&safe=active&tbm=isch).  If you are just starting out you could use some LEDs or maybe a logic analyzer.  Otherwise you might have something more specific in mind ;)

### Software
You will need to install the latest development environment and other support tools for the MachXO2 FPGAs and the A-Series boards.

1. Download and install [Lattice Diamond](http://www.latticesemi.com/latticediamond).  It is available for both [Windows](http://www.latticesemi.com/latticediamond#windows) and [Linux](http://www.latticesemi.com/latticediamond#linux).
2. [Request a free license file](http://www.latticesemi.com/Support/Licensing/DiamondAndiCEcube2SoftwareLicensing/DiamondFree.aspx) in order to use the [Lattice Diamond](http://www.latticesemi.com/latticediamond) software.
3. The TinyFPGA A-Series GitHub Repository has Lattice Diamond template projects that you may find useful.  They include an empty top-level verilog module with pin constraints to map board pins to the correct IOs on the MachXO2 FPGA chip.  You could [download the latest files directly in a zip file](https://github.com/tinyfpga/TinyFPGA-A-Series/archive/master.zip) or [clone the repo using git](https://github.com/tinyfpga/TinyFPGA-A-Series.git).

### First Project

Once you have all of your hardware and software ready you can get started developing some digital logic.  This first project won't go into all the details of designing and implementing digital logic circuits in general, but it will guide you through the specifics of setting up a simple project, writing verilog, generating a bitstream for your TinyFPGA A1 or A2 board, and programming your board with the bitstream. 

#### 1. Copy the template project from the [TinyFPGA A-Series Repository](https://github.com/tinyfpga/TinyFPGA-A-Series/archive/master.zip)

Copy either the [`template_a1`](https://github.com/tinyfpga/TinyFPGA-A-Series/tree/master/template_a1) or [`template_a2`](https://github.com/tinyfpga/TinyFPGA-A-Series/tree/master/template_a2) directories to a new directory and rename it `blink_project`.

#### 2. Open your newly copied template project

Open the Lattice Diamond application.  From the `File` menu select `Open` and `Project...`.  In the newly opened file chooser, navigate to the `blink_project` directory you just created and select the `template_a1.ldf` or `template_a2.ldf` project file.

#### 3. Implement your logic

Now that we have opened our new project we can write some verilog code.  Make sure the `File List` tab is open on the left-hand side view and open up the `TinyFPGA_A1.v` or `TinyFPGA_A2.v` verilog file.

This is a very simple top-level verilog module that represents the IO pins available on the TinyFPGA A-Series boards.  Right now this top-level is assigning all the pins to `1'bz`.  This means the pins will be left floating or disconnected.  Let's implement some logic to blink a few LEDs.

Before we can do anything, we need a clock source.  The MachXO2 FPGAs have an internal oscillator we can use.  To use the internal oscillator we need to instantiate the `OSCH` module and tell it what frequency we want.

```verilog
  wire clk;
  
  OSCH #(
    .NOM_FREQ("2.08")
  ) internal_oscillator_inst (
    .STDBY(1'b0), 
    .OSC(clk)
  ); 
```

Since we are just blinking some LEDs we don't need a high frequency so we can use the lowest internal frequency available which is 2.08 MHz.  For more details on the clock resources available in the MachXO2 FPGAs please see the [MachXO2 sysCLOCK PLL Design and Usage Guide](http://www.latticesemi.com/~/media/LatticeSemi/Documents/ApplicationNotes/MO/MachXO2sysCLOCKPLLDesignandUsageGuide.pdf?document_id=39080).

_NOTE: If you are familiar with VHDL or Verilog you may decide to quickly skim through the rest of this step or skip it completely.  If you are not at all familiar with Verilog you should pay close attention and take a look at the additional resources at the end of this step._

If you're Now that we have a clock we can implement some sequential logic.  We will create a simple counter to time the blinking of our LEDs.

```verilog
  reg [23:0] led_timer;
  
  always @(posedge clk) begin
    led_timer <= led_timer + 1; 
  end
```

The timer will increment by 1 every clock period.  We can use the upper bits to blink our LEDs but we need to assign them to external pins.  Edit the corresponding `assign` statements so they match the code below.

```verilog
  assign pin9_jtgnb = led_timer[23];
  assign pin10_sda = led_timer[22];
  assign pin11_scl = led_timer[21];
```

At this point you should save all your changes by clicking the floppy disk icon below the menubar or by using the `CTRL + SHIFT + S` keyboard shortcut.  

#### 4. Generate JEDEC programming file

Select the `Process` tab on the left hand side view.  This will bring up a tree of tasks that need to be executed in order to generate the JEDEC programming file.  At the very bottom of the tree you should see an entry labeled `JEDEC File`.  Double-click that entry and the programming file should be created.  When it is done it will have a green checkmark next to it.

#### 5. P