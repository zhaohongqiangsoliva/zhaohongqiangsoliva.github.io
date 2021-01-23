---
layout:     post
title:      Trapezoidal calculation of the integral method
subtitle:   Trapezoidal calculation of the integral method
date:       2020-06-09
author:     soliva
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - 留学
---



# MPI to message collect

## install

Step1 install python3 for anaconda

Then install openmpi

Step2 used pip install mpi4py

## introduction 

used package 

```python
from mpi4py import MPI
```



## point to point communications 

### Arguments

```
MPI.COMM_SELF  
MPI.COMM_WORLD  
comm = MPI.COMM_WORLD     
comm.Get_rank() 
comm.Get_size()
comm.Get_group()
comm.Split(0, 0)
comm.send()
comm.recv()
```

```python
from mpi4py import MPI
import numpy as np
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()
if rank == 0:
    data = range(10)
    comm.send(data, dest=1, tag=11)
    print("process {} send {}...".format(rank, data))
else:
    data = comm.recv(source=0, tag=11)
    print("process {} recv {}...".format(rank, data))
```

