# ChunkGraph: Efficient Large Graph Processing with Chunk-Based Graph Representation Model

This respository is for following paper:
**ATC'24 paper:** Efficient Large Graph Processing with Chunk-Based Graph Representation Model.
Our implementation ChunkGraph is based on the Ligra system, which is a high-performance graph processing system. 
You can refer to the [Ligra repository](https://github.com/jshun/ligra) for more details: 


## Abstract

Existing external graph processing systems face challenges
in terms of low I/O efficiency, expensive computation overhead,
and high development costs when running on emerging
NVMe SSDs. Due to their reliance on complex loading and
computing models that aim to convert numerous random I/Os
into a few sequential I/Os. While in-memory graph systems
working with memory-storage cache systems like OS page
cache or TriCache, offer a promising solution for large graph
processing with fine-grained I/Os and easy algorithm programming,
they often overlook the specific characteristics of
the graph, resulting in inefficient graph processing. To address
these challenges, we introduce ChunkGraph, an I/O-efficient
graph system designed for processing large-scale graphs on
NVMe SSDs. ChunkGraph introduces a novel chunk-based
graph representation model, featuring classified and hierarchical
vertex storage and efficient chunk layout optimization.
Evaluations show that ChunkGraph can outperform existing
external graph systems, as well as in-memory graph systems
relying on general cache systems, running several times faster.

## Getting Started Instructions

### prequisites
Our system is based on Ubuntu 20.04.6 LTS with Linux kernel of 5.4.0, and it requires g++ >= 5.3.0 with OpenMP.


### Installation
```bash
## get the source code
$ git clone 
$ cd ChunkGraph
## Make executable file for different graph processing algorithms
$ cd apps
$ export CHUNK=1
$ make
```

### Prepare the dataset
ChunkGraph provides interfaces to ingest graph data from chunk-format files of binary format. 
Typically, the downloaded datasets are edge list files of text format. 
For convenience, we provide a script to convert the input data from text format to csr-based binary and 
chunk-format binary. 

**Example: LiveJournal (small graph for function test):**
```bash
# Download dataset and unzip
$ export ChunkGraph=$PWD
$ mkdir Dataset && cd Dataset
$ mkdir LiveJournal && cd LiveJournal
$ mkdir txt && cd txt
$ wget https://snap.stanford.edu/data/soc-LiveJournal1.txt.gz
$ gunzip soc-LiveJournal1.txt.gz
$ mv soc-LiveJournal1.txt livejournal.txt

## convert to csr format
$ cd $ChunkGraph
$ bash ./preprocess/text2csr.sh Dataset/LiveJournal/ livejournal
## convert to chunk format
$ mkdir ./Dataset/LiveJournal/chunk
$ bash ./preprocess/csr2chunk.sh Dataset/LiveJournal/csr_bin/ livejournal Dataset/LiveJournal/chunk/
```

### Running
**Example: Run BFS algorithm in LiveJournal dataset by ChunkGraph.**

```bash
$ cd ./apps
$ ./BFS -b -chunk -r 12 -t 48 $ChunkGraph/Dataset/LiveJournal/chunk/livejournal
```

The key statistic data for graph querying would be recorded to `chunkgraph_query_time.csv` file, e.g.
```bash
#[algo],[dataset_path],query_time
[./BFS],[/Dataset/LiveJournal/chunk/livejournal@buffer:0],2.85
```

## Detailed Instructions

### Overview
This artifact provides the source code of ChunkGraph, a graph processing system designed for processing large-scale graphs on NVMe SSDs. It also provides the source code of Blaze and Ligra-mmap for comparison. The experiments include graph query performance for ChunkGraph, Blaze and Ligra-mmap on six graphs (Figure 8), I/O overhead analysis (Figure 9), and computation overhead analysis (Figure 10) in the paper.

### Hardware dependencies
This artifact runs on a server with two processors configured in a non-uniform memory access (NUMA) architecture. 
Each processor has 24 physical cores with hyper-threading enabled (48 logical cores). 
For memory, it equips with 8 × 16 GB (128 GB) DRAM.
For storage, it equips with two 4 PCIe-attached Intel P5520 NVMe SSDs.

### Software dependencies
This artifact runs on Ubuntu 20.04.6 LTS with Linux kernel of 5.4.0, and we use GCC 10.5.0 with -O3 optimization for evaluation. 
Additionally, some extra libraries may need to be installed, such as NUMA, ZLib development libraries, etc. 

### Installation
Users need to download the source code and scripts from Zenodo to the server. 
The following is the directory structure of the source code, scripts, and instructions:
* README.md: This file contains a detailed step-by-step "Getting Started Instructions" guide, and "Detailed Instructions" for running the experiments. 
* src: This directory contains the source code of ChunkGraph and Ligra-mmap.
* CSRGraph: This directory contains the source code of Chunk Layout Optimization of ChunkGraph. 
* apps: This directory contains contains the graph query algorithms for ChunkGraph and Ligra-mmap.
* blaze: This directory contains the source code of Blaze.
* scripts: This directory contains the scripts for running the experiments.
* preprocess: This directory contains the scripts for converting the input data from text format to csr-based binary and chunk-format binary.


After downloading the source code and scripts, users need to compile ChunkGraph and prepare data sets. 
To evaluate the performance of comparison systems, users need to compile Blaze and Ligra-mmap. 

### Dataset preparing
We use four real-world graphs Twitter, Friendster, UKdomain and Yahoo Web, 
as well as two synthetic Kronecker graphs, i.e., Kron-29 and Kron-30, 
which are generated by graph500 generator, for our evaluation. 
Datasets download links: 
* **Twitter:** https://anlab-kaist.github.io/traces/WWW2010
* **Friendster:** http://konect.uni-koblenz.de/networks/friendster
* **UKdomain:** http://konect.cc/networks/dimacs10-uk-2007-05
* **Yahoo Web:** http://webscope.sandbox.yahoo.com

Graph500 generator link and Kronecker graph generation commands:
* **Generator link:** https://github.com/rwang067/graph500-3.0
* **Make genetator:** cd graph500-3.0/src \&\& make graph500\_reference\_bfs
* **Generate Kron29:** ./graph500\_reference\_bfs 29 16 kron29.txt
* **Generate Kron30:** ./graph500\_reference\_bfs 30 16 kron30

**Example1: Friendster (Medium graph, one of our evaluation tested dataset):**
```bash
## Download and unzip
$ export ChunkGraph=$PWD
$ mkdir Dataset && cd Dataset
$ mkdir Friendster && cd Friendster
$ mkdir txt && cd txt
$ wget http://konect.cc/files/download.tsv.friendster.tar.bz2 
$ tar -jxvf friendster.tar.bz2
$ mv out.friendster friendster.txt

## convert to csr format
$ cd $ChunkGraph
$ bash ./preprocess/text2csr.sh ./Dataset/Friendster/ friendster
## convert to chunk format
$ mkdir ./Dataset/Friendster/chunk
$ bash ./preprocess/csr2chunk.sh ./Dataset/Friendster/csr_bin/ friendster ./Dataset/Friendster/chunk/
```

**Example2: Kron30 (Largest synthetic graph generated by graph500 generator):**
```bash
## Build graph500 
## A valid MPI-3 library is required to compile this project
$ git clone https://github.com/rwang067/graph500-3.0
$ cd graph500-3.0/src
$ make graph500_reference_bfs 

## Generate Kron30
$ export ChunkGraph=$PWD
$ mkdir Dataset && cd Dataset
$ mkdir Kron30 && mkdir Kron30/txt
$ ./graph500_reference_bfs 30 16 Kron30/txt/kron30.txt

## convert to csr format
$ cd $ChunkGraph
$ bash ./preprocess/text2csr.sh ./Dataset/Kron30/ kron30
## convert to chunk format
$ mkdir ./Dataset/Kron30/chunk
$ bash ./preprocess/csr2chunk.sh ./Dataset/Kron30/csr_bin/ kron30 ./Dataset/Kron30/chunk/
```

### Compilers
* g++ >= 5.3.0 with OpenMP.

### Compiling 

Using the following commands, one can easily compile the `ChunkGraph`. 
Different graph processing algorithms can be compiled by the `Makefile` in the `apps` directory.

```bash
## Make executable file for different graph processing algorithms
$ cd apps
$ export CHUNK=1
$ make
```

### Running

**Arguments setup:**
You can set up the arguments by using following command-line options.

```bash
./apps/[algo] [OPTION] [DATASET_PATH]
./exe options.
 -chunk: indicates input a chunk graph.
 -s: indicates input a symmetric graph.
 -c: indicates input a compressed graph.
 -b: indicates input a binary graph.
 -t: specifies the number of threads.
 -rounds: specifies the number of rounds for the algorithm.
 -buffer: specifies the buffer size for ChunkGraph.
 -threshold: specifies the threshold used by top-down/bottom-up selection.
```

### Experiment workflow
The experiment workflow includes graph query performance for ChunkGraph, Blaze and Ligra-mmap on six graphs (Figure 8), I/O overhead analysis (Figure 9), and computation overhead analysis (Figure 10) in the paper.
The suggested workflow is organized in scripts/, which includes all the scripts for running the experiments.
Users can run the experiments by `bash scripts/run.sh`. 
The running progress and the expected completing time of each experiment would be printed to the file `scripts/progress.txt` automatically.
Note that you may have to change some arguments according to your environment, 
such as dataset path, taskset-cpu list, and the number of threads.

The evaluation results would be generated to the directory `results/`.
Users can reproduce the results in Figure 8, Figure 9, and Figure 10, 
which should roughly match the respective figures from the paper. 


### Comparison Systems

**Ligra-mmap**

We provide the mmap version of Ligra for comparison. 
Ligra-mmap ingests graph data from csr-format files of binary format. 
Compile and run the Ligra-mmap by the following commands.

```bash
# compile Ligra-mmap
$ cd apps && make
# run BFS on Twitter dataset
$ ./BFS -b -r 12 /Dataset/Twitter/twitter
```

**Blaze**

Blaze is a graph processing system designed for SSDs. 
We use the Blaze system for comparison. 
Blaze ingests graph data from customized csr-format files of binary format. 
You can refer the [Blaze repository](https://github.com/NVSL/blaze) for more details. 
Compile and run the Blaze by the following commands.


```bash
$ cd blaze
# compile Blaze
$ mkdir -p build && cd build && cmake .. && make -j
# run BFS on Twitter dataset
$ ./bin/bfs -computeWorkers 16 -startNode 12 /datapath/to/blaze/twitter/twitter.gr.index /datapath/to/blaze/twitter/twitter.gr.adj.0
# run BC on Twitter dataset
$ ./bin/bc -computeWorkers 16 -startNode 12 /datapath/to/blaze/twitter/twitter.gr.index /datapath/to/blaze/twitter/twitter.gr.adj.0 -inIndexFilename /datapath/to/blaze/twitter/twitter.tgr.index -inAdjFilenames /datapath/to/blaze/twitter/twitter.tgr.adj.0
```
