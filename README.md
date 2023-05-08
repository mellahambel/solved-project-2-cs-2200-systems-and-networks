Download Link: https://assignmentchef.com/product/solved-project-2-cs-2200-systems-and-networks
<br>
<h1>1      Introduction</h1>

We have spent the last few weeks implementing our 32-bit datapath. The simple 32-bit LC-900 is capable of performing advanced computational tasks and logical decision making. Now it is time for us to move on to something more advanced—the upgraded LC-900a enables the ability for programs to be interrupted. Your assignment is to fully implement and test interrupts using the provided datapath and CircuitSim. You will hook up the interrupt and data lines to the new timer device, modify the datapath and microcontroller to support interrupt operations, and write an interrupt handler to operate this new device. You will also use the tiny, inexpensive LC-900a as an embedded system to monitor a kitchen appliance.

<h1>2      Requirements</h1>

Before you begin, please ensure you have done the following:

<ul>

 <li>Download the proper version of CircuitSim. A copy of CircuitSim is available under Files on Gradescope. You may also download it from the CircuitSim website (https://ra4king.github.io/CircuitSim/). In order to run CircuitSim, Java must be installed. If you are a Mac user, you may need to right-click on the JAR file and select “Open” in the menu to bypass Gatekeeper restrictions.</li>

 <li>CircuitSim is still under development and may have unknown bugs. Please back up your work using some form of version control, such as a local/private git repository or Dropbox. <strong>Do not use public git repositories; it is against the Georgia Tech Honor Code</strong>.</li>

 <li>The LC-900a assembler is written in Python. If you do not have Python 2.6 or newer installed on your system, you will need to install it before you continue.</li>

</ul>

<h1>3         What We Have Provided</h1>

<ul>

 <li>A reference guide to the LC-900a is located in <em>Appendix A: LC-900a Instruction Set Architecture</em>. <strong>Please read this first before you move on! </strong>The reference introduces several new instructions that you will implement for this project.</li>

 <li>A CircuitSim circuit (int-devices.sim) containing a timer device and rice cooker subcircuit that you will use for this project. <strong>You should copy and paste the contents of the new devices into subcircuits in your main circuit file.</strong></li>

 <li>A new microcode configuration spreadsheet xlsx with additional bits for the new signals that will be added in this project.</li>

 <li>A timer device that will generate an interupt signal at regular intervals. The pinout and functionality of this device are described in <em>Adding an External Timer Device</em>.</li>

 <li>A rice cooker that will generate an interrupt signal at regular intervals, and provides rice cooker power readings. The pinout and functionality of this device are described in <em>Adding a Rice Cooker</em>.</li>

 <li>An <em>incomplete </em>assembly program s that you will complete and use to test your interrupt capabilities.</li>

 <li>An assembler with support for the new instructions to assemble the test program.</li>

 <li>A completed LC-900 datapath circuit (LC-900.sim) from Project 1 is provided. You may use this as a base to add the basic interrupt support for the LC-900a or build off of your own Project 1 datapath, <strong>but you must rename the file to </strong>LC-900a.sim. Most of the work can be easily carried over from one datapath to another.</li>

 <li>A microcode file (xlsx) that meets the requirements of Project 1; however, feel free to supply your own.</li>

</ul>

<h1>4          Phase 1 – Implementing a Basic Interrupt</h1>

<h2>Figure 1: Basic Interrupt Hardware for the LC-900a Processor</h2>

For this assignment, you will add interrupt support to the LC-900a datapath. Then, you will test your new capabilities to handle interrupts using an external timer device.

<strong>Work in the LC-900a.sim file. </strong>If you wish to use your existing datapath, make a copy with this name, and add the devices we provided.

<h2>4.1      Interrupt Hardware Support</h2>

First, you will need to add the hardware support for interrupts.

<strong>You must do the following:</strong>

<ol>

 <li>Our processor needs a way to turn interrupts on and off. Create a new one-bit “Interrupt Enable” (IE) register. You’ll connect this register to your microcontroller in a later step.</li>

 <li>Create the INT line. The external device you will create in 4.2 will pull this line high (assert a ’1’) when they wish to interrupt the processor. Because multiple devices can share a single INT line, only one device can write to it at once. When a device does not have an interrupt, it neither pulls the line high nor low. You must accomodate this in your hardware by making sure that the final value going to the microcontroller always has a value (i.e. not a blue wire in CircuitSim). This can be done by using a specific gate to act like a pull-down resistor so that there is always a value asserted.</li>

 <li>When a device receives an <strong>IntAck </strong>signal, it will drive a 32-bit device ID onto the I/O Data Bus. To prevent devices from interfering with the processor, the I/O Data Bus is attached to the Main Bus with a tri-state driver. Create this driver and the bus, and attach the microcontroller’s <strong>DrDATA </strong>signal to the driver.</li>

 <li>Modify the datapath so that the PC starts at 0x08 when the processor is reset. Normally the PC starts at 0x00, however we need to make space for the interrupt vector table (IVT). Therefore, when you actually load in the test code that you will write, it needs to start at 0x08. Please make sure that your solution ensures that datapath can never execute from below 0x08 – or in other words, force the PC to drive the value 0x08 if the PC is pointing in the range of the vector table.</li>

 <li>Create hardware to support selecting the register $k0 within the microcode. This is needed by some interrupt related instructions. Because we need to access $k0 outside of regular instructions, we cannot use the Rx / Ry / Rz bits. <strong>HINT: </strong>Use only the register selection bits that the main ROM already outputs to select $ Notice that there is an unused input to the RegSel multiplexer.</li>

</ol>

<h2>4.2        Adding an External Timer Device</h2>

Hardware timers are an essential device in any CPU design. They allow the CPU to monitor the passing of various time intervals, without dedicating CPU instructions to the cause.

The ability of timers to raise interrupts also enables preemptive multitasking, where the operating system periodically interrupts a running process to let another process take a turn. Timers are also essential to ensuring a single misbehaving program cannot freeze up your entire computer.

You will connect an external timer device to the datapath. It is internally configured to have a <strong>device ID of 0x0 </strong>and <strong>interrupt every 2000 clock ticks</strong>.

The pinout of the timer device is described below. If you like, you may also examine the internals of the device in CircuitSim.

<ul>

 <li><strong>CLK</strong>: The clock input to the device. Make sure you connect this to the same clock as the rest of your circuit.</li>

 <li><strong>INT</strong>: The device will begin to assert this line when its time interval has elapsed. It will not be lowered until the cycle after it receives an INTA signal.</li>

 <li><strong>INTA IN</strong>: When the INTA IN line is asserted while the device has asserted the INT line, it will drive its device ID to the DATA line and lower its INT line <strong>on the next clock cycle</strong>.</li>

 <li><strong>INTA OUT</strong>: When the INTA IN line is asserted while the device does not have an interrupt pending, its value will be propagated to INTA OUT. This allows for daisy chaining of devices.</li>

 <li><strong>DATA</strong>: The device will drive its ID (0x0) to this line after receiving an INTA.</li>

</ul>

The INT and DATA lines from the timer should be connected to the appropriate buses that you added in the previous section.

<h2>4.3       Microcontroller Interrupt Support</h2>

<strong>Before beginning this part, be sure you have read through </strong><em>Appendix A: LC-900a Instruction Set Architecture </em><strong>and </strong><em>Appendix B: Microcontrol Unit </em><strong>and pay special attention to the new instructions. However, for this part of the project, you do not need to worry about the LdDAR signal or the IN instruction.</strong>

In this part of the assignment you will modify the microcontroller and the microcode of the LC-900a to support interrupts. You will need to do the following:

<ol>

 <li>Be sure to read the appendix on the microcontroller before starting this section.</li>

 <li>Modify the microcontroller to support asserting four new signals:

  <ul>

   <li><strong>LdEnInt </strong>&amp; <strong>EnInt </strong>to control whether interrupts are enabled/disabled. You will use these 2 signals to control the value of your interrupts enabled register.</li>

   <li><strong>IntAck </strong>to send an interrupt acknowledge to the device.</li>

   <li><strong>DrDATA </strong>to drive the value on the I/O Data Bus to the Main Bus.</li>

  </ul></li>

 <li>Extend the size of the ROM accordingly.</li>

 <li>Add the fourth ROM described in <em>Appendix B: Microcontrol Unit </em>to handle onInt.</li>

 <li>Modify the FETCH macrostate microcode so that we actively check for interrupts. Normally this is done within the INT macrostate (as described in Chapter 4 of the book and in the lectures) but we are rolling this functionality in the FETCH macrostate for the sake of simplicity. You can accomplish this by doing the following:

  <ul>

   <li>First check to see if the CPU should be interrupted. To be interrupted, two conditions must be true: (1) interrupts are enabled (i.e., the IE register must hold a ’1’), and (2), a device must be asserting an interrupt.</li>

   <li>If not, continue with FETCH normally.</li>

   <li>If the CPU should be interrupted, then perform the following:

    <ol>

     <li>Save the current PC to the register $</li>

     <li>Disable interrupts.</li>

    </ol></li>

  </ul></li>

</ol>

<ul>

 <li>Assert the interrupt acknowledge signal (IntAck). Next, drive the device ID from the I/O Data Bus and use it to index into the interrupt vector table to retrieve the new PC value. The device will drive its device ID onto the I/O Data Bus one clock cycle <strong>after </strong>it receives the IntAck signal.</li>

</ul>

<ol>

 <li>This new PC value should then be loaded into the PC.</li>

</ol>

<strong>Note: </strong>onInt <strong>works in the same manner that CmpOut did in Project 1. The processor should branch to the appropriate microstate depending on the value of </strong>onInt<strong>. </strong>onInt <strong>should be true when interrupts are enabled AND when there is an interrupt to be acknowledged. Note: The mode bit mechanism and user/kernel stack separation discussed in the textbook has been omitted for simplicity.</strong>

<ol start="6">

 <li>Implement the microcode for three new instructions for supporting interrupts as described in Chapter 4. These are the EI, DI, and RETI instructions. You need to write the microcode in the main ROM controlling the datapath for these three new instructions. Keep in mind that:

  <ul>

   <li>EI sets the IE register to 1.</li>

   <li>DI sets the IE register to 0.</li>

   <li>RETI loads $k0 into the PC, and enables interrupts.</li>

  </ul></li>

</ol>

<h2>4.4       Implementing the Timer Interrupt Handler</h2>

Our datapath and microcontroller now fully support interrupts from devices, BUT we must now implement the interrupt handler t1_handler within the prj2.s file to support interrupts from the timer device while also not interfering with the correct operation of any user programs.

In prj2.s, we provide you with a modified version of pow.s that will run while you are waiting for interrupts. For this part of the project, you need to write interrupt handler for the timer device (device ID 0x0). You should refer to Chapter 4 of the textbook to see how to write a correct interrupt handler. As detailed in that chapter, your handler will need to do the following:

<ol>

 <li>First save the current value of $k0 (the return address to where you came from to the current handler) 2. Enable interrupts (which should have been disabled implicitly by the processor within the INT macrostate).</li>

</ol>

<ol start="3">

 <li>Save the state of the interrupted program.</li>

 <li>Implement the actual work to be done in the handler. In the case of this project, we want you to <strong>increment a counter variable in memory</strong>, which we have already provided.</li>

 <li>Restore the state of the original program and return using RETI.</li>

</ol>

The handler you have written for the timer device should run every time the device’s interrupt is triggered. Make sure to write the handler such that interrupts can be nested. With that in mind, interrupts should be enabled for <strong>as long as possible </strong>within the handlers.

You will need to do the following:

<ol>

 <li>Write the interrupt handler (should follow the above instructions or simply refer to Chapter 4 in your book). In the case of this project, we want the interrupt handler to keep time in memory at the predetermined location: 0xFFFF</li>

 <li>Load the starting address of the first handler you just implemented in s into the interrupt vector table at the appropriate addresses (the table is indexed using the device ID of the interrupting device).</li>

</ol>

<strong>Test your design before moving onto the next section. </strong>If it works correctly, you should see the value at address 0xFFFF in memory increment as the program runs.

<h1>5           Phase 2 – Implementing Interrupts from Input Devices</h1>

<h2>Figure 2: Interrupt Hardware for the LC-900a with Basic I/O Support</h2>

Eager to put your newfound knowledge of device interrupts from CS2200 to good use, you decide to apply what you’ve learned to your second favorite hobby (besides CS2200): cooking. You use a rice cooker often (as a college student does), but are concerned with how much power your rice cooker uses to keep the rice warm, especially when you forget to take the rice out of the rice cooker.

You’ve rigged up a device that is able to report the power consumption of your rice cooker to an LC-900a processor via an interrupt. There’s only one issue: as of right now, your datapath can detect when an external device is ready to interrupt the processor, but it cannot retrieve data from external devices.

In this phase of the project, you will add functionality for device-addressed input. You will then make use of this functionality by adding a device simulating a rice cooker and writing a simple handler for the device.

<h2>5.1       Basic I/O Support</h2>

Before adding the rice cooker, you will first need to add support for device addressed I/O. In order to get input from a device such as a rice cooker, you will write a value to an Address Bus, which instructs the device with that address (which in this case is the same as the device ID) to write its output data to the I/O Data Bus.

<strong>You must do the following:</strong>

<ol>

 <li>Create the device address register (DAR) and connect its enable to the LdDAR signal from your microcontroller. This register gets its input from the Main Bus, and its output will be directly connected to the Address Bus. It will allow us to send assert a value on the Address Bus while using the Main Bus for other operations.</li>

 <li>Modify the microcontroller to support a new control signal, <strong>LdDAR</strong>. This signal will be used in order to enable writing to the DAR.</li>

 <li>Implement the IN instruction in your microcode. This instruction takes a device address from the immediate, loads it into the DAR, and writes the value on the data bus into a register. When it is done, it <strong>must clear the DAR to zero </strong>(since interrupts use the data bus to communicate device IDs). Examine the format of the IN instruction and consider what signals you might raise in order to write a constant zero into the DAR.</li>

</ol>

<h2>5.2       Adding a Rice Cooker</h2>

You will connect a rice cooker to your datapath that simulates a real rice cooker by returning the power consumption of the cooker. Its internals are similar to the timer device, meaning it asserts interrupts and handles acknowledgements in the same way. Every 1000 cycles, it will assert an interrupt signaling that an power value has been captured. This power reading can be fetched as a 32-bit word by writing the device’s address to the ADDR line.

The rice cooker is internally configured to have a <strong>device ID of 0x1</strong>.

Place the rice cooker in your datapath circuit. This device will share the INT and DATA lines with the timer you added previously. However, it should receive its INTA signal from the INTA OUT pin on the timer device. This ensures that if both the timer and rice cooker raise an interrupt at the same time, the timer will be acknowledged first, and the rice cooker will be acknowledged after. <strong>This is known as “daisy chaining” devices.</strong>

<h2>5.3        Implementing the Rice Cooker Interrupt Handler</h2>

Now that your LC-900a datapath can accept data from your rice cooker, we need to decide what to do with the data. After cooking rice, it takes a lot of power to keep rice warm (especially if you leave it in there for a while), so you will be calculating the <em>total </em>amount of power the rice cooker expends to keep the rice warm <strong>after </strong>cooking it. When the rice is cooking, it will use a huge amount of power (over 500W), but it needs <strong>less than 50W </strong>to keep rice warm. You’ll have to implement this logic in your handler, which will work similarly to the one you wrote for the timer device. However, instead of incrementing a timer at a memory location, <strong>you will be keeping track of the total power expended to keep rice warm in a memory location. </strong>This means you will need to keep adding the new values you get to the sum of previous values.

In addition to the usual overhead of an interrupt handler, your rice cooker handler must do the following:

<ol>

 <li>Use the IN instruction to obtain the most recently captured power value from the cooker.</li>

 <li>Add the value obtained from the cooker to the memory location with address 0xFFE0 <strong>only if the power value is less than 50W</strong>.</li>

</ol>

<strong>Make sure that you properly install the location of the new handler into the IVT.</strong>

The rice cooker hardware is designed to emit a sequence of numbers representing power consumption. If your design is working properly, you should see the value stored in the memory location increase linearly after a few thousand clock cycles as it updates when a new power value is pushed onto the datapath.

To validate you’re updating the power expended correctly, you can check the values that the rice cooker will emit by inspecting the internals of the circuit and checking the values in the ROM labeled ’Power Buffer’.

<h1>6      Deliverables</h1>

Please submit all of the following files in a <strong>.tar.gz </strong>archive generated by one of the following:

<ul>

 <li><strong>On Linux: </strong>Use the provided Makefile. The Makefile will work on any Unix or Linux-based machine (on Ubuntu, you may need to sudo apt-get install build-essential if you have never installed the build tools). Run make submit to automatically package your project into the correct archive format.</li>

 <li><strong>On Windows: </strong>Use the provided submit.bat script. Submitting through this method will require 7zip (https://www.7-zip.org/) to be installed on your system. Run bat to automatically package your project into the correct archive format.</li>

 <li><strong>On Mac: </strong>Use the provided Makefile. If you haven’t yet installed Command Line Tools, you’ll need to do so. If you have Xcode installed on your machine, you already have Command Line Tools. Otherwise, you can install them by either installing Xcode or installing Command Line Tools standalone from Apple’s developer site. Run make submit to automatically package your project into the correct archive format.</li>

</ul>

The generated archive should contain at a minimum the following files:

<ul>

 <li>CircuitSim datapath file (LC-900a.sim)</li>

 <li>Microcode file (microcode.xlsx)</li>

 <li>Assembly code (prj2.s)</li>

</ul>

<strong>Always re-download your assignment from Gradescope after submitting to ensure that all necessary files were properly uploaded. If what we download does not work, you will get a 0 regardless of what is on your machine.</strong>

<strong>This project will be demoed</strong>. In order to receive full credit, you must sign up for a demo slot and complete the demo. We will announce when demo times are released.

<h1>7            Appendix A: LC-900a Instruction Set Architecture</h1>

The LC-900a is a simple, yet capable computer architecture. The LC-900a combines attributes of both ARM and the LC-2200 ISA defined in the Ramachandran &amp; Leahy textbook for CS 2200.

The LC-900a is a <strong>word-addressable</strong>, <strong>32-bit </strong>computer. <strong>All addresses refer to words</strong>, i.e. the first word (four bytes) in memory occupies address 0x0, the second word, 0x1, etc.

All memory addresses are truncated to 16 bits on access, discarding the 16 most significant bits if the address was stored in a 32-bit register. This provides roughly 64 KB of addressable memory.

<h2>7.1     Registers</h2>

The LC-900a has 16 general-purpose registers. While there are no hardware-enforced restraints on the uses of these registers, your code is expected to follow the conventions outlined below.

<h3>Table 1: Registers and their Uses</h3>

<table width="426">

 <tbody>

  <tr>

   <td width="115">Register Number</td>

   <td width="50">Name</td>

   <td width="174">Use</td>

   <td width="88">Callee Save?</td>

  </tr>

  <tr>

   <td width="115">0</td>

   <td width="50">$zero</td>

   <td width="174">Always Zero</td>

   <td width="88">NA</td>

  </tr>

  <tr>

   <td width="115">1</td>

   <td width="50">$at</td>

   <td width="174">Assembler/Target Address</td>

   <td width="88">NA</td>

  </tr>

  <tr>

   <td width="115">2</td>

   <td width="50">$v0</td>

   <td width="174">Return Value</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">3</td>

   <td width="50">$a0</td>

   <td width="174">Argument 1</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">4</td>

   <td width="50">$a1</td>

   <td width="174">Argument 2</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">5</td>

   <td width="50">$a2</td>

   <td width="174">Argument 3</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">6</td>

   <td width="50">$t0</td>

   <td width="174">Temporary Variable</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">7</td>

   <td width="50">$t1</td>

   <td width="174">Temporary Variable</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">8</td>

   <td width="50">$t2</td>

   <td width="174">Temporary Variable</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">9</td>

   <td width="50">$s0</td>

   <td width="174">Saved Register</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">10</td>

   <td width="50">$s1</td>

   <td width="174">Saved Register</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">11</td>

   <td width="50">$s2</td>

   <td width="174">Saved Register</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">12</td>

   <td width="50">$k0</td>

   <td width="174">Reserved for OS and Traps</td>

   <td width="88">NA</td>

  </tr>

  <tr>

   <td width="115">13</td>

   <td width="50">$sp</td>

   <td width="174">Stack Pointer</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">14</td>

   <td width="50">$fp</td>

   <td width="174">Frame Pointer</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">15</td>

   <td width="50">$ra</td>

   <td width="174">Return Address</td>

   <td width="88">No</td>

  </tr>

 </tbody>

</table>

<ol>

 <li><strong>Register 0 </strong>is always read as zero. Any values written to it are discarded. <strong>Note: </strong>for the purposes of this project, you must implement the zero register. Regardless of what is written to this register, it should always output zero.</li>

 <li><strong>Register 1 </strong>is used to hold the target address of a jump. It may also be used by pseudo-instructions generated by the assembler.</li>

 <li><strong>Register 2 </strong>is where you should store any returned value from a subroutine call.</li>

 <li><strong>Registers 3 – 5 </strong>are used to store function/subroutine arguments. <strong>Note: </strong>registers 2 through 8 should be placed on the stack if the caller wants to retain those values. These registers are fair game for the callee (subroutine) to trash.</li>

 <li><strong>Registers 6 – 8 </strong>are designated for temporary variables. The caller must save these registers if they want these values to be retained.</li>

 <li><strong>Registers 9 – 11 </strong>are saved registers. The caller may assume that these registers are never tampered with by the subroutine. If the subroutine needs these registers, then it should place them on the stack and restore them before they jump back to the caller.</li>

 <li><strong>Register 12 </strong>is reserved for handling interrupts. While it should be implemented, it otherwise will not have any special use on this assignment.</li>

 <li><strong>Register 13 </strong>is your anchor on the stack. It keeps track of the top of the activation record for a subroutine.</li>

 <li><strong>Register 14 </strong>is used to point to the first address on the activation record for the currently executing process.</li>

 <li><strong>Register 15 </strong>is used to store the address a subroutine should return to when it is finished executing.</li>

</ol>

<h2>7.2      Instruction Overview</h2>

The LC-900a supports a variety of instruction forms, only a few of which we will use for this project. The instructions we will implement in this project are summarized below.

<h3>Table 2: LC-900a Instruction Set</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0000</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td colspan="2" width="47">SR2</td>

  </tr>

  <tr>

   <td width="47">0001</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td colspan="2" width="47">SR2</td>

  </tr>

  <tr>

   <td width="47">0010</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">immval20</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">0011</td>

   <td width="47">DR</td>

   <td width="47">BaseR</td>

   <td width="187">offset20</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">0100</td>

   <td width="47">SR</td>

   <td width="47">BaseR</td>

   <td width="187">offset20</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">0101</td>

   <td colspan="2" width="94">unused</td>

   <td width="187">offset20</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">0110</td>

   <td width="47">RA</td>

   <td width="47">AT</td>

   <td width="187">unused</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">0111</td>

   <td colspan="2" width="94"></td>

   <td width="187">unused</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="187">unused</td>

   <td width="12"></td>

   <td width="35">001</td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="187">unused</td>

   <td width="12"></td>

   <td width="35">011</td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="187">unused</td>

   <td width="12"></td>

   <td width="35">010</td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="187">unused</td>

   <td width="12"></td>

   <td width="35">101</td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="187">unused</td>

   <td width="12"></td>

   <td width="35">100</td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="187">unused</td>

   <td width="12"></td>

   <td width="35">110</td>

  </tr>

  <tr>

   <td width="47">1001</td>

   <td width="47">DR</td>

   <td width="47">unused</td>

   <td width="187">PCoffset20</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">1010</td>

   <td colspan="2" width="94"></td>

   <td width="187">unused</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">1011</td>

   <td colspan="2" width="94"></td>

   <td width="187">unused</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">1100</td>

   <td colspan="2" width="94"></td>

   <td width="187">unused</td>

   <td colspan="2" width="47"></td>

  </tr>

  <tr>

   <td width="47">1101</td>

   <td width="47">DR</td>

   <td width="47">0000</td>

   <td width="187">addr20</td>

   <td colspan="2" width="47"></td>

  </tr>

 </tbody>

</table>

ADD

NAND

ADDI

LW

SW

BR

JALR

HALT

SKPLT

SKPLE

SKPEQ

SKPNE

SKPGT

SKPGE

LEA

EI

DI

RETI

IN

<h3>7.2.1        Conditional Branching</h3>

Branching in the LC-900a ISA is a bit different than usual. We have one branch instruction: the BR instruction, which, if executed, unconditionally jumps/changes the PC. Then, we have a series of instructions known as the skip-instructions, or SKP instructions. These instructions use the comparison operators, comparing the values of two source registers. If the comparisons are true (for example, with the SKPGT instruction, if SR1 <em>&gt; </em>SR2), then we skip over the next line of code – we increment PC by 1 (remember that at the time of execution, PC has already been incremented by 1, so this is an additional increment).

These SKP instructions all have the same opcode and use bits 2:0 to determine the comparison type (CmpSel). Bit 0 controls less then, bit 1 controls equals, and bit 2 controls greater than. For example, if you wanted to do a SKPGE (skip greater than or equal), you would set the CmpSel bits as: 110 (high for greater than and high for equals.)

Branching can then be accomplished by using the SKP instructions to do comparison-checking, and using BR instructions below those SKP instructions. Particular BR instructions are taken if a particular comparison result is reached.

<h2>7.3       Detailed Instruction Reference</h2>

<strong>7.3.1       ADD</strong>

<strong>Assembler Syntax</strong>

ADD           DR, SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0000</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = SR1 + SR2;

<h3>Description</h3>

The ADD instruction obtains the first source operand from the SR1 register. The second source operand is obtained from the SR2 register. The second operand is added to the first source operand, and the result is stored in DR.

<strong>7.3.2        NAND</strong>

<strong>Assembler Syntax</strong>

NAND        DR, SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0001</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = ~(SR1 &amp; SR2);

<h3>Description</h3>

The NAND instruction performs a logical NAND (AND NOT) on the source operands obtained from SR1 and SR2. The result is stored in DR.

<table width="623">

 <tbody>

  <tr>

   <td width="623"><strong>HINT: </strong>A logical NOT can be achieved by performing a NAND with both source operands the same.For instance,NAND DR, SR1, SR1…achieves the following logical operation: <em>DR</em>←<em>SR</em>1.</td>

  </tr>

 </tbody>

</table>

<strong>7.3.3       ADDI</strong>

<strong>Assembler Syntax</strong>

ADDI           DR, SR1, immval20

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0010</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="234">immval20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = SR1 + SEXT(immval20);

<h3>Description</h3>

The ADDI instruction obtains the first source operand from the SR1 register. The second source operand is obtained by sign-extending the immval20 field to 32 bits. The resulting operand is added to the first source operand, and the result is stored in DR.

<strong>7.3.4      LW</strong>

<strong>Assembler Syntax</strong>

LW             DR, offset20(BaseR)

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0011</td>

   <td width="47">DR</td>

   <td width="47">BaseR</td>

   <td width="234">offset20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = MEM[BaseR + SEXT(offset20)];

<h3>Description</h3>

An address is computed by sign-extending bits [19:0] to 32 bits and then adding this result to the contents of the register specified by bits [23:20]. The 32-bit word at this address is loaded into DR.

<strong>7.3.5       SW</strong>

<strong>Assembler Syntax</strong>

SW             SR, offset20(BaseR)

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0100</td>

   <td width="47">SR</td>

   <td width="47">BaseR</td>

   <td width="234">offset20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

MEM[BaseR + SEXT(offset20)] = SR;

<h3>Description</h3>

An address is computed by sign-extending bits [19:0] to 32 bits and then adding this result to the contents of the register specified by bits [23:20]. The 32-bit word obtained from register SR is then stored at this address.

<strong>7.3.6       BR</strong>

<strong>Assembler Syntax</strong>

BR           offset20

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0101</td>

   <td width="94">unused</td>

   <td width="234">offset20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

PC = incrementedPC + offset20

<h3>Description</h3>

A branch is unconditionally taken. The PC will be set to the sum of the incremented PC (since we have already undergone fetch) and the sign-extended offset[19:0].

<strong>7.3.7        JALR</strong>

<strong>Assembler Syntax</strong>

JALR          RA, AT

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0110</td>

   <td width="47">RA</td>

   <td width="47">AT</td>

   <td width="234">unused</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

RA = PC;

PC = AT;

<h3>Description</h3>

First, the incremented PC (address of the instruction + 1) is stored into register RA. Next, the PC is loaded with the value of register AT, and the computer resumes execution at the new PC.

<strong>7.3.8        HALT</strong>

<strong>Assembler Syntax</strong>

HALT

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0111</td>

   <td width="328">unused</td>

  </tr>

 </tbody>

</table>

<strong>Description</strong>

The machine is brought to a halt and executes no further instructions.

<strong>7.3.9        SKPLT</strong>

<strong>Assembler Syntax</strong>

SKPLT SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="199">unused</td>

   <td width="35">001</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

if (SR1 &lt; SR2) {

PC = incrementedPC + 1

}

<strong>Description</strong>

The incrementedPC is further incremented by 1 if SR1 is less than SR2.

<strong>7.3.10        SKPLE</strong>

<strong>Assembler Syntax</strong>

SKPLE SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="199">unused</td>

   <td width="35">011</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

if (SR1 &lt;= SR2) {

PC = incrementedPC + 1

}

<strong>Description</strong>

The incrementedPC is further incremented by 1 if SR1 is less than or equal to SR2.

<strong>7.3.11        SKPEQ</strong>

<strong>Assembler Syntax</strong>

SKPEQ SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="199">unused</td>

   <td width="35">010</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

if (SR1 == SR2) {

PC = incrementedPC + 1

}

<strong>Description</strong>

The incrementedPC is further incremented by 1 if SR1 is equal to SR2.

<strong>7.3.12        SKPNE</strong>

<strong>Assembler Syntax</strong>

SKPNE SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="199">unused</td>

   <td width="35">101</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

if (SR1 != SR2) {

PC = incrementedPC + 1

}

<strong>Description</strong>

The incrementedPC is further incremented by 1 if SR1 is not equal to SR2.

<strong>7.3.13        SKPGT</strong>

<strong>Assembler Syntax</strong>

SKPGT SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="199">unused</td>

   <td width="35">100</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

if (SR1 &gt; SR2) {

PC = incrementedPC + 1

}

<strong>Description</strong>

The incrementedPC is further incremented by 1 if SR1 is greater than SR2.

<strong>7.3.14        SKPGE</strong>

<strong>Assembler Syntax</strong>

SKPGE SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">SR1</td>

   <td width="47">SR2</td>

   <td width="199">unused</td>

   <td width="35">110</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

if (SR1 &gt;= SR2) {

PC = incrementedPC + 1

}

<strong>Description</strong>

The incrementedPC is further incremented by 1 if SR1 is greater or equal to SR2.

<strong>7.3.15       LEA</strong>

<strong>Assembler Syntax</strong>

LEA            DR, label

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1001</td>

   <td width="47">DR</td>

   <td width="47">unused</td>

   <td width="234">PCoffset20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = PC + SEXT(PCoffset20);

<h3>Description</h3>

An address is computed by sign-extending bits [19:0] to 32 bits and adding this result to the incremented PC (address of instruction + 1). It then stores the computed address into register DR.

<strong>7.3.16      EI</strong>

<strong>Assembler Syntax</strong>

EI

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1010</td>

   <td width="328">unused</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

IE = 1;

<strong>Description</strong>

The Interrupts Enabled register is set to 1, enabling interrupts.

<strong>7.3.17      DI</strong>

<strong>Assembler Syntax</strong>

DI

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1011</td>

   <td width="328">unused</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

IE = 0;

<strong>Description</strong>

The Interrupts Enabled register is set to 0, disabling interrupts.

<strong>7.3.18       RETI</strong>

<strong>Assembler Syntax</strong>

RETI

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1100</td>

   <td width="328">unused</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

PC = $k0;

IE = 1;

<h3>Description</h3>

The PC is restored to the return address stored in $k0. The Interrupts Enabled register is set to 1, enabling interrupts.

<strong>7.3.19      IN</strong>

<strong>Assembler Syntax</strong>

IN DR, DeviceADDR

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1101</td>

   <td width="47">DR</td>

   <td width="47">0000</td>

   <td width="234">addr20</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

DAR = addr20;

DR = DeviceData;

DAR = 0;

<h3>Description</h3>

The value in addr20 is sign-extended to determine the 32-bit device address. This address is then loaded into the Device Address Register (DAR). The processor then reads a single word value off the device data bus, and writes this value to the DR register. The DAR is then reset to zero, ending the device bus cycle.

<h1>8         Appendix B: Microcontrol Unit</h1>

As you may have noticed, we currently have an unused input on our multiplexer. This gives us room to add another ROM to control the next microstate upon an interrupt. You need to use this fourth ROM to generate the microstate address when an interrupt is signaled. The input to this ROM will be controlled by your interrupt enabled register and the interrupt signal asserted by the timer interrupt. This fourth ROM should have a 1-bit input and 6-bit output.

<h2>Figure 3: Three ROM Microcontrol Unit</h2>

The outputs of the FSM control which signals on the datapath are raised (asserted). Here is more detail about the meaning of the output bits for the microcontroller:

<h2>Table 3: ROM Output Signals</h2>

<table width="536">

 <tbody>

  <tr>

   <td width="34">Bit</td>

   <td width="90">Purpose</td>

   <td width="36">Bit</td>

   <td width="66">Purpose</td>

   <td width="36">Bit</td>

   <td width="65">Purpose</td>

   <td width="36">Bit</td>

   <td width="72">Purpose</td>

   <td width="36">Bit</td>

   <td width="68">Purpose</td>

  </tr>

  <tr>

   <td width="34">0</td>

   <td width="90">NextState[0]</td>

   <td width="36">6</td>

   <td width="66">DrREG</td>

   <td width="36">12</td>

   <td width="65">LdIR</td>

   <td width="36">18</td>

   <td width="72">WrMEM</td>

   <td width="36">24</td>

   <td width="68">ChkCmp</td>

  </tr>

  <tr>

   <td width="34">1</td>

   <td width="90">NextState[1]</td>

   <td width="36">7</td>

   <td width="66">DrMEM</td>

   <td width="36">13</td>

   <td width="65">LdMAR</td>

   <td width="36">19</td>

   <td width="72">RegSelLo</td>

   <td width="36">25</td>

   <td width="68">LdEnInt</td>

  </tr>

  <tr>

   <td width="34">2</td>

   <td width="90">NextState[2]</td>

   <td width="36">8</td>

   <td width="66">DrALU</td>

   <td width="36">14</td>

   <td width="65">LdA</td>

   <td width="36">20</td>

   <td width="72">RegSelHi</td>

   <td width="36">26</td>

   <td width="68">EnInt</td>

  </tr>

  <tr>

   <td width="34">3</td>

   <td width="90">NextState[3]</td>

   <td width="36">9</td>

   <td width="66">DrPC</td>

   <td width="36">15</td>

   <td width="65">LdB</td>

   <td width="36">21</td>

   <td width="72">ALULo</td>

   <td width="36">27</td>

   <td width="68">IntAck</td>

  </tr>

  <tr>

   <td width="34">4</td>

   <td width="90">NextState[4]</td>

   <td width="36">10</td>

   <td width="66">DrOFF</td>

   <td width="36">16</td>

   <td width="65">LdCmp</td>

   <td width="36">22</td>

   <td width="72">ALUHi</td>

   <td width="36">28</td>

   <td width="68">DrData</td>

  </tr>

  <tr>

   <td width="34">5</td>

   <td width="90">NextState[5]</td>

   <td width="36">11</td>

   <td width="66">LdPC</td>

   <td width="36">17</td>

   <td width="65">WrREG</td>

   <td width="36">23</td>

   <td width="72">OPTest</td>

   <td width="36">29</td>

   <td width="68">LdDAR</td>

  </tr>

 </tbody>

</table>

Table 4: Register Selection Map

<table width="242">

 <tbody>

  <tr>

   <td width="69">RegSelHi</td>

   <td width="70">RegSelLo</td>

   <td width="103">Register</td>

  </tr>

  <tr>

   <td width="69">0</td>

   <td width="70">0</td>

   <td width="103">RX (IR[27:24])</td>

  </tr>

  <tr>

   <td width="69">0</td>

   <td width="70">1</td>

   <td width="103">RY (IR[23:20])</td>

  </tr>

  <tr>

   <td width="69">1</td>

   <td width="70">0</td>

   <td width="103">RZ (IR[3:0])</td>

  </tr>

  <tr>

   <td width="69">1</td>

   <td width="70">1</td>

   <td width="103">$k0 (1100)</td>

  </tr>

 </tbody>

</table>

Table 5: ALU Function Map

<table width="188">

 <tbody>

  <tr>

   <td width="58">ALUHi</td>

   <td width="63">ALUlLo</td>

   <td width="67">Function</td>

  </tr>

  <tr>

   <td width="58">0</td>

   <td width="63">0</td>

   <td width="67">ADD</td>

  </tr>

  <tr>

   <td width="58">0</td>

   <td width="63">1</td>

   <td width="67">SUB</td>

  </tr>

  <tr>

   <td width="58">1</td>

   <td width="63">0</td>

   <td width="67">NAND</td>

  </tr>

  <tr>

   <td width="58">1</td>

   <td width="63">1</td>

   <td width="67">A + 1</td>

  </tr>

 </tbody>

</table>