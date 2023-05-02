Download Link: https://assignmentchef.com/product/solved-comp2300-lab-2
<br>
In this week’s lab you will:

<ol>

 <li>write your own first pieces of machine code</li>

 <li>learn about how the instructions (i.e. the program) are represented and executed on your discoboard</li>

 <li>learn a few assembler directives for getting data into your program</li>

 <li>manipulate values with boolean logic instructions</li>

</ol>

This lab will build your competency towards <a class="acton-tabs-link-processed" href="https://programsandcourses.anu.edu.au/course/ENGN2219#learning-outcomes">Learning Outcomes(LO)</a> 2 &amp; 3.

<hr>

<h2 id="preparation">Preparation</h2>

Before you attend this week’s lab, make sure:

<ol>

 <li><a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/labs/01-intro/#exercise-3-git">You can fork, commit &amp; push your work to GitLab</a></li>

 <li>You can use VSCode to edit assembly (<code class="highlighter-rouge">.S</code>) files</li>

 <li><a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/labs/01-intro/#exercise-5-debugging">You can connect your discoboard to your computer and start a debugging session.</a></li>

 <li>You have read the lab content below</li>

</ol>

If you’re not confident about that stuff then feel free to follow the links to the relevant part of lab 1.

You may find these references useful in this lab – we’ll refer back to them later:

<ul>

 <li><a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-cheat-sheet.pdf">ARM assembly cheat sheet</a></li>

 <li><a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-M-architecture-reference-manual.pdf">ARM®v7-M Architecture Reference Manual</a></li>

</ul>

<hr>

<p class="info-box">Make sure the <code class="highlighter-rouge">COMP2300 Cortex-Debug</code> extension is up to date before starting the lab!

<h2 id="introduction">Introduction</h2>

In the week 2 lectures you saw how your CPU/MCU is built out of logic gates which are built into different components, including registers for storing memory and adders (as part of the <a class="acton-tabs-link-processed" href="https://en.wikipedia.org/wiki/Arithmetic_logic_unit">ALU</a>) for adding numbers together.

Today you will see how to explain to your CPU that your plan is to calculate <code class="highlighter-rouge">2+2</code>, and see how words (i.e. numbers) in memory can be formed and interpreted as opcodes (operation codes) which will instruct your CPU to do things (e.g. add two values).

<h2 id="one-plus-two">Exercise 1: 1+2</h2>

Fork the <a class="acton-tabs-link-processed" href="https://gitlab.cecs.anu.edu.au/engn2219/engn2219-2021-lab-2">lab 2 template</a> on gitlab them clone to your computer with Git, as you did <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/labs/01-intro/#exercise-3-git">last week in lab 1</a>.

Your job in this first exercise is to write an assembly program which calculates <code class="highlighter-rouge">1+2</code> and leaves the result in register 1 (<code class="highlighter-rouge">r1</code>).

Remember from last week’s lab that you can see the values in your discoboard’s registers while debugging, in the registers pane:

<img decoding="async" alt="Register pane" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/register-viewlet.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/register-viewlet.png?w=980&amp;ssl=1" alt="Register pane" data-recalc-dims="1">

 </noscript>

<p class="info-box">You can set the display format for a specific register in the register view. Right click on the register, select “set number format” and then select the desired format. This will help you make sense of the value of a register.

<h3 id="arm-assembly-syntax">ARM assembly syntax</h3>

This is probably the first time you’ve written any ARM assembly code, so for this course we’ve prepared a <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-cheat-sheet.pdf">cheat sheet</a> to help you out. It looks pretty intimidating at first – mostly because it crams a lot of information into a small space. So let’s pick one line of the cheat sheet – the <code class="highlighter-rouge">sub</code> instruction – and pick it apart.

First, the syntax column:

<pre><code class="language-ARM hljs"><span class="hljs-symbol">sub</span>{s}&lt;c&gt;&lt;q&gt; {&lt;Rd&gt;,} &lt;Rn&gt;, &lt;Rm&gt; {,&lt;shift&gt;}</code></pre>

The first token on the line is the instruction name, and after that is the (comma-separated) argument list. Conveniently, all of our assembly instructions will have a similar format.

<ul>

 <li>Anything in braces (<code class="highlighter-rouge">{}</code>) is optional, e.g. the <code class="highlighter-rouge">s</code> at the end of <code class="highlighter-rouge">sub{s}</code> means that it can be either <code class="highlighter-rouge">sub</code> or <code class="highlighter-rouge">subs</code>. Adding <code class="highlighter-rouge">s</code> to instructions will cause flags to be set by the operation – we’ll cover this later in the course.</li>

 <li>The <code class="highlighter-rouge">&lt;c&gt;</code> and <code class="highlighter-rouge">&lt;q&gt;</code> parts relate to the condition codes and opcode size boxes on the second page of the cheat sheet. They are also optional and we’ll visit these later in the course.</li>

 <li><code class="highlighter-rouge">{&lt;Rd&gt;,}</code> is the destination register (e.g. <code class="highlighter-rouge">r3</code> or <code class="highlighter-rouge">r11</code>), where the result of the instruction is stored. If you do not specify a destination register, <code class="highlighter-rouge">&lt;Rn&gt;</code> will be used instead.</li>

 <li><code class="highlighter-rouge">&lt;Rn&gt;, &lt;Rm&gt;</code> are the two operands (arguments) for the <code class="highlighter-rouge">sub</code> instruction.</li>

 <li>Finally, the optional <code class="highlighter-rouge">{,&lt;shift&gt;}</code> is for using the discoboard’s barrel shifter to do logical shifts. We’ll also cover this later in the course.</li>

</ul>

There are a couple of other parts of the syntax which aren’t covered in the <code class="highlighter-rouge">sub</code> instruction:

<ul>

 <li>Instructions which use constant values will use decimal by default. You can prefix your values to indicate a different base: <code class="highlighter-rouge">0b</code> for binary (e.g. <code class="highlighter-rouge">0b1101101</code>), <code class="highlighter-rouge">0o</code> for octal (e.g. <code class="highlighter-rouge">0o125</code>) or <code class="highlighter-rouge">0x</code> for hexadecimal (<code class="highlighter-rouge">0xef20</code>).</li>

 <li>When it comes to load &amp; store operations, square brackets <code class="highlighter-rouge">[]</code> indicate that the instruction should use the memory address in the register, e.g. <code class="highlighter-rouge">[r2]</code> tells the discoboard to “use the memory address in <code class="highlighter-rouge">r2</code>” for that instruction.</li>

</ul>

You won’t need to know all of this stuff to complete this lab, so just remember that it’s here if you need to come back to it.

The semantic column on your cheat sheet describes what the instruction does. For example, the semantic for the <code class="highlighter-rouge">sub</code> instruction is <code class="highlighter-rouge">Rd(n) := Rn - Rm{shifted}</code>, which in English translates to something like:

<blockquote>

 in the <code class="highlighter-rouge">Rd</code> register (or <code class="highlighter-rouge">Rn</code>, if <code class="highlighter-rouge">Rd</code> was not specified) store the result of subtracting the value in the <code class="highlighter-rouge">Rm</code> register (with an optional bit-shift, if present) from the value in the <code class="highlighter-rouge">Rn</code> register.

</blockquote>

You can probably see why we use assembly language for telling our CPU what to do rather than English – it’s much less wordy.

The flags column of the cheat sheet specifies which of the special condition code flags that instruction sets if the optional <code class="highlighter-rouge">s</code> suffix is present. (We’ll cover this in next week’s lab, but if you’re curious there’s a box on the second page of the cheat sheet which lists the flags.)

<p class="info-box">Since there is a lot of information in the ARM instruction syntax, you don’t need to memorize everything. Just keep the cheat sheet nearby and take a closer look when you need to find a specific syntax. <strong>You will be given an assembly instruction cheat-sheet in exams.</strong>

Please be aware that there are several instructions that can use different sets of arguments. For example, the <code class="highlighter-rouge">sub</code> instruction can be:

<ul>

 <li><code class="highlighter-rouge">sub</code> with <code class="highlighter-rouge">&lt;Rn&gt;, &lt;Rm&gt; {,&lt;shift&gt;}</code></li>

 <li><code class="highlighter-rouge">sub</code> with <code class="highlighter-rouge">&lt;Rn&gt;, #&lt;const&gt;</code> Both of them will do the same <code class="highlighter-rouge">sub</code> operation, but using different parameters for executing the <code class="highlighter-rouge">sub</code>. For this instruction, we can either subtract two registers, or a register and a constant value.</li>

</ul>

<h3 id="the-task">The task</h3>

To actually complete your “1 + 2” task, you’ll need to

<ol>

 <li>get number <code class="highlighter-rouge">1</code> into any register</li>

 <li>add <code class="highlighter-rouge">2</code> to the register value and put the result in <code class="highlighter-rouge">r1</code></li>

</ol>

<p class="think-box">Look over the cheat sheet—which assembly instructions allow you to place a constant value into a register? There are also a number of machine instructions which will implement an addition – which one do you want, and <em>why?</em>

Once you’ve written a program which you think will do what you want, step through it with the debugger and make sure that the value which <code class="highlighter-rouge">r1</code> holds at the end is actually <code class="highlighter-rouge">3</code>.

<p class="info-box">The method you use to find “1 + 2” is only one solution to the problem. There are other ways that we can use to answer the question using different instructions. Can you think of any?

<p class="push-box">Save &amp; make a commit now that you have finished <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/labs/02-first-machine-code/#one-plus-two">Exercise 1</a>. It’s a good idea to keep a version once you have completed each exercise in the lab; that’s what a version control system is for, right?

<h2 id="reverse-engineering">Exercise 2: reverse engineering with the memory viewer</h2>

Now we really look under the hood and leave no bit unturned. Start a debugging session with your program from the previous exercise and step through until you get to <code class="highlighter-rouge">main</code>, then leave it “hanging”—do not execute any further. It will “pause” the program execution for the moment.

<p class="info-box">Although most of the process of code execution is displayed inside VSCode, in reality, all the register and memory values are taken directly from your discoboard’s CPU.

<p class="think-box">Do you know where each of your numbers are represented in your program when it’s actually running on your discoboard?

To do that we need to be able to view the memory in the disco board. In VSCode you can look at your memory directly using the Memory View: type <code class="highlighter-rouge">memory</code> in the <a class="acton-tabs-link-processed" href="https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette">command palette</a> and select the Cortex Debug view memory option. VSCode will then ask you to input the starting address and the number of bytes to read. In the example below, the starting adress used is <code class="highlighter-rouge">0x080001dc</code>, and <code class="highlighter-rouge">512</code> bytes of memory have been read.

<img decoding="async" alt="Memory view" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/memory-view.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/memory-view.png?w=980&amp;ssl=1" alt="Memory view" data-recalc-dims="1">

 </noscript>

This might look overwhelming, but the 2D “grid” layout is pretty simple: the hex numbers down the left hand side are the base memory addresses, and the hex numbers along the top represent the “offset” of that particular byte from the base address. So, to work out the exact address of a particular byte, add the row and column values(base+offset).

<p class="think-box">You can find the bytes in memory which correspond to the instructions you wrote in your <code class="highlighter-rouge">main.S</code> file using this view. The trick is figuring out where to look—what should the starting address be? Even your humble discoboard has a lot of addressable memory. Discuss with your lab neighbor—where should you look to find your program? The value in the <code class="highlighter-rouge">pc</code>, or <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/lectures/week-2/#the-pc-register">program counter</a> may be a good place to start. See if you can figure out exactly what this value represents.

There’s an assembler directive called <a class="acton-tabs-link-processed" href="https://sourceware.org/binutils/docs/as/hword.html#hword"><code class="highlighter-rouge">.hword</code></a> which you can use to put 16-bit numbers into your program (“hword” is short for half-word, and comes from the fact that your discoboard’s CPU uses 32-bit “words”).

Modify your program to use the <code class="highlighter-rouge">.hword</code> directive to put some data into your program, so it looks something like this:

<pre><code class="language-arm hljs armasm"><span class="hljs-symbol">.syntax</span> unified<span class="hljs-symbol">.global</span> main<span class="hljs-symbol">.type</span> main, %<span class="hljs-meta">function</span><span class="hljs-symbol">main:</span>  <span class="hljs-keyword">nop</span>  <span class="hljs-meta">.hword</span> <span class="hljs-number">0xdead</span>  <span class="hljs-meta">.hword</span> <span class="hljs-number">0xbeef</span>  <span class="hljs-keyword">b</span> main<span class="hljs-symbol">.size</span> main, .-main</code></pre>

Note we need to add a <code class="highlighter-rouge">nop</code> instruction here for the debugger to work correctly – <code class="highlighter-rouge">0xdead</code> isn’t a real instruction, and our CPU can get confused.

<p class="talk-box">What do you think a <code class="highlighter-rouge">nop</code> instruction does? Aside from this very specific case, when do you think such an instruction might be useful?

Build &amp; upload your program and open a memory viewer session. Use the address of your first instruction as the base address, and load enough bytes to view your entire program. Can you see the <code class="highlighter-rouge">0xdead</code> and <code class="highlighter-rouge">0xbeef</code> values you put into your program?

<p class="think-box">If you can’t see them <em>exactly</em>, can you see something which looks suspiciously like them? What do you notice?

<h3 id="endianness">Endianness</h3>

To make sense of the numbers displayed in the memory view, we need to talk about endianness.

Values are stored in memory as individual bytes (i.e. 8-bit numbers, which can be represented with two hex digits). Endianness refers to the order in which these small 8-bit bytes are arranged into larger numbers (e.g. 32-bit words). In the little-endian format used by our discoboards, the byte stored at the lowest address is the least significant byte(LSB). Big-endian is the opposite – the byte stored at the lowest address is the most significant byte(MSB).

Here’s an example: suppose we have the number <code class="highlighter-rouge">0x01</code> stored at a lower memory address (e.g. <code class="highlighter-rouge">0x000001e0</code>), and the number <code class="highlighter-rouge">0xF1</code> stored at a higher memory address (<code class="highlighter-rouge">0x000001e1</code>), as shown below:

<img decoding="async" alt="Endian example" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/endian-example.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/endian-example.png?w=980&amp;ssl=1" alt="Endian example" data-recalc-dims="1">

 </noscript>

If we tell our CPU to read a half-word(16 bits) from the memory address <code class="highlighter-rouge">0x000001e0</code> under the little-endian format, it represents <code class="highlighter-rouge">0xF101</code> (the <code class="highlighter-rouge">0x01</code> at the lower address is treated as less significant). In a CPU under the big-endian format, it represents <code class="highlighter-rouge">0x01F1</code> (the <code class="highlighter-rouge">0x01</code> at the lower address is treated as more significant).

When reading four bytes from the memory, the CPU can read them as four 8-bit bytes, two 16-bit half-words, or as one 32-bit word. The endianness format applies everytime when combining bytes into bigger words. The following diagram illustrates this using the little-endian format:

<img decoding="async" alt="Little-endian byte ordering" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/little-endian.jpg?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/little-endian.jpg?w=980&amp;ssl=1" alt="Little-endian byte ordering" data-recalc-dims="1">

 </noscript>

You need to be aware of this byte ordering to make sense of the memory view.

<p class="extension-box">How might you figure out on your discoboard’s Cortex-M4 CPU whether a 32-bit instruction is read as one 32-bit word or as two 16-bit half-words? (hint: have a look at A5.1 in the <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-M-architecture-reference-manual.pdf">ARM®v7-M Architecture Reference Manual</a>).

<p class="info-box">According to <a class="acton-tabs-link-processed" href="https://en.wikipedia.org/wiki/Endianness#Etymology">Wikipedia</a>, Danny Cohen introduced the terms <em>Little-Endian</em> and <em>Big-Endian</em> for byte ordering in an article from 1980. In this technical and political examination of byte ordering issues, the “endian” names were drawn from Jonathan Swift’s 1726 satire, Gulliver’s Travels, in which civil war erupts over whether the big end or the little end of a boiled egg is the proper end to crack open, which is analogous to counting from the end that contains the most significant bit or the least significant bit.

<h3 id="instruction-encodings">Instruction encoding(s)</h3>

Now that you know how bytes fit together into words, let’s get back to the task of figuring out how the instructions in your program are encoded in memory.

To help, you can use the known half-words you put into your program earlier to help you out. Update your program like so to add a single instruction (i.e. one line of assembly code) from the your 1+2 program you wrote in <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/labs/02-first-machine-code/#one-plus-two">Exercise 1</a>.

<pre><code class="language-arm hljs armasm"><span class="hljs-symbol">.syntax</span> unified<span class="hljs-symbol">.global</span> main<span class="hljs-symbol">.type</span> main, %<span class="hljs-meta">function</span><span class="hljs-symbol">main:</span>  <span class="hljs-keyword">nop</span>  <span class="hljs-meta">.hword</span> <span class="hljs-number">0xdead</span>  <span class="hljs-comment">@ put a single "real" assembly instruction here from your 1+2 program</span>  <span class="hljs-meta">.hword</span> <span class="hljs-number">0xbeef</span>  <span class="hljs-keyword">b</span> main<span class="hljs-symbol">.size</span> main, .-main</code></pre>

Build, upload and start a new debug session, then find your program again in the memory view. <strong>What does your instruction look like in memory?</strong> Try making a note of the bytes, then modify the instruction arguments (e.g. change the number, or the register you’re using) and see how the bytes change in memory (you’ll need to re-build &amp; run your program and call the view memory command each time you do).

<p class="extension-box">Discuss with your neighbour: what do you think the different bits (and bytes) in the instruction mean? How does the discoboard know what to do with them? And if you’ve figured that out, why doesn’t your program actually work as written?

To fully make sense of these instruction encodings you need more than just your cheat sheet – you need the <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-M-architecture-reference-manual.pdf#G11.4953054">ARM®v7-M Architecture Reference Manual</a>. Dig to the deepest levels of the manual, by going to section A7.7 Alphabetical list of ARMv7-M Thumb instructions (page A7-184). Use the bookmarks in your pdf viewer to navigate to the relevant instructions inside this huge document.

For each instruction you will see a number of encodings. They detail bit-by-bit the different ways of specifying the machine instructions that your discoboard CPU understands. You may find this number format conversion tool helpful:

<table class="conversion-widget">

 <tbody>

  <tr>

   <td class="basetype">Decimal</td>

   <td><input id="dec_val" type="text" value="0"></td>

  </tr>

  <tr>

   <td class="basetype">Hex</td>

   <td><input id="hex_val" type="text" value="0x 0000 0000"></td>

  </tr>

  <tr>

   <td class="basetype">Binary</td>

   <td><input id="bin_val" type="text" value="0b 0000 0000 0000 0000 0000 0000 0000 0000"></td>

  </tr>

 </tbody>

</table>

<p class="push-box">Commit your “reverse engineering” program with a comment about what the instruction looks like in memory. It doesn’t matter that it doesn’t actually run at this point—you’ll get there in the next exercise.

<p class="extension-box">Can you tell which specific encoding has been used for the instruction you wrote earlier in exercise 1? Note that not every encoding can express every version of the instruction, but sometimes a more complex encoding can also express what the a simpler form could have done as well. Can you hint the assembler towards the specific encoding you want?

<h2 id="excercise-3-hand-crafted-instructions">Excercise 3: hand-crafted instructions</h2>

Now that you’ve identified the spot in memory where your instructions live, in this exercise we turn our approach around and program the CPU by writing specific numbers directly to memory locations. Where we earlier inserted <code class="highlighter-rouge">0xdead</code> and <code class="highlighter-rouge">0xbeef</code>, we are going to insert hex values that correspond to the machine code for real instructions.

Instead of calculating <code class="highlighter-rouge">1+2</code>, you are going to make the CPU calculate <code class="highlighter-rouge">3-1</code> by putting the right numbers into memory.

In fact, you have been doing this all along, except that the assembler has helped you by converting your human-readable instructions in to their raw machine code representations. Replace your program with the following assembly code:

<pre><code class="language-arm hljs armasm"><span class="hljs-symbol">.syntax</span> unified<span class="hljs-symbol">.global</span> main<span class="hljs-symbol">.type</span> main, %<span class="hljs-meta">function</span><span class="hljs-symbol">main:</span>  <span class="hljs-keyword">nop</span>  <span class="hljs-meta">.hword</span> <span class="hljs-number">0xffff</span>  <span class="hljs-meta">.hword</span> <span class="hljs-number">0xffff</span>  <span class="hljs-keyword">b</span> main<span class="hljs-symbol">.size</span> main, .-main</code></pre>

This time we want you to put on your assembler hat and figure out the actual <code class="highlighter-rouge">.hword</code> values which will make the CPU load <code class="highlighter-rouge">3</code> into a <code class="highlighter-rouge">r1</code> and subtract <code class="highlighter-rouge">1</code> from it. Remember that it’ll be similar to the words you looked at in the memory viewer earlier, but some of the bits will be different (since we’re dealing with <code class="highlighter-rouge">-</code>, <code class="highlighter-rouge">3</code> and <code class="highlighter-rouge">1</code> instead of <code class="highlighter-rouge">+</code>, <code class="highlighter-rouge">1</code> and <code class="highlighter-rouge">2</code>).

<p class="info-box">You can find the architecture reference manual pages for the <code class="highlighter-rouge">add</code> instruction <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-M-architecture-reference-manual.pdf#G11.4954505">here</a>, and the pages for <code class="highlighter-rouge">mov</code> <a class="acton-tabs-link-processed" href="https://cs.anu.edu.au/courses/engn2219/assets/manuals/ARMv7-M-architecture-reference-manual.pdf#G11.5006667">here</a>.

Note the line at the bottom:

<pre><code class="language-arm hljs armasm"><span class="hljs-symbol">.size</span> main, .-main</code></pre>

This tells the assembler the size of the <code class="highlighter-rouge">main</code> function, and it is essential for the <strong>disassembler</strong> to work correctly. The disassembler view can be opened by typing <code class="highlighter-rouge">Cortex-Debug: View Disassembly (Function)</code> in the command palette during an active debug session. It will then ask for which function to disassemble, type the function name (e.g. <code class="highlighter-rouge">main</code>). It will look something like this:

<img decoding="async" alt="Disassembly view" data-recalc-dims="1" data-src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/disassembly-view.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/cs.anu.edu.au/courses/engn2219/assets/labs/lab-2/disassembly-view.png?w=980&amp;ssl=1" alt="Disassembly view" data-recalc-dims="1">

 </noscript>

Bring up the disassembler view for <code class="highlighter-rouge">main</code>—you’re now looking at the program as it will be understood by the CPU.

<p class="think-box">Looking at the disassembled code (i.e. the way the CPU will interpret the instructions) did you see what you intended? Will your new, hand-assembled program show the correct result in <code class="highlighter-rouge">r1</code> after it has been run? If it doesn’t, what might have gone wrong?

You can now cheer as loud as your upbringing allows that you will never again have to hand-craft the bits needed to instruct the CPU to do this, and you can leave this job to the assembler from now on.

As a side effect, you also learnt something about security: your system can be compromised by injecting some data into memory (an array of numbers, a string or anything which the host system would accept) and making the CPU somehow stumble into executing it.

<p class="talk-box">Discuss with your lab neighbour: after completing this exercise, how would you explain the way a CPU works to your grandma/grandpa?

<p class="push-box">Make a commit now that you’ve knocked down Exercise 3. Congrats!

<h2 id="exercise-4-boolean-logic-bit-vectors-and-labels">Exercise 4: Boolean logic, bit vectors, and labels</h2>

Load some new data into register <code class="highlighter-rouge">r1</code> by adding this assembly code to your program:

<pre><code class="language-arm hljs armasm">  <span class="hljs-comment">@ load "COPE" into r1</span>  <span class="hljs-keyword">ldr</span> <span class="hljs-built_in">r1</span>, string<span class="hljs-symbol">loop:</span>  <span class="hljs-keyword">nop</span>  <span class="hljs-keyword">b</span> loop<span class="hljs-symbol">string:</span>  <span class="hljs-meta">.ascii</span> <span class="hljs-string">"COPE"</span></code></pre>

This code introduces a new assembler directive: labels.

<p class="extension-box">If you’re the kind of person who likes documentation, you can find it <a class="acton-tabs-link-processed" href="https://sourceware.org/binutils/docs/as/Labels.html#Labels">here.</a>

In the above code there are two new labels: <code class="highlighter-rouge">string</code> and <code class="highlighter-rouge">loop</code>. A label is a way of attaching a human-readable name to a location in your program. Always remember that labels are just a name attached to a memory location – when the assembler builds your program, it will have a specific memory address which you can store in a register, do arithmetic on, etc.

<p class="info-box">While you can use just about anything for a label name, try and use something informative – as if you are naming a function or variable.

<p class="think-box">What will you see in <code class="highlighter-rouge">r1</code> after the <code class="highlighter-rouge">ldr r1, string</code> line? Can you guess the address of <code class="highlighter-rouge">string</code> and find it in the memory viewer?

You’ll notice a new <a class="acton-tabs-link-processed" href="https://sourceware.org/binutils/docs/as/Ascii.html#Ascii"><code class="highlighter-rouge">.ascii</code></a> compiler directive in this code: this allows you to put data into your program using the <a class="acton-tabs-link-processed" href="https://en.wikipedia.org/wiki/ASCII">ASCII</a> encoding. This works the same as the <code class="highlighter-rouge">.hword</code> directive you used earlier, except for the data format. While <code class="highlighter-rouge">.hword</code> uses numbers, <code class="highlighter-rouge">.ascii</code> takes characters and encodes each one to a specific byte value. You can find a table of characters and their ascii-encoded value <a class="acton-tabs-link-processed" href="https://en.wikipedia.org/wiki/ASCII#Printable_characters">here.</a>

Your goal in this exercise is to isolate and modify individual bytes within the <code class="highlighter-rouge">"COPE"</code> word:

<ol>

 <li>first, change it into <code class="highlighter-rouge">"HOPE"</code> and store in <code class="highlighter-rouge">r2</code></li>

 <li>then, change it into <code class="highlighter-rouge">"HOPS"</code> and store in <code class="highlighter-rouge">r3</code></li>

</ol>

Each of these steps requires isolating and manipulating one 8-bit (1-character) part of the 32-bit word without messing with the rest of it. What boolean logic or arithmetic operations can you use to modify the appropriate bits and bytes?

It might be helpful to use a piece of paper here: write out what the <code class="highlighter-rouge">"COPE"</code> data looks like in memory (remember endianness!), and figure out what operations you need to make the transformations into <code class="highlighter-rouge">"HOPS"</code>.

There are several ways to do this, how many can you think of? Show your program to your neighbour or tutor to get ideas about how it could be done differently.

<p class="extension-box">You might have used one (or more) large numbers in your bit manipulation adventures. How many bits is that number? Do you think it will fit into the instruction encoding? Have a look at the disassembly, and cross check it with the instruction manual. Can you make sense of what’s happening here? (Hint: <a class="acton-tabs-link-processed" href="https://alisdair.mcdiarmid.org/arm-immediate-value-encoding/">this blog post</a> might be helpful to you). Give it a crack, but we will revisit this again later in the course.

<p class="push-box">Finalise your program so that the <code class="highlighter-rouge">main</code> function performs the <code class="highlighter-rouge">"COPE"</code> -&gt; <code class="highlighter-rouge">"HOPE"</code> -&gt; <code class="highlighter-rouge">"HOPS"</code> transformation and leaves the <code class="highlighter-rouge">"HOPS"</code> value in <code class="highlighter-rouge">r2</code>. Commit &amp; push your changes up to your repo on the GitLab server.