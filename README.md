Download Link: https://assignmentchef.com/product/solved-ece463-563-project-1-cache-design-memory-hierarchy-design-version-1-0
<br>
<h1>1. Groundrules</h1>

This is the first project for the course, so let me begin by discussing some groundrules:




<ol>

 <li>All students must work alone. The project scope is reduced (but still substantial) for ECE 463 students, as detailed in this specification.</li>

 <li>Sharing of code between students is considered <u>cheating</u> and will receive appropriate action in accordance with University policy. <u>The TAs will scan source code (from current and past</u> <u>semesters) through various tools available to us for detecting cheating. Source code that is</u> <u>flagged by these tools will be dealt with severely: 0 on the project and referral to the Office</u> <u>of Student Conduct.</u></li>

 <li>A Moodle message board will be provided for posting questions, discussing and debating issues, and making clarifications. It is an essential aspect of the project and communal learning. Students must not abuse the message board, whether inadvertently or intentionally. Above all, never post actual code on the message board (unless permitted by the TAs/instructor). When in doubt about posting something of a potentially sensitive nature, email the TAs and instructor to clear the doubt.</li>

 <li>It is recommended that you do all your work in the C, C++ or Java languages. Exceptions must be approved by the instructor.</li>

 <li><u>Use of the Eos Linux environment is <em>required</em>. This is the platform where the TAs will</u> <u>compile and test your simulator.</u> (WARNING: If you develop your simulator on another platform, get it working on that platform, and then try to port it over to Eos Linux at the last minute, you may encounter major problems. Porting is not as quick and easy as you think. What’s worse, malicious bugs can be hidden until you port the code to a different platform, which is an unpleasant surprise close to the deadline.)</li>

</ol>




<h1>2. Project Description</h1>

In this project, you will implement a flexible cache and memory hierarchy simulator and use it to compare the performance, area, and energy of different memory hierarchy configurations, using a subset of the SPEC-2000 benchmark suite.

<h1>3. Specification of Memory Hierarchy</h1>

Design a generic cache module that can be used at any level in a memory hierarchy. For example, this cache module can be “instantiated” as an L1 cache, an L2 cache, an L3 cache, and so on. Since it can be used at any level of the memory hierarchy, it will be referred to generically as CACHE throughout this specification.

<h2>3.1. Configurable parameters</h2>

CACHE should be configurable in terms of supporting any cache size, associativity, and block size, specified at the beginning of simulation:

<ul>

 <li>SIZE: Total bytes of data storage.</li>

 <li>ASSOC: The associativity of the cache. (ASSOC = 1 is a direct-mapped cache. ASSOC</li>

</ul>

= # blocks in the cache = SIZE/BLOCKSIZE is a fully-associative cache.) o BLOCKSIZE: The number of bytes in a block.




There are a few constraints on the above parameters: 1) BLOCKSIZE is a power of two and 2) the number of <em>sets</em> is a power of two. <em>Note that ASSOC (and, therefore, SIZE) need not be a power of two</em>. As you know, the number of sets is determined by the following equation:

#<em>sets </em>= <em>SIZE         </em>

<h2>ASSOC<sub>× </sub>BLOCKSIZE 3.2. Replacement policy</h2>

CACHE should use the LRU (least-recently-used) replacement policy.




<em>Note regarding simulator output: When printing out the final contents of CACHE, you must print the blocks within a set based on their recency of access, i.e., print out the MRU block first, the next most recently used block second, and so forth, and the LRU block last. This is required so that your output is consistent with the output of the TAs’ simulator. </em>

<h2>3.3. Write policy</h2>

CACHE should use the WBWA (write-back + write-allocate) write policy.

o Write-allocate: A write that misses in CACHE will cause a block to be allocated in CACHE. Therefore, both write misses and read misses cause blocks to be allocated in CACHE. o Write-back: A write updates the corresponding block in CACHE, making the block dirty. It does not update the next level in the memory hierarchy (next level of cache or memory). If a dirty block is evicted from CACHE, a writeback (<em>i.e.</em>, a write of the entire block) will be sent to the next level in the memory hierarchy.

<h2>3.4. Allocating a block: Sending requests to next level in the memory hierarchy</h2>

Your simulator must be capable of modeling one or more instances of CACHE to form an overall memory hierarchy, as shown in Fig. 1.




CACHE receives a read or write request from whatever is above it in the memory hierarchy (either the CPU or another cache). The only situation where CACHE must interact with the next level below it (either another CACHE or main memory) is when the read or write request misses in CACHE. When the read or write request misses in CACHE, CACHE must “allocate” the requested block so that the read or write can be performed.




Thus, let us think in terms of allocating a requested block X in CACHE. The allocation of requested block X is actually a two-step process. <em>The two steps must be performed in the following order</em>.

<ol>

 <li><em>Make space for the requested block X</em>. If there is at least one invalid block in the set, then there is already space for the requested block X and no further action is required (go to step 2). On the other hand, if all blocks in the set are valid, then a victim block V must be singled out for eviction, according to the replacement policy (Section 3.2). If this victim block V is dirty, then a write of the victim block V must be issued to the next level of the memory hierarchy.</li>

 <li><em>Bring in the requested block X</em>. Issue a read of the requested block X to the next level of the memory hierarchy and put the requested block X in the appropriate place in the set (as per step 1).</li>

</ol>




To summarize, when allocating a block, CACHE issues a write request (<em>only</em> if there is a victim block and it is dirty) followed by a read request, both to the next level of the memory hierarchy. Note that each of these two requests could themselves miss in the next level of the memory hierarchy (if the next level is another CACHE), causing a cascade of requests in subsequent levels. <em>Fortunately, you only need to correctly implement the two steps for an allocation locally within CACHE. If an allocation is correctly implemented locally (steps 1 and 2, above), the memory hierarchy as a whole will automatically handle cascaded requests globally. </em>




<strong>Fig. 1: Your simulator must be capable of modeling one or more instances of CACHE to form an overall memory hierarchy. </strong>

<h2>3.5. Updating state</h2>

After servicing a read or write request, whether the corresponding block was in the cache already (hit) or had just been allocated (miss), remember to update other state. This state includes LRU counters affiliated with the set as well as the valid and dirty bits affiliated with the requested block.

<h1>4. ECE 563 Students: Augment CACHE with a Victim Cache</h1>

Students enrolled in ECE 563 must additionally augment CACHE with a Victim Cache (VC).




In this project, consider the VC to be an extension implemented within CACHE. This preserves the clean abstraction of one or more instances of CACHE interacting in an overall memory hierarchy (see Fig. 1), where each CACHE may have a VC within it.

<strong><em>4.1. Victim cache can be enabled or disabled </em></strong>

Your simulator should be able to specify, for each CACHE, whether or not its VC is enabled.

<h2>4.2. Victim cache parameters</h2>

The VC is fully-associative and uses the LRU replacement policy. The number of blocks in the VC should be configurable. A CACHE and its VC have the same BLOCKSIZE.

<h2>4.3. Operation: Interaction between CACHE and its own VC</h2>

This section describes how CACHE and its VC interact, when its VC is enabled.




The only time CACHE and its VC <em>may</em> interact is when a read or write request misses in CACHE. In this case, the requested block X was not found in CACHE. If the corresponding set has at least one invalid block (<em>the set is not full</em>), then this set has never transferred a victim block to VC, therefore, VC cannot possibly have the requested block X and need not be searched. In this special case, CACHE need not interact with VC and instead goes directly to the next level. (If you want, as a sanity check, you can “assert” VC does not have requested block X.) Suppose, however, that the set does not have any invalid blocks (<em>the set is full</em>). In this more common case, a victim block V is singled out for eviction from CACHE, based on its replacement policy. CACHE issues a “swap request [X, V]” to its VC. Here is how the swap request works.




If VC has block X, then block X and block V are swapped:

<ul>

 <li>block V moves from CACHE to VC (it is placed where block X was), and o block X moves from VC to CACHE (it is placed where block V was).</li>

</ul>




Note the following:

<ul>

 <li>When swapping the blocks, everything is carried along with the blocks, including dirty bits.</li>

 <li>After swapping the blocks, block X is considered to be the most-recently-used block in its set in CACHE and block V is considered to be the most-recently-used block in VC.</li>

 <li>After swapping the blocks, the dirty bit of block X in CACHE may need to be updated (based on whether the original request to block X is a read or a write).</li>

</ul>




On the other hand, if VC does not have block X, then three steps must be performed in the following order:

<ol>

 <li>VC must make space for block V from CACHE. If VC is not full (it has at least one invalid block), then there is already space. If VC is full (only valid blocks), then VC singles out a secondary victim block V2 for eviction based on its replacement policy. If secondary victim block V2 is dirty, then VC must issue a write of secondary victim block V2 to the next level in the memory hierarchy with respect to CACHE.</li>

 <li>Block V (including its dirty bit) is put in VC, in the appropriate place as determined in step 1 above (<em>e.</em>, it replaces either an invalid block or secondary victim block V2).</li>

 <li>CACHE issues a read of requested block X to the next level in the memory hierarchy.</li>

</ol>

<h1>5. Memory Hierarchies to be Explored in this Project</h1>

While Fig. 1 illustrates an arbitrary memory hierarchy, you will only study the memory hierarchy configurations shown in Fig. 2a (ECE 463) and Fig. 2b (ECE 563). Also, these are the only configurations the TAs will test.




For this project, all CACHEs in the memory hierarchy will have the same BLOCKSIZE.




<table>

 <tbody>

  <tr>

   <td width="172"></td>

   <td width="135"></td>

   <td width="32"></td>

   <td width="136"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

   <td></td>

   <td rowspan="2"></td>

  </tr>

  <tr>

   <td></td>

  </tr>

 </tbody>

</table>

from CPU                                                                       from CPU

<strong>Fig. 2a: ECE463: Two configurations to be studied. </strong>







from CPU                                                                       from CPU                                                               from CPU                                                                        from CPU




Main Memory                                                                                    Main Memory

<strong>Fig. 2b: ECE563: Four configurations to be studied. </strong>




<h1>6. Inputs to Simulator</h1>

The simulator reads a trace file in the following format:




r|w &lt;hex address&gt; r|w &lt;hex address&gt; …




“r” (read) indicates a load and “w” (write) indicates a store from the processor.




Example:




r ffe04540 r ffe04544 w 0eff2340 r ffe04548 …




Traces are posted on the Moodle website.










<strong><u>NOTE:</u> </strong>

All addresses are 32 bits. When expressed in hexadecimal format (hex), an address is 8 hex digits as shown in the example trace above. In the actual trace files, you may notice some addresses are comprised of fewer than 8 hex digits: this is because there are leading 0’s which are not explicitly shown. For example, an address “ffff” is really “0000ffff”, because all addresses are 32 bits, <em>i.e.</em>, 8 nibbles.

<h1>7. Outputs from Simulator</h1>




Your simulator should output the following: (see posted validation runs for exact format) 1. Memory hierarchy configuration and trace filename.

<ol start="2">

 <li>The final contents of all caches.</li>

 <li>The following measurements:</li>

</ol>




<ol>

 <li>number of L1 reads</li>

 <li>number of L1 read misses</li>

 <li>number of L1 writes</li>

 <li>number of L1 write misses</li>

 <li>number of swap requests from L1 to its VC (<em>swap requests=0 if no VC or VC disabled</em>)</li>

</ol>

Note: As described earlier, a “swap request [X, V]” is when the L1 searches its VC for X.

<ol>

 <li>swap request rate = SRR = (swap requests)/(L1 reads + L1 writes)</li>

 <li>number of swaps between L1 and its VC (<em>swaps=0 if no VC or VC is disabled</em>) Note: A “swap” is when X is found in VC, leading to swapping of X and V.</li>

 <li>combined L1+VC miss rate = MR<sub>L1+VC</sub> =</li>

</ol>

(L1 read misses + L1 write misses – swaps)/(L1 reads + L1 writes)

<ol>

 <li>number of writebacks from L1 or its VC (if enabled), to next level</li>

 <li>number of L2 reads (<em>should match: L1 read misses + L1 write misses – swaps</em>)</li>

 <li>number of L2 read misses</li>

 <li>number of L2 writes (<em>should match i: number of writebacks from L1 or its VC</em>)</li>

 <li>number of L2 write misses</li>

 <li>L2 miss rate (<em>from standpoint of stalling the CPU</em>) = MR<sub>L2</sub> = (L2 read misses)/(L2 reads)</li>

 <li>number of writebacks from L2 to memory</li>

 <li>total memory traffic = number of blocks transferred to/from memory</li>

</ol>

(<em>with L2,</em> <em>should match k+m+o: L2 read misses + L2 write misses + writebacks from L2</em>)

(<em>without L2, should match: L1 read misses + L1 write misses – swaps + writebacks from L1 or VC</em>)




<h1>8. Validation and Other Requirements</h1>

<h2>8.1. Validation requirements</h2>

Sample simulation outputs will be provided on the website. These are called “validation runs”. You must run your simulator and debug it until it matches the validation runs.




Each validation run includes:

<ol>

 <li>Memory hierarchy configuration and trace filename.</li>

 <li>The final contents of all caches.</li>

 <li>All measurements described in Section 7.</li>

</ol>




Your simulator must print outputs to the console (<em>i.e.</em>, to the screen). (Also see Section 8.2 about this requirement.)




Your output must match both <u>numerically</u> and in terms of <u>formatting</u>, because the TAs will literally “diff” your output with the correct output. You must confirm correctness of your simulator by following these two steps for each validation run:

1) Redirect the console output of your simulator to a temporary file. This can be achieved by placing     &gt; your_output_file     after the simulator command. 2) Test whether or not your outputs match properly, by running this unix command: diff –iw &lt;<em>your_output_file</em>&gt; &lt;<em>posted_output_file</em>&gt;

The –iw flags tell “diff” to treat upper-case and lower-case as equivalent and to ignore the amount of whitespace between words. Therefore, you do not need to worry about the exact number of spaces or tabs as long as there is some whitespace where the validation runs have whitespace.

<h2>8.2. Requirements for compiling and running simulator</h2>

You will hand in source code and the TAs will compile and run your simulator. As such, you must meet the following strict requirements. Failure to meet these requirements will result in point deductions (see section “Grading”).




<ol>

 <li>You must be able to compile and run your simulator on Eos Linux machines. This is required so that the TAs can compile and run your simulator.</li>

 <li>Along with your source code, you must provide a <strong>Makefile</strong> that automatically compiles the simulator. This Makefile must create a simulator named “sim_cache”. The TAs should be able to type only “make” and the simulator will successfully compile. The TAs should be able to type only “make clean” to automatically remove object (.o) files and the simulator executable. Example Makefiles will be posted on the web page, which you can copy and modify for your needs.</li>

 <li>Your simulator must accept exactly 7 command-line arguments in the following order:</li>

</ol>




sim_cache   &lt;<em>BLOCKSIZE</em>&gt;

&lt;<em>L1_SIZE</em>&gt;  &lt;<em>L1_ASSOC</em>&gt;  &lt;<em>VC_NUM_BLOCKS</em>&gt;

&lt;<em>L2_SIZE</em>&gt;  &lt;<em>L2_ASSOC</em>&gt;

&lt;<em>trace_file</em>&gt;




<ul>

 <li><em>BLOCKSIZE</em>: Positive integer.  Block size in bytes.  (Same block size for all caches in the memory hierarchy.)</li>

 <li><em>L1_SIZE</em>: Positive integer.  L1 cache size in bytes. o <em>L1_ASSOC</em>:  Positive integer.  L1 set-associativity. (L1_ASSOC = 1 is directmapped.  L1_ASSOC = L1_SIZE/BLOCKSIZE simulates a fully-associative cache.)</li>

 <li><em>VC_NUM_BLOCKS</em>: Positive integer.  Number of blocks in the Victim Cache.</li>

</ul>

<em>VC_NUM_BLOCKS</em> = 0 signifies that there is no Victim Cache. o <em>L2_SIZE</em>:  Positive integer.  L2 cache size in bytes.  <em>L2_SIZE</em> = 0 signifies that there is no L2 cache.

<ul>

 <li><em>L2_ASSOC</em>: Positive integer.  L2 set-associativity. (L2_ASSOC = 1 is directmapped.  L2_ASSOC = L2_SIZE/BLOCKSIZE simulates a fully-associative cache.)</li>

 <li><em>trace_file</em>: Character string.  Full name of trace file including any extensions.</li>

</ul>




Example:  8KB 4-way set-associative L1 cache with 32B block size, 7-block victim cache affiliated with L1, 256KB 8-way set-associative L2 cache with 32B block size, gcc trace:

sim_cache  32  8192  4  7  262144  8  gcc_trace.txt




<ol start="4">

 <li>Your simulator must print outputs to the console (<em>e.</em>, to the screen). This way, when a TA runs your simulator, he/she/they can simply redirect the output of your simulator to a filename of his/her/their choosing for validating the results.</li>

</ol>

<h2>8.3. Keeping backups</h2>

It is good practice to frequently make backups of all your project files, including source code, your report, <em>etc</em>.  You can backup files to another hard drive (Eos account vs. home PC vs. laptop … keep consistent copies in multiple places) or removable media (Flash drive, <em>etc</em>.).  A private github repository is another option (not shared with anyone).

<h2>8.4. Run time of simulator</h2>

<em>Correctness</em> of your simulator is of paramount importance. That said, making your simulator <em>efficient</em> is also important for a couple of reasons.




First, the TAs need to test every student’s simulator. Therefore, we are placing the constraint that your simulator must finish a single run in 2 minutes or less. If your simulator takes longer than 2 minutes to finish a single run, please see the TAs as they may be able to help you speed up your simulator.




Second, you will be running many experiments: many memory hierarchy configurations and multiple traces. Therefore, you will benefit from implementing a simulator that is reasonably fast.




One simple thing you can do to make your simulator run faster is to compile it with a high optimization level. The example Makefile posted on the web page includes the –O3 optimization flag.




Note that, when you are debugging your simulator in a debugger (such as gdb), it is recommended that you compile without –O3 and with –g. Optimization includes register allocation. Often, register-allocated variables are not displayed properly in debuggers, which is why you want to disable optimization when using a debugger. The –g flag tells the compiler to include symbols (variable names, <em>etc</em>.) in the compiled binary. The debugger needs this information to recognize variable names, function names, line numbers in the source code, <em>etc</em>. When you are done debugging, recompile with –O3 and without –g, to get the most efficient simulator again.

<h2>8.5. Test your simulator on Eos linux machines</h2>

<u>You must test your simulator on Eos linux machines such as:</u> <em><u>remote.eos.ncsu.edu</u></em><u> and <em>grendel.ece.ncsu.edu</em>.</u>




<h1>9. Experiments and Report</h1>




<strong>Benchmarks </strong>




Use the GCC benchmark for all experiments.







<h2>Calculating AAT and Area</h2>




Table 1 gives names and descriptions of parameters and how to get these parameters.







<strong>Table 1. Parameters, descriptions, and how you obtain these parameters. </strong>

<table width="638">

 <tbody>

  <tr>

   <td width="91"><em>Parameter </em></td>

   <td width="264"><em>Description </em></td>

   <td width="283"><em>How to get parameter </em></td>

  </tr>

  <tr>

   <td width="91">SRR</td>

   <td width="264">Swap request rate.</td>

   <td rowspan="3" width="283">From your simulator. See Section 7.</td>

  </tr>

  <tr>

   <td width="91">MRL1+VC</td>

   <td width="264">Combined L1+VC miss rate.</td>

  </tr>

  <tr>

   <td width="91">MRL2</td>

   <td width="264">L2 miss rate (from standpoint of stalling the CPU).</td>

  </tr>

  <tr>

   <td width="91">HTL1</td>

   <td width="264">Hit time of L1.</td>

   <td rowspan="3" width="283">“Access time (ns)” from CACTI tool. A spreadsheet of CACTI results is available on the Project-1 website. Refer to sheet “regular-caches” for L1 and L2, and sheet “victim-caches” for VC.</td>

  </tr>

  <tr>

   <td width="91">HTVC</td>

   <td width="264">Hit time of VC.</td>

  </tr>

  <tr>

   <td width="91">HTL2</td>

   <td width="264">Hit time of L2.</td>

  </tr>

  <tr>

   <td width="91">Miss_Penalty</td>

   <td width="264">Time to fetch one block from main memory.</td>

   <td width="283">Refer to the spreadsheet of CACTI results available on the Project-1 website: sheet “sample-AATs”, column H “Miss_Penalty”.</td>

  </tr>

  <tr>

   <td width="91">AL1</td>

   <td width="264">Die area of L1.</td>

   <td rowspan="3" width="283">“Area (mm*mm)” from CACTI tool. A spreadsheet of CACTI results is available on the Project-1 website. Refer to sheet “regular-caches” for L1 and L2, and sheet “victim-caches” for VC.</td>

  </tr>

  <tr>

   <td width="91">AVC</td>

   <td width="264">Die area of VC.</td>

  </tr>

  <tr>

   <td width="91">AL2</td>

   <td width="264">Die area of L2.</td>

  </tr>

 </tbody>

</table>







For memory hierarchy <em>without</em> L2 cache:




Total access time <sub>=</sub> (L1 reads <sub>+</sub> L1 writes)<sub>⋅ </sub>HT<sub>L1 </sub>+ (swap requests)<sub>⋅ </sub>HT<sub>VC </sub>+ (L1 read misses <sub>+</sub> L1 write misses – swaps)<sub>⋅ </sub>Miss_Penalty




<table width="215">

 <tbody>

  <tr>

   <td rowspan="2" width="215">Average access time (AAT) = Total access time(L1 reads <sub>+</sub> L1 writes)</td>

   <td width="0"></td>

  </tr>

 </tbody>

</table>

AAT <sub>= </sub>HTL1 <sub>+</sub><sub> </sub>L1 swapreads requests<sub>+ </sub>L1writes<sub></sub><sub>⋅</sub>HTVC <sub>+</sub><sub> </sub><u>L1 read misses</u>L1 reads +<u> L1</u><sub>+</sub><u> write</u>L1 writes<u> misses – swaps</u><sub>⋅</sub>Miss_Penalty

<sub></sub>

= HT<sub>L1 </sub>+SRR ⋅HT<sub>VC </sub>+ MR<sub>L1</sub><sub>+</sub><sub>VC </sub>⋅Miss_Penalty







For memory hierarchy <em>with</em> L2 cache:




Total access time = (L1 reads + L1 writes)⋅ HT<sub>L1 </sub>+ (swap requests)⋅ HT<sub>VC </sub>+ (L1 read misses + L1 write misses – swaps)⋅ HT<sub>L2 </sub>+ (L2 read misses)⋅ Miss_Penalty




<table width="215">

 <tbody>

  <tr>

   <td rowspan="2" width="215">Average access time (AAT) = Total access time(L1 reads <sub>+</sub> L1 writes)</td>

   <td width="0"></td>

  </tr>

 </tbody>

</table>




AAT <sub>= </sub>HTL1 <sub>+ </sub><sub></sub> L1 swapreads requests<sub>+ </sub>L1writes<sub></sub><sub> ⋅ </sub>HTVC <sub>+ </sub> <u>L1 read misses</u>L1 reads <sup>+</sup><u> L1</u><sub>+</sub><u> write</u>L1 writes<u> misses – swaps</u><sub></sub><sub> ⋅ </sub>HTL2 <sub>+ </sub> L1 L2reads read<sub>+</sub> missesL1 writes<sub></sub><sub> ⋅ </sub>Miss_Penalty

= HTL1 + SRR ⋅ HTVC + MRL1+VC ⋅ HTL2 +  L1 L2reads read+ missesL1 writes ⋅ Miss_Penalty



= HTL1 + SRR ⋅ HTVC + MRL1+VC ⋅HTL2 +  MR1L1+VC  ⋅ L1 L2reads read+ missesL1 writes ⋅ Miss_Penalty

          

= HTL1 + SRR ⋅ HTVC + MRL1+VC ⋅HTL2 +  L1 read missesL1 reads + L1+ writeL1 writes misses – swaps ⋅ L1 L2reads read+ missesL1 writes ⋅ Miss_Penalty<sub></sub>

          

= HTL1 + SRR ⋅ HTVC + MRL1+VC ⋅HTL2 +  L1 read missesL2 + read L1 write misses misses – swaps ⋅ Miss_Penalty 

= HTL1 + SRR ⋅ HTVC + MRL1+VC ⋅HTL2 +  <u>L2 </u>L2<u>read</u> reads<u> misses</u> ⋅ Miss_Penalty



= HT<sub>L1 </sub>+ SRR ⋅ HT<sub>VC </sub>+ MR<sub>L1</sub><sub>+</sub><sub>VC </sub>⋅(HT<sub>L2 </sub>+ MR<sub>L2 </sub>⋅ Miss_Penalty)







The total area of the caches:

Area = A<sub>L1 </sub>+ A<sub>VC </sub>+ A<sub>L2</sub>

If a particular cache does not exist in the memory hierarchy configuration, then its area is 0.







<h2>9.1. L1 cache exploration: SIZE and ASSOC</h2>




<h3><u>GRAPH #1</u>   (total number of simulations: 55)</h3>




For this experiment:

<ul>

 <li>L1 cache: SIZE is varied, ASSOC is varied, BLOCKSIZE = 32.</li>

 <li>Victim Cache: None.</li>

 <li>L2 cache: None.</li>

</ul>




Plot L1 miss rate on the y-axis versus log<sub>2</sub>(SIZE) on the x-axis, for eleven different cache sizes: SIZE = 1KB, 2KB, … , 1MB, in powers-of-two. (That is, log<sub>2</sub>(SIZE) = 10, 11, …, 20.)  The graph should contain five separate curves (<em>i.e.</em>, lines connecting points), one for each of the following associativities: direct-mapped, 2-way set-associative, 4-way set-associative, 8-way setassociative, and fully-associative.  All points for direct-mapped caches should be connected with a line, all points for 2-way set-associative caches should be connected with a line, <em>etc</em>.




Discussion to include in your report:

<ol>

 <li>Discuss trends in the graph. For a given associativity, how does increasing cache size affect miss rate?  For a given cache size, what is the effect of increasing associativity?</li>

 <li>Estimate the <em>compulsory miss rate</em> from the graph.</li>

 <li>For each associativity, estimate the <em>conflict miss rate</em> from the graph.</li>

</ol>







<h3><u>GRAPH #2</u>   (no additional simulations with respect to GRAPH #1)</h3>




Same as GRAPH #1, but the y-axis should be AAT instead of L1 miss rate.




Discussion to include in your report:

<ol>

 <li>For a memory hierarchy with only an L1 cache and BLOCKSIZE = 32, which configuration yields the best (<em>i.e.</em>, lowest) AAT?</li>

</ol>







<h3><u>GRAPH #3</u>   (total number of simulations: 45)</h3>




Same as GRAPH #2, except make the following changes:

<ul>

 <li>Add the following L2 cache to the memory hierarchy: 512KB, 8-way set-associative, same block size as L1 cache.</li>

 <li>Vary the L1 cache size only between 1KB and 256KB (since L2 cache is 512KB).</li>

</ul>




Discussion to include in your report:

<ol>

 <li>With the L2 cache added to the system, which L1 cache configurations result in AATs close to the best AAT observed in GRAPH #2 (<em>g.</em>, within 5%)?</li>

 <li>With the L2 cache added to the system, which L1 cache configuration yields the best (<em>e.</em>, lowest) AAT? How much lower is this optimal AAT compared to the optimal AAT in GRAPH #2?</li>

 <li>Compare the <em>total area</em> required for the optimal-AAT configurations with L2 cache (GRAPH #3) versus without L2 cache (GRAPH #2).</li>

</ol>




<h2>9.2. L1 cache exploration: SIZE and BLOCKSIZE</h2>




<h3><u>GRAPH #4</u>   (total number of simulations: 24)</h3>




For this experiment:

<ul>

 <li>L1 cache: SIZE is varied, BLOCKSIZE is varied, ASSOC = 4.</li>

 <li>Victim Cache: None.</li>

 <li>L2 cache: None.</li>

</ul>




Plot L1 miss rate on the y-axis versus log<sub>2</sub>(BLOCKSIZE) on the x-axis, for four different block sizes: BLOCKSIZE = 16, 32, 64, and 128.  (That is, log<sub>2</sub>(BLOCKSIZE) = 4, 5, 6, and 7.)  The graph should contain six separate curves (<em>i.e.</em>, lines connecting points), one for each of the following L1 cache sizes: SIZE = 1KB, 2KB, …, 32KB, in powers-of-two.  All points for SIZE = 1KB should be connected with a line, all points for SIZE = 2KB should be connected with a line, <em>etc</em>.




Discussion to include in your report:

<ol>

 <li>Discuss trends in the graph. Do smaller caches prefer smaller or larger block sizes? Do larger caches prefer smaller or larger block sizes?  Why?  As block size is increased from 16 to 128, is the tradeoff between <em>exploiting more spatial locality</em> versus <em>increasing cache pollution</em> evident in the graph, and does the balance between these two factors shift with different cache sizes?</li>

</ol>




<h2>9.3. L1 + L2 co-exploration</h2>




<h3><u>GRAPH #5</u>   (total number of simulations: 44)</h3>




For this experiment:

<ul>

 <li>L1 cache: SIZE is varied, BLOCKSIZE = 32, ASSOC = 4.</li>

 <li>Victim Cache: None.</li>

 <li>L2 cache: SIZE is varied, BLOCKSIZE = 32, ASSOC = 8.</li>

</ul>




Plot AAT on the y-axis versus log<sub>2</sub>(L1 SIZE) on the x-axis, for nine different L1 cache sizes: L1

SIZE = 1KB, 2KB, … , 256KB, in powers-of-two. (That is, log<sub>2</sub>(L1 SIZE) = 10, 11, …, 18.)  The graph should contain six separate curves (<em>i.e.</em>, lines connecting points), one for each of the following L2 cache sizes: 32KB, 64KB, 128KB, 256KB, 512KB, 1MB.  All points for the 32KB L2 cache should be connected with a line, all points for the 64KB L2 cache should be connected with a line, etc.  <em>For a given curve / L2 cache size, only plot points for which the L1 cache is smaller than the L2 cache.  For example, the curve for L2 SIZE = 32KB should have only 5 points total, corresponding to L1 SIZE = 1KB through 16KB.  In contrast, the curve for L2 SIZE </em>

<em>= 512KB (and 1MB) will have 9 points total, corresponding to L1 SIZE = 1KB through 256KB.  </em>

<em>There should be a total of 44 points (5+6+7+8+9+9, for the six curves / L2 cache sizes, respectively). </em>







Discussion to include in your report:

<ol>

 <li>Which memory hierarchy configuration yields the best (<em>e.</em>, lowest) AAT?</li>

 <li>Which memory hierarchy configuration has the smallest total area, that yields an AAT within 5% of the best AAT?</li>

</ol>




<h2>9.4. Victim cache study (ECE 563 students only)</h2>




<h3><u>GRAPH #6</u>   (total number of simulations: 42)</h3>




For this experiment:

<ul>

 <li>L1 cache: SIZE is varied, ASSOC is varied, BLOCKSIZE = 32.</li>

 <li>Victim Cache: # entries is varied.</li>

 <li>L2 cache: SIZE = 64KB, ASSOC = 8, BLOCKSIZE = 32.</li>

</ul>




Plot AAT on the y-axis versus log<sub>2</sub>(L1 SIZE) on the x-axis, for six different L1 cache sizes: L1 SIZE = 1KB, 2KB, … , 32KB, in powers-of-two. (That is, log<sub>2</sub>(L1 SIZE) = 10, 11, …, 15.)  The graph should contain seven separate curves (<em>i.e.</em>, lines connecting points), one for each of the following scenarios:

<ul>

 <li>Direct-mapped L1 cache with no Victim Cache.</li>

 <li>Direct-mapped L1 cache with 2-entry Victim Cache.</li>

 <li>Direct-mapped L1 cache with 4-entry Victim Cache.</li>

 <li>Direct-mapped L1 cache with 8-entry Victim Cache.</li>

 <li>Direct-mapped L1 cache with 16-entry Victim Cache.</li>

 <li>2-way set-associative L1 cache with no Victim Cache.</li>

 <li>4-way set-associative L1 cache with no Victim Cache.</li>

</ul>




Discussion to include in your report:

<ol>

 <li>Discuss trends in the graph. Does adding a Victim Cache to a direct-mapped L1 cache yield performance comparable to a 2-way set-associative L1 cache of the same size?  …for which L1 cache sizes?  …for how many Victim Cache entries?</li>

 <li>Which memory hierarchy configuration yields the best (<em>e.</em>, lowest) AAT?</li>

 <li>Which memory hierarchy configuration has the smallest total area, that yields an AAT within 5% of the best AAT?</li>

</ol>




<h1>10. What to Submit via Moodle</h1>

You must hand in a single zip file called <strong>project1.zip</strong> that is no more than 1MB in size. (Notify the TAs beforehand if you have special space requirements. However, a zip file of 1MB should be sufficient.)




Below is an example showing how to create <strong>project1.zip</strong> from an Eos Linux machine. Suppose you have a bunch of source code files (*.cc, *.h), the Makefile, and your project report (report.pdf).

zip  <strong>project1</strong>  *.cc  *.h  Makefile  report.pdf




<strong><u>project1.zip</u></strong><u> must contain the following (any deviation from the following requirements may</u> <u>delay grading your project and may result in point deductions, late penalties, <em>etc</em>.):</u>




<ol>

 <li><strong>Project report</strong>. This must be a single PDF document named <strong>pdf</strong>. The report must include the following:

  <ul>

   <li>A cover page with the project title, the Honor Pledge, and your full name as electronic signature of the Honor Pledge. A sample cover page will be posted on the project website.</li>

   <li>See Section 9 for the required content of the report.</li>

  </ul></li>

 <li><strong>Source code</strong>. You must include the commented source code for the simulator program itself. You may use any number of .cc/.h files, .c/.h files, <em>etc</em>.</li>

 <li><strong>Makefile</strong>. See Section 8.2, item #2, for strict requirements. If you fail to meet these requirements, it may delay grading your project and may result in point deductions.</li>

</ol>