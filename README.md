# LaminarIR

LaminarIR is a lower-level intermediate representation (IR) for stream
programming that expresses semantics of actors and data channels 
between actors. In contrast to prior work, tokens on data channels 
are named, which enables a compiler to shift data channel management 
from run-time to compile-time. 

The compiler for LaminarIR is written in Python and generates C code
for a given LaminarIR program. Currently, LaminarIR has only a single 
front-end that translates StreamIt to LaminarIR.  

This code repository contains the source code of the LaminarIR compiler,
the adapted StreamIt frontend to translate StreamIt to LaminarIR, and 
a set of StreamIt benchmarks for performance evaluation.

The LaminarIR framework is open source software distributed under the terms of
the license agreement found in LICENSE.txt.

# CONTACTS

 * Yousun Ko           <yousun.ko@yonsei.ac.kr>
 * Bernd Burgstaller   <bburg@yonsei.ac.kr>
 * Bernhard Scholz     <Bernhard.Scholz@sydney.edu.au>

# INSTALLATION

The LaminarIR framework requires a Linux system that has installed the following 
software packages (stated versions are known to work):

 * JDK       1.7.0
 * Python    2.7.6
 * pygraph   1.8.2
 * StreamIt  github.r8592
 * Clang & LLVM  3.5.0, 3.7.0
 * [LIKWID](https://code.google.com/p/likwid/)    3.1.2, 3.1.3
 * [PAPI](http://icl.cs.utk.edu/papi/)      5.2.0, 5.3.2
 * ANTLR v3.1 (3.1, 3.1.1, 3.1.2)

JDK is needed for the parser generator of LaminarIR and the execution of 
StreamIt. Python and pygraph are required for the LaminarIR compiler. 
StreamIt is required to evaluate StreamIt for benchmarking.
Clang and LLVM are required to compile C codes and generate 
compiler optimization statistics.  LIKWID and PAPI are required for
performance measurements.  LIKWID is used to pin threads to specific cores,
and PAPI provides a wrapper to access hardware performance counters.  ANTLR
v3.1 is required to generate LaminarIR parser. Please download the ANTLR v.3.1 jar
file and locate under src/bin.

The LaminarIR compiler needs to be parameterised for the underlying 
hardware architecture. This setting is kept in `LaminarIR/framework/configure`.
Set the `MACHINE` variable to match the underlying host CPU.
Then run the configure script:

 ```
 ./configure
 ```

This process will generate Defines.make which defines machine specific 
compilation settings for code compliation and compile the whole framework.


# HOW TO RUN THE FRAMEWORK

The framework provides a python script `LaminarIR/framework/autorun.py`, to 
(re)configure the system, compile and execute each benchmarks, and collect 
the measured results at once. Running `autorun.py` will invoke each steps 
below in order. 

1. Configure current processor architecture setting given in `configure` 
and set the compiler options accordingly.

2. Generate LaminarIR direct access format or FIFO queues from a 
   high-level stream programming language. The LaminarIR frontend for StreamIt 
   is provided which reads StreamIt source codes from 
   `LaminarIR/framework/examples/streamit/strs`.

3. Generate C code on LaminarIR direct access format or FIFO queues by 
   reading LaminarIR codes under the benchmark directories e.g.,
   `LaminarIR/framework/examples/laminarir/direct` (or read generated C++ codes
   by StreamIt-2.1.1 compiler stored under 
   `LaminarIR/framework/examples/streamit/instrumented`.)

4. Compile and run the obtained C/C++ code with clang.

5. Collect the measured performance and append to the performance measurements to
   `LaminarIR/framework/papi_events.csv`

6. Backup all the by-product codes and measured performance under 
   corresponding directory under `LaminarIR/framework/results` e.g., 
   `LaminarIR/framework/results/laminar_direct` or 
   `LaminarIR/framework/results/streamit`.

The file `LaminarIR/framework/report.log` contains the logs of the executed commands invoked 
by the `autorun.py` script.

# Benchmarks

The LaminarIR compiler provides two code generators: the `direct memory access` 
backend performs the data channel management at compiletime , and the `FIFO queues` 
backend uses FIFO buffers for data channels. All benchmarks from the 
experimental evaluation of the conference paper are provided, i.e., 
DCT, DES, FFT2, MatrixMult, AutoCor, Lattice, Serpent, JPEGFeed, BeamFormer,
ComparisonCounting, and RadixSort.
LaminarIR code in direct token-access is provided for each benchmark in,
  `LaminarIR/framework/examples/laminarir/direct`. 
LaminarIR code employing FIFO queues is provided for each benchmark in,
  `LaminarIR/framework/examples/laminarir/fifo`.
The C++ code of each benchmark as generated by StreamIt-2.1.1 compiler is found 
under,
  `LaminarIR/framework/examples/streamit/instrumented`

For each benchmark, two versions are provided: the original StreamIt 
benchmark code, plus one version that we created for using randomized input
(the original StreamIt benchmarks use (mostly) static input). A file name
`<name>.rand.*` denotes code with randomized input, and `<name>.*` denotes
code with static input.

## EXPERIMENTAL OPTIONS

### Generating LaminarIR codes

Running following commands in the directory `LaminarIR/framework/frontend` will generate the
LaminarIR direct-access format (or FIFO queues with `--laminar fifo` option) 
with static inputs (or random inputs with `--random-input`) in the directory
`LaminarIR/framework/frontend/compile` by reading StreamIt codes from 
`LaminarIR/framework/examples/streamit/strs`.

```
python LaminarIRGen.py --laminar direct --random-input DCT
```

### Performance Evaluation

Running the command,
```
  python autorun.py --laminar direct --laminar fifo --streamit
```
in the directory `LaminarIR/framework` will evaluate LaminarIR
with direct-access, FIFO queues, and StreamIt over all benchmarks and
generate a `papi_events.csv`. The CSV file contains:
execution times, clock cycles per introduction, number of loads and stores,
energy consumption of each CPU of each benchmark if the corresponding
hardware performance counters are provided by the machine that the framework
runs on.

A specific benchmark is executed by the command by adding its name to the command line, e.g., 
```
  python autorun.py --laminar direct --laminar fifo --streamit DCT
```

Instead of both code generators (direct and fifo) only a single one can be specified, e.g.,
``` 
  python autorun.py --laminar direct --streamit DCT
```
will run DCT with the LaminarIR direct generator and StreamIt only.

The command-line option `--generate-LaminarIR=streamit` is used to generate the LaminarIR code 
from StreamIt fronted for LaminarIR. If not specified, the framework will 
read LaminarIR codes from the directory `LaminarIR/framework/examples/laminarir`.

The command-line options `--static-input` or `--random-input` determines whether the input is 
either static or randomly generated. The default option is `--random-input`.

The command-line option `-O` or `--opt-test` determines LLVM optimization passes,  e.g.,
```
 python autorun.py --laminar direct --laminar fifo --streamit --opt-test DCT
```
Note that this option may take several hours for a benchmark.

The command-line option `-a` or `--llvm_analyzer` produces a LLVM compiler statistics e.g., 
```
  python autorun.py --laminar direct --laminar fifo --streamit --llvm_analyzer
```
and will generate `<name>.O3.stats` in addition which contains `Number of 
allocas promoted to SSA values`.

### Structural Statistics 

The command 

```
  python laminar.py --elim_comm examples/laminarir/direct/DCT.sdf
```
executed in the directory `LaminarIR/framework` will compute the total communication 
in bytes and the eliminated communication in byte of DCT along with numbers of 
actors and split-joins  etc. Note that the full path to the code should be 
given unlike the other cases.
