---
layout: post
title: parellel programming
categories : OpenCL
refer1 : https://downloads.ti.com/mctools/esd/docs/opencl/execution/kernels-workgroups-workitems.html
---

------------

What are 3 traditional ways HW designers make computers run faster?

빠르게 구멍을 파고 싶을 때 

1) 삽질을 빨리한다. 
2) 생산성이 높은 삽을 얻는다(한 개의 삽에 삽 헤드가 여러 개)
3) 여러 명의 도움을 얻는다

각각을 Computation 문제에 적용하면 

1) Clock을 빠르게 한다(shorter time for each computation) -> power comsumption 증가 
2) Instruction 단위 parallelism per cycle -> 생산성 증대에 한계
3) many weaker less powerful processors


modern gpu 

1) thousands of ALUs (수 천개의 연산을 동시에 수행)
2) hundreds of processors ()
3) tens of thousands of concurrent threads (동시 수행 흐름을 갖는다)

=> 하지만 gpu를 사용하여 성능을 증가시키기 위하여 parallel하게 프로그래밍을 해야한다.


Minimize time spent on memory


=> move frequetly-accessed data to fast memory

local (특정 thread에 속함 - local variable 등) > shared(thread block) >> global(all thread) >> CPU(host memory)

local은 보통 레지스터나 l1 cache에 존재한다. 

coalesce global memory accesses

GPU most efficient when threads read or write contiguous  memory locations
