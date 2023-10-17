---
layout: post
date: 2021-05-28 15:59:00-0400
inline: False
title: Joined AMD's Server Performance team as an Intern! 
---

It was a great experience. I had deferred my admission from Georgia Tech and was looking for an internship. I was very fortunate to work in the The Server Performance Group at AMD. The folks in the team are absolutely fantastic and I learned so much. I am so glad I got to do this before starting Grad School. The team deals with analysing the performance of the AMD EPYC based servers which requires a good understanding of both the benchmark (or software) and the processor's architecture.  

I was tasked with evaluating the performance of a microservice benchmark on the AMD EPYC processors and learned how to establish reliable performance baselines and identify important performance bottlenecks. I was also introduced to the standard microbenchmarks used to benchmark different components of the system. This provides a way to correlate the performance of an application to the system architecture. I specifically dove into the [lmbench](https://www.usenix.org/conference/usenix-1996-annual-technical-conference/lmbench-portable-tools-performance-analysis) context switch benchmark. This also involved delving into Linux kernel internals to a significant extent and I was able to isolate the exact line in the kernel code that was responsible for the performance disparity.

Another interesting thing that I worked on taught me how small changes in hardware configurations could have an extremely large impact on microservice workloads. Hypothesizing possible explanations for performance inefficiencies was really fun and exciting and my mentors taught me how to go about prioritizing the different hypotheses while debugging. I feel like this is a very important skill as one can come up with a huge number of explanations for any given issue and prioritizing how to isolate the correct issue is very important. 

I really loved it and am very grateful for the opportunity! 