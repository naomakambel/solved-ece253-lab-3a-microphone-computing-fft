Download Link: https://assignmentchef.com/product/solved-ece253-lab-3a-microphone-computing-fft
<br>
Lab 3 makes use of a custom IP Core enabling the use of the Nexys4 DDR omnidirectional MEMS microphone. The goal of Lab3A and Lab3B is to build a Chromatic Tuner, which is a tool used by musicians to properly tune their instruments. First, the musician selects an octave (frequency range). Next the musician begins to play the instrument at a frequency, for example, 440 Hz is A4, or the note A in the 4th octave. The Chromatic Tuner responds by displaying the closest note and how far the tone of the instrument is off from the note. To do this, we need to implement an efficient way of computing FFT to determine the frequency of the note being played. <strong>This is the objective of Lab3A. For Lab3A, you need to demonstrate your FFT implementation by testing for input frequencies between 200 Hz and 5-10 kHz and displaying the note and its frequency on the terminal. The GUI and implementation of Chromatic Tuner will be completed in Lab 3B.</strong>

The default software application provided to you samples the microphone input using a custom stream grabber port into a Microblaze buffer and displays the base frequency on an equal-tempered diatonic scale (A=440 Hz). However, you will discover that the default FFT implementation is highly inefficient. You will need to improve the FFT code so as to accurately determine the input frequency over a large frequency range (50 Hz to 15 kHz) with a quick response time.

<h1>1       Block Design</h1>

While the focus of Lab3A is on efficient FFT computation, it is a good idea to add all the hardware peripherals you will need in Lab3B now itself. You can do the software configuration of the peripherals (not being used in Lab3A) in Lab3B. You should start from your Lab 2B project.

<strong>1.1      Rotary Encoder</strong>

Make sure the Vivado project is properly configured for the rotary encoder, just like it was in Lab 2B.

<strong>1.2       LCD Display</strong>

Make sure Vivado project has a properly configured interface for the LCD.

<h2>1.3      Customizing Microblaze</h2>

To build the Chromatic Tuner it is useful to further customize the Microblaze processor to increase arithmetic performance. Screenshots of a recommended configuration for the Chromatic Tuner are included at the back of the Lab Write Up. Right click on your Microblaze and copy the suggested configuration.

<h2>1.4     Microphone</h2>

The Microphone integrated circuit (IC) consists of two inputs and one output. The output signal is a Pulse Density Modulated single bit stream <strong>mic data </strong>which transmits at the rate of the input <strong>mic clk</strong>. A <em>Hierarchy </em>is created in the Vivado Block Design to accommodate the custom IP (Intellectual Property) Cores used to interface to the Analog Device ADMP241 chip.

The onboard microphone: The microphone uses an Analog Device ADMP421 chip which has a high signal to noise ratio (SNR) of 61dBA and high sensitivity of -26 dBFS. It also has a flat frequency response ranging from 100 Hz to 15 kHz. The digitized audio is output in the pulse density modulated (PDM) bit stream.

First a filter/decimator block (in nopll mic block) filters the incoming 3.125Mb/s PDM bit stream down to a 48 kHz 27-bit integer sampled data stream. The specific form of PDM is 4th order Sigma-Delta. The output sample rate, more precisely, is 48.828125 kHz or 100<em>MHz/</em>32<em>/</em>64. This module also generates the clock to drive the microphone.

Next we must get this stream into the processor. For this a stream grabber block is used. This accepts input samples from the filter block, but discards them by default. When triggered by the processor, it grabs a contiguous sequence of samples into its internal buffer. It always fills the buffer (up to 4096 samples). The processor can poll the device to see if enough samples have been received. When re-triggered it will start grabbing a new block of samples.

Figure 1: The microphone block consists of a microphone control block that outputs a continuous stream of data, and a stream grabber that, when commanded by the processor, grabs a section into its buffer to be read back into the processor buffer. <strong>IMPORTANT</strong>: Make a list of the module interconnections and hand it in with your lab report. If these connections are completed incorrectly and you managed to generate and export a bitstream, you may see the error in SDK “<strong>Unable to Stop Processor</strong>”. Power off your board, close xSDK, fix your Vivado Design, and regenerate the bit-stream.

Figure 2: The Nexys4 onboard microphone requires the connection of 3 pins. mic clk, mic data and mic lr sel. Make these 3 pins external, by right clicking the ports and selecting ‘Make External’. Edit the Nexys4 master DDR.xdc file to connect to the pins.

<strong>1.4.1           Creating the Microphone Hierarchy Block</strong>

<ol>

 <li>Add the provided nopll mic block.v and stream grabber.v to your project using the “Add sources” in the file menu.</li>

 <li>Add the nopll mic block to your design, and also a stream grabber. To add a block from Verilog source open the block diagram, and in the sources window (on the left) right click on the source file and choose “Add to block diagram”.</li>

 <li>Choose the tool tip “Select Area”. Completely select both blocks without wiring.</li>

 <li>Right click and choose ‘Create Hierarchy’. Name this sub-diagram mic block.</li>

 <li>Connect the AXI stream interface between the two modules: from m axis on the nopll mic blk to the s axis stream on the stream grabber.</li>

 <li>The option to “Run Connection Automation” should appear as a banner a the top of the diagram. option should appear.</li>

</ol>

Use this to connect s axi cpu from the stream grabber to the interconnect and to connect sys clk to the processor clock (ui clk from the MIG). <strong>Ensure this clock is set right, the wizard might choose the wrong default.</strong>

This should also make sys rst connect to the peripheral aresetn of the processor reset, same as your other peripherals.

Ensure that this is the case.

<ol start="7">

 <li>Make the mic data in, mic clk and mic lr sel ports external.</li>

 <li>Add the constraints given in mic constraints.xdc to your main XDC file to hook up the microphone pins externally.</li>

 <li>If the pin names from “Make External” end up with a trailing “ 0” remove this so the names agree with the constraints.</li>

 <li>Go to the Address Editor tab (If this tab doesn’t automatically appear, go to Window then Address Editor) and ensure mic block/stream grabber 0 has been assigned an address. If not it will show up under an “Unmapped Slave” sub-category. If so, right-click on it and choose auto-assign address.</li>

 <li>After the usual cycle of “Generate Bitstream”, “Export Hardware” and “Re-generate BSP Sources”, the sample code should now compile. If it does not, please check the names assigned to stream grabber, as its address is hard-coded into stream grabber.c. Find the right version of XPAR [Block Name] [Instance Name] BASEADDR in xparameters.h and update stream grabber.c</li>

 <li>In SDK, do not forget to run the linker script, map all sections of the memory to the DDR (mig 7series 0 memaddr…) and <strong>not </strong>the BRAM. Also make sure that the the heap and stack sizes are 4096 (4 KB).</li>

</ol>

<h1>2      Octaves</h1>

The frequencies detectable by our microphone range from tens of hertz to the kilohertz range. Octaves are used to divide this wide range of frequencies into smaller groups of frequencies on a logarithmic scale that are labeled 0-9. Each group contains

12 notes:

<em>C,C</em>#<em>,D,D</em>#<em>,E,F,F</em>#<em>,G,G</em>#<em>,A,A</em>#<em>,B</em>

The frequency corresponding to a note is described by the following formula:

where <em>n </em>is the number corresponding to a note (C is 0, A is 9, A# is 10, etc.) and <em>k </em>is the number corresponding to an octave. So <em>A</em>4 is 440 Hz, <em>A</em>1 is 55 Hz, <em>C</em>4 is 261.626 Hz etc. The use of octaves to identify the note is very useful when calculating the FFT and can help decrease the time to compute the FFT by allowing a smaller, lower resolution FFT to be used selectively. (ref: Scratch Wiki list of MIDI notes)

<h1>3       Fast Fourier Transform</h1>

The Fast Fourier transform places recorded samples into frequency bins. The FFT is calculated up to the Nyquist frequency (half the sampling frequency) which here is 24.4140625 kHz. The resolution of the frequencies placed in the upper half of the range is greater than those placed in the lower half of the range. This is shown in the figure below. Looking at Fig. 3:plot(1)

If we are analyzing the raw samples from the microphone, the upper half would contain frequencies ranging from 12 kHz to 24 kHz. This means that if we are trying to detect a lower frequency, like 100 Hz, only a few samples contribute. Furthermore since our PDM to PWM conversion limits the frequencies seen by the microphone to about 15 kHz, we are making little use of the most accurate part of our FFT in the default setup.

There are two ways we could fix our resolution, and potentially increase the speed of our FFT. This is shown above in Fig. 3:plot(2), which computes the FFT using 128 samples. The 128 samples could either be a decimated read from the buffer, where we take only every 4th value, or the samples could be averaged values from the original 512 samples. 4-way averaging the samples makes an effective 128 samples at 48<em>kHz/</em>4 = 12<em>kHz </em>so it will have high resolution for signals from 3 kHz to 6 kHz. <strong>Averaging also acts as a low-pass filter to lower the noise floor by removing higher frequency components.</strong>

On the other hand we could grab, say, 2048 samples at 48 kHz and decimate/average down to 512 samples at 12 kHz to get really good resolution at 3-6 kHz, or even 128 samples at 3 kHz for a computationally efficient look at the 750 Hz-1.5 kHz range.

The above discussion shows that there is a considerable space of trade-offs in resolution vs. run-time. In a practical application, any delay aver 50 ms is noticeable, so the time to get a measurement is <em>&lt; </em>30<em>ms</em>. You need to be able to resolve 10 Hz at 200 Hz input and 10 Hz at 5 kHz (although this might require 2 separate measurement modes).

The audio amplitudes as captured by the microphone are sampled at 48.828125 kHz (100 MHz/2048), with 512 samples per read of the buffer. In the raw output all frequencies are present. To increase the resolution of the FFT at a particular octave, average the samples together to optimize the FFT for the frequency range your musician is playing a note for.

Lab 3A provides default source code to verify functionality of the microphone. The demo code tells the stream grabber to start grabbing data from the microphone, waits for it to fill its buffer, then copies that into a software buffer. This occurs in read fsl values(). The sample code provided in <strong>src/fft.c </strong>computes the Fast Fourier Transform (FFT) and xil printf() outputs the guessed base frequency. The FFT response time for the default code is long and must be improved. To improve the code it is necessary to to conduct a performance analysis. There are three sections of code in particular to focus on.

<ol>

 <li>Focus on the timing around <strong>read fsl values()</strong>, which is the function responsible for reading the 512 microphone sample from the Microblaze buffer.

  <ul>

   <li>The existing code starts the sample grabber, waits until it gets 512 values into the grabber’s buffer, then copies them into a software buffer. In fact the grabber keeps going up to 4096 samples but the software doesn’t need to check this. The hardware is fixed at the microphone’s sampling rate so lower effective sampling rates must be achieved in software.</li>

   <li>The existing code gets 512 values into the grabber, then reads them into the microblaze. Look into modifying the process of reading from the buffer. Try reading less values, or decimating the buffer data. Experiment with the conversion to floating point. Moving a single line of code can result in dramatic changes to your code performance. Equivalently for accuracy a longer capture can be used.</li>

   <li>Given this, experiment with different effective sample frequencies (after decimation) to speed up your FFT and increase the accuracy of your design. Review the earlier section on the FFT and octaves, and think about how you could change the code of the FFT to make sure the target octave falls in the upper half of the FFT.</li>

   <li>Note that the grabber, once started, runs in the background. Thus one can either pipeline the system (start the next grab while computing the current FFT), or use the non-blocking sample checks in a GUI to be more responsive while waiting for data.</li>

  </ul></li>

 <li>Study the FFT code, and focus on finding small changes which result in noticeable speed-ups of your code. The original code uses floating-point arithmetic. Rewriting the FFT to use a fixed-point FFT (which would remove the need for floating point numbers altogether) would allow for a faster FFT.</li>

</ol>

This class of optimization is particularly important since if you can run a 512-point FFT in the time it would take someone else to run a 128-point FFT you can have that extra speed without the accuracy trade-off of a smaller FFT.

Your goal should be to build the base for a fluid interface for the Chromatic Tuner, that has a quick response time with reasonably high accuracy.

<h1>4       Performance Analysis</h1>

Obviously the most trivial way to do performance analysis is to measure time with the AXI timer. For more detailed analysis, a method called “profiling” is used. Here we add an assembly language trick as a simple way to do profiling. We call this trick the “performance monitor”. It allows us to measure the relative proportion of time spent by the program in different sections of the code.

<h2>4.1       Performance Monitor tool</h2>

A performance monitor is a small interrupt driven program, which runs alongside your main program. The performance monitor interrupts on a regular basis, and maintains a log of where the current hardware program counter is at the time of the interrupt.

Specifically it maintains a table of how many times it is found between addresses denoting the lower and upper bounds of the code of each of the procedures in the base line code.

Your performance monitor code needs to be fast and as small as possible, because it must fit in the program space, along with that of the program under analysis.

<h2>4.2       Performance Monitor Details</h2>

In order to analyze the performance of your code, you need a periodic interrupt. When you enter the interrupt handler, you can see where the code was, by checking the return address of the interrupt. This address is stored in register 14 and can be accessed via inline assembly. The necessary code for this is:

<em>uint</em>32 <em>a</em>;

<em>asm</em>(”<em>add </em>%0<em>,r</em>0<em>,r</em>14” : ” = <em>r</em>”(<em>a</em>));

The above code adds zero to register 14 and stores the result in <strong>a</strong>. Once the address is found, compare it to the address in the symbol table of your executable, <strong>your </strong><strong>xsdk name</strong>.elf . The executable is located in

<em>YOUR </em><em>VIVADO NAME</em>.sdk/SDK/SDK Export/<em>YOUR XSDK </em><em>NAME</em>/Debug

To access the symbol table, open your regular terminal, navigate to the folder containing your .elf and type:

nm <strong>your xsdk </strong><strong>name</strong>.elf

Another alternative is to use <strong>objdump</strong>, which has additional options for viewing more information about your code. To see the symbol table using <strong>objdump </strong>type:

objdump -t <strong>your xsdk name</strong>.elf

<h2>4.3      Performance Optimization</h2>

Use the above analysis and your knowledge of what each segment of code is doing to lower the latency below 30 ms, using only software changes in the C code. Hint: This is easier than it sounds. Look for small pieces of code that cost large numbers of cycles and think about what is really needed. It is possible to achieve <em>&lt; </em>30 ms (Record is 10-12 ms) making only software changes to the project, with no loss of accuracy. (You’d be well advised to think about how to ensure that the code precision has not changed, or has improved).

<h1>5       Reporting</h1>

Write a 2 page lab report on the following topics:

<ol>

 <li>Include a Block Design Schematic of your Vivado Project.</li>

 <li>Include a list of the Mic Block internal wiring.</li>

 <li>Include a description of the steps taken for optimizing the original FFT code.</li>

 <li>Was there a difference in the FFT computation for low and high frequencies?</li>

 <li>How does the performance of your FFT code compare to that of the default code?</li>

 <li>Include results from your performance analyzer. The performance analysis should consist of an estimate of the fraction of total execution time spent in each program code module. Estimate the average time you spend in your profile code per sample interrupt (times the size of its code segment). This is the ‘goodness’ metric for your profiler.</li>

</ol>