<!--yml
category: 未分类
date: 2024-07-01 18:16:59
-->

# Ray: A Distributed Execution Framework for Emerging AI Applications (Ion Stoica) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2017/12/ray-a-distributed-execution-framework-for-emerging-ai-applications-ion-stoica/](http://blog.ezyang.com/2017/12/ray-a-distributed-execution-framework-for-emerging-ai-applications-ion-stoica/)

The below is a transcript of a talk by [Ion Stoica](https://people.eecs.berkeley.edu/~istoica/) on [Ray](https://github.com/ray-project/ray), at the [ML Systems Workshop](https://nips.cc/Conferences/2017/Schedule?showEvent=8774) at NIPS'17.

* * *

We've been working on it at Berkeley for more than one year. Over the past years, there's been tremendous progress in AI. Ad targeting, image&speech, many more. Many applications are based on supervised learning with DNNs. Supervised plus unsupervised are the two dominant approaches.

However, the next generation of AI applications will be very different. They're deployed in mission critical scenarios, need to continually learn from a rapidly changing env. Robotics, self driving cars, unmanned drones, dialogue systems. Implementing this new generation of AI applications requires a broader range of techniques. Stochastic optimization, parallel simulations, many more.

Ray provides a unified platform for implementing these approaches. To motivate Ray, I'll use reinforcement learning. RL learns by interacting with env. A policy mapping from state/observation to action that maximizes a certain reward. What are the reqs of RL? Many applications exhibit nested parallelism: search, where they use data parallel SGD, which then calls a component that does policy evaluation with a model to simulate, that runs in parallel on multiple CPUs. Second, these workloads can be highly heterogenous in hardware and time. Many of these computations require not only CPUs, but GPUs TPUs and FPGAs. Second, this computation can take wildly different times. Simulate a chess game: 3 moves to lose, or 50 moves to win or draw. And in robotics, we need to process in real time, processing the data from sensors in parallel, tens of ms.

Meeting these requirements is not easy. To meet these requirements, you need a system that is flexible and performant. Flexible: it should create and schedule tasks dynamically, and support arbitrary dependencies. Perf: it should scale to hundreds of nodes, sub-millisecond latency, millions of task, and efficiently share numeric data.

Next, I'm going to say how we achieve these challenges. Flexibility? We provide a very flexible model: dynamic tasks graphs. On top of this, we give the two models: parallel tasks and actors.

To talk about parallel tasks, here is Python code: one reads an array from a file, and the other adds two arrays. The code is simple: it creates two arrays a and b from file1 and file2, and sum them up. So now, parallelizing this program is quite easy. If we want to parallelize a function, in order to do that, we need to add a ray.remote decorator to each function. When we invoke these functions, you need to invoke remote method. Remove doesn't return object itself, just the object id. This is very similar to the futures abstraction. To get the actual object, you must invoke ray.get on the object id.

To get a better idea of how Ray is executing, let's execute a simple program. Assumes files stored on different nodes. When read_array on file1, it schedules read_array on the appropriate node. The remote call returns immediately, before the actual read finishes. This allows the driver to run the second task in parallel, running on the node on file 2, and launch the add remote function. All functions have been scheduled remotely, but none of them have finished. To actually get the result, you have to call ray.get on the result. This is a blocking call, you'll wait for the entire computation graph to be executed.

Tasks are very general, but they are not enough. Consider that you want to run a simulator, and this simulator is closed source. In this case, you do not have access to the state. You have state, action, simulations, to set up state in simulator, you cannot do it. So to get around this, there is another use case, where the state is too expensive to create. For example, DNNs on GPUs, in this case, you want to initialize it once, and reinitialize for each simulation.

In order to address these use cases, we add Actor abstraction. An actor is just a remote class. If you have a Counter, you mark it ray.remote, and the when you create the class or invoke methods, you use remote keyword. This is a computation graph for this very simple example. Notice the method invocations also return object identifiers. To get the results, you need to call ray.get on object identifiers. Ray also allows you to specify the number of resources, for actors and tasks.

To put things together, and provide a more realistic example, evaluation strategy, a scalable form of RL, by Salimans et al in OpenAI. In a nutshell, evolution strategy, tries lots of policies, and tries to see which runs best. This is highly parallel. So here is pseudocode for parallel strategies. A worker that does simulation and returns the reward, create twenty workers, and then 200, do 200 simulations, update policy. Again, if you want to parallelize this code, we have to add a bunch of remote, and now on the right hand side, you'll notice I'm also sharing the computation graph. When you invoke now, the Worker.remote, you create 20 remote workers to do it in parallel. And you invoke with the remote keyword. Again, notice that in this case, the results are not the rewards themselves, but they're ids to the reward objects. In order to get the rewards to get policy, you have to call ray.get.

This hopefully gives you a flavor how to program in Ray. Next time, I switch gears, presents system design of Ray; how Ray gets high performance and scalability.

Like many classic computing frameworks, it has a driver, and a bunch of workers. Driver runs a program, worker runs task remotely. You can run and write a bunch of actors. The drivers actors on the same node, they share the data, on shared memory, and the workers and actors of cross nodes, share through distributed object store we built. Each node has a local scheduler, so when a driver wants to run another task, the local scheduler tries to schedule it locally. If it cannot schedule it locally, it invokes global scheduler, and it will schedule another node that has resources. Actor, remote method. Finally, what we do, and one essential part of the design, is we have a Global Control State. It takes all of the state of the system, and centralizes it. The metadata for the objects, in objects table, function. This allows system to be stateless. All these other components can fail, you can bring them up, get the most recent data from global control state. It also allows us to parallelize the global scheduler, because these replicas are going to share the same state in the GCS.

Another nice effect of having a GCS is that it makes it easy to build a bunch of profiling and debugging tools.

This design is highly scalable. Let me try to convince you why this is. To make GcS scalable, we just shard it. All these keys are pseudorandom, so it's easy to shard and load balance. The scheduler as you see is distributed; each node has a local scheduler, and Ray tries to schedule tasks which are spawned by a worker/driver on another task that is locally. The global scheduler, becomes a bottleneck, we can also replicate it. Finally, in systems, even if scheduler is super scalable, in Spark, there's another bottleneck: only the driver can launch new tasks. In order to get around that, we allow in Ray the workers and actors to launch tasks. Really, there is no single bottleneck point.

A few words about implementation. The GCS is implemented with Redis. For object store, we leverage Apache Arrow. For fault tolerance, we use lineage based fault tolerance like Spark. Actors are part of task graph; methods are treated as tasks, so we have a uniform model for providing fault tolerance.

So now some evaluation results. This plot represents the number of tasks per second, and you can see the number of nodes; it scales linearly. You can schedule over 1.8 M/s. Latency of local task execution is 300us, the latency of remote task is 1ms. This plot illustrates fault tolerance. You may ask why you care about fault tolerance? The problem is you need in your program that the simulation may not finish; this makes the program far more complicated, even if you're willing to ignore some results. Here, on this axis, you have the time in seconds, you have two y axes, number of nodes in system, and the throughput. As you can see, the number of nodes is starting at 50, then 25, then to 10, and goes back to 50\. In the red area, you show the number of tasks per second; it follows as you may expect, the number of nodes in the system. If you look a little bit, there are some drops; every time, you have a drop in the number of tasks. It turns out this is because of the object reconstruction. When some nodes go away, you lose the objects on the node, so you have to reconstruct them. Ray and Spark reconstruct them transparently. With blue, you can see the re-executed tasks. If you add them, you get a very nice filling curve.

Finally, for evolution strategies, we compared with reference ES from... we followed the OpenAI, and on the X axis, you have number of CPUs, mean time to solve the particular problem; simulator, learning to run, there are three points to notice. One is, as expected, as you add more CPUs, the time to solve goes down. The second is that Ray is actually better than the reference ES, better results, even though the reference ES is specialized for beating. Third, for a very large number of CPUs, ref couldn't do it, but Ray could do better and better. I should add that Ray takes half the amount of code, and was implemented in a couple of hours.

Related work: look, in this area, there are a huge number of systems, that's why you are here, lots of systems. Ray is complimentary to TF, MXNet, PyTorch, etc. We use these systems to implement DNNs. We integrate with TF and PyT. There are more general systems, like MPI and Spark; these have limited support for nested parallelism; computation model, and they have much coarser grained tasks.

To conclude, Ray is a system for high performance and flexibility and scalability. We have two libraries on top of Ray: RLlib and Ray Tune. It's open source, please try, we'd love your feedback. Robert, Philip, Alex, Stephanie, Richard, Eric, Heng, William, and many thanks to my colleague Michael Jordan.

Q: In your system, you also use actor; actor is built up on shared memory. Do you have separate mailbox for actors? How do you do that?

A: No, the actors communicate by passing the argument to the shared object store.

Q: What is the granularity of parallelism? Is it task atomic, or do you split task?

A: The task granularity is given by what is the overhead for launching a task and scheduling the task. The task you see, we are targeting task, low and few ms. The task is not implementing something like activation function. we leave that job to much better frameworks. And a task is executing atomically, a method, in the actors, are serialized.

Q: Question about fault tolerance: in Spark, when you don't have a response for some time, it says this node died. Here, the task is much more, because NN, something like that. So we don't have the same time.

A: We do not do speculation; implicit speculation in Ray, for the reason you mentioned.

Q: Can you give me more details on the reference implementation, doesn't scale

A: The reference implementation, it's the OpenAI implementation, Robert here can provide you a lot more detailed answers to that question.