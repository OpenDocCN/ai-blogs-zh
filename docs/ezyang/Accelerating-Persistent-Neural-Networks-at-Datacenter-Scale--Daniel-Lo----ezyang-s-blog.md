<!--yml
category: 未分类
date: 2024-07-01 18:16:59
-->

# Accelerating Persistent Neural Networks at Datacenter Scale (Daniel Lo) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2017/12/accelerating-persistent-neural-networks-at-datacenter-scale-daniel-lo/](http://blog.ezyang.com/2017/12/accelerating-persistent-neural-networks-at-datacenter-scale-daniel-lo/)

The below is a transcript of a talk by [Daniel Lo](https://www.microsoft.com/en-us/research/people/dlo/) on [BrainWave](https://www.microsoft.com/en-us/research/blog/microsoft-unveils-project-brainwave/), at the [ML Systems Workshop](https://nips.cc/Conferences/2017/Schedule?showEvent=8774) at NIPS'17.

* * *

Deploy and serve accelerated DNNs at cloud scale. As we've seen, DNNs have enabled amazing applications. Architectures achieve SoTA on computer vision, language translation and speech recognition. But this is challenging to serve in large-scale interactive because there are latency, cost and power constraints. Also, DNNs are growing larger in size and complexity.

We've seen a Cambrian explosion in startups to solve this problem. Research groups have produced DNN processing units, DPUs, custom hardware solutions to prove high throughput efficient serving of DNNs. We categorize them into two categories: fast DPUs, where the algorithms and applications have to be fixed in at design time, because they're fabbing an ASIC, or a soft DPU, FPGA. But for soft DPUs, we haven't seen them deployed at scale.

To address this, we've been working on Project BrainWave. Solution to deploy large scale DNNs with FPGA-acceleration. We've designed it to be fast, flexible and friendly. High throughput, low latency acceleration using FPGAs. Flexibility with adaptive numerical precision, update to latest AI algorithms with reconfigurable FPGAs. And it's user friendly, because we have a full stack solution, compile CNTK/Caffe/TF and compile them down. This is deployed on our configurable cloud, an outer layer of CPUs, a data center that puts everything together, and a layer of reconfigurable FPGAs.

We've been deployed DNN models. LSTM model that takes tens to hundreds of milliseconds CPU. What we see is the 99th percentile for latency; even at 99 we are able to achieve sub-millisecond latencies. When you get to these levels of acceleration, it's negligible in the E2E pipeline.

Next I'll dive into details. It's a full stack solution. starting with a compiler and runtime that takes model sin high level frameworks and compiles them down to our architecture. A flexible ISA for serving DNNs. We have a throughput, low latency serving. We do this all with persistency at scale, to keep models pinned in FPGA memories. Deployed on our wide deployment of Intel FPGAs using hardware microservices.

To begin with, let's talk about hardware microservices. This is something we presented at Micro. The architecture of reconfigurable cloud is FPGAs sit between CPU and network. CPU can use FPGA locally for acceleration, but because FPGAs are connected over network, they can distribute between them. We have a proprietary network protocol for low latency compute.

We'vec disaggregated FPGA compute plane from CPU. So we can aggregate FPGAs together to form larger accelerators, and you don't have to match the rate of FPGAs to CPUs. You can serve a large number of CPUs with a small cluster of FPGAs, or vice versa.

Next I'll talk about the compiler and runtime. Goal is to make it very easy for ML specialists to do this. The typical ML specialist doesn't know how to program this. Models developed in high level frameworks, compile them down to our architecture. If you compile them down first into an intermediate graph based representation. We split them into portions split on FPGAs, and portions on CPU. When we execute, we also have runtime that handles orchestration and scheduling that handles it between parts.

There are two main categories of DNNs we have to optimize for. DNNs that have very high compute to data ratio, convnets, these are well studied. I'm going to focus on the other class of DNNs, those with less compute to data ratio, e.g. dense layers and RNNs.

The conventional approach to accelerating DNNs on FPGAs, you keep all model parameters in DRAM. When a request comes in, you're going to stream the model parameters of DRAM, and return a request. The issue with this is when you have DNN layers that are memory bandwidth bound, you're limited in how fast you can run this by memory bandwidth; you're not getting full compute capabilities of FPGA. Typically the way to solve this is with batching; you send a number of requests and use the model parameters for all requests. WHile you may achieve good throughput, latency will increase. For realtime services, this violates your SLA. What we want to do is provide high performance at low or no batching.

The way we do this is with persisted Dnets. FPGAs have lots of memory on chip: 10MB memory. Since they're on chip, it's high bandwidth. So we're going to keep the model parameters on the chip, so that when we get one request in, we distribute it across the entire FPGA chip.

The obvious question is, what happens if your model doesn't fit on chip? We take advantage of the hardware microcenter. We'll distribute a single model over multiple FPGAs in the datacenter.

Let's look at the architecture and microarchitecture of the processing unit we developed. The BrainWave DPU is a software programmable processor, programmed in single-threaded C, but we've added a number of instructions for serving DNNs, e.g., matrix multiply, convolution, nonlinear activations, embeddings. The processor is designed to use narrow precision format (float16) and easily flexible for extending to newer algorithms.

The microarchitecture of the processor, main portion is dedicated to matrix vector unit; matrix vector multiply, consisting of a number kernels on a tile of a larger matrix. Tiling gives us flexibility while maintaining performance. Other compute units are multifunction units; vector-vector operations, such as element-wise multiply, add and activation functions. Tying it all together is an on-chip network that lets us keep all the compute together at time.

Most of the chip is dedicated to matrix vector unit. It's composed of hundreds of multilane dot product units. Each of these dot product units is consists of tens of adds and muls. To keep them fed with data, each dot product unit is fed by a set of dedicated block rams.

Next, I'd like to show performance results for this architecture. Two years ago, we had a deployment of Stratix V FPGAs. It shows the effective teraflops of this format. 16 bit integer.. we've been playing with our own format Microsoft Floating Point. 4.5Tflops at MSFP5.8\. These Stratix are pretty old.

(Demo for latest generation of FPGAs)

Looking at throughput oriented DPU, the latency is 65.81ms. With brainwave, latency is 0.98ms. Under 1 millisecond.

This was done on initial engineering silicon. For production silicon, we're expecting to get 12TOps at 16-bit integer. 90TOps for MSFP8\. One question is how does numeric output affects output. Here is the normalized accuracy for three in-house text models, using GRU and LSTM. The orange bar shows what happens when you go to MSFP9, but we've developed a way to fine tune networks for this precision, and you see we recover our accuracy. We're working with MSFP8 and see similar results.

Project BrainWave is our project for accelerating DNNs at cloud scale. We hope it will be fast, friendly and cloud-scale, and expand capabilities of AI in the cloud, providing a way to run higher dimensional RNN networks for NLP and other great applications. We're planning to release to third parties, stay tuned.

Q: When you decrease batch size, what hardware are you evaluating? Hardware utilization as we decrease?

A: We stay highly utilized even as we decrease batch size; even at high batch size, we're still sending requests one by one. (Only one step will be processed?) Right.

Q: Regarding the FP9 and FP8, nine and eight being the number of bits used? (Yes) Is it in any way related to Flexpoint at Intel?

A: We developed this independently of flexpoint, and I'm not able to talk about our numeric format.

Q: In MS, do you really write Verilog for your FPGA, or do you use high level synthesis tool?

A: For this, we are writing System Verilog

Q: Batchnorm layers, which require batch computation; how do you put that onto the FPGA?

A: Part of the work of the compiler is to do splitting between CPU and FPGA. So things that are not amenable to FPGA, including batchnorm, we're still running them on CPU.