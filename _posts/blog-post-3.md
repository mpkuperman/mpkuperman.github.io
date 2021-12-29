---
title: How to optimize Julia SDEs
date: 2021-12-29
permalink: /posts/2021/12/blog-post-3/
 
---

Hi all!. 
In this post I'll show how to optimize the simulation of a Stochastic Differential Equations System (SDEs) written in Julia Language. 
To that end I'll use the great DifferentialEquations.jl package. 
Firstly, stack and heap memory types will be defined. 
Then, the Longstaff-Schwartz SDEs system will be defined in two flavours: the in-place (iip) and out-of-place (oop) versions will be shown. 
For the oop system the StaticArrays.jl package will be used in order to provide stack allocated arrays. 
Finally the different defined SDEs will be benchmarked for adaptive and non-adaptive  in both heap allocations and elapsed time.

## Stack vs Heap allocations: the key for speed
![image](https://i.stack.imgur.com/9c2VH.png)
Stack vs Heap ordering representation. Image taken from https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap.

## Longstaff-Schwartz SDEs system

![image](https://user-images.githubusercontent.com/29048170/147709341-01becdd8-385d-410b-95d8-813f4c66e4c9.png)

![image](https://user-images.githubusercontent.com/29048170/147709286-c674e73a-12c3-479f-9761-c3be389a36eb.png)


```julia

  u0 = [0.1, 0.1]

  tspan = (0.0, 1.0)
  p = (κ₁ = 0.1, κ₂ = 0.1, θ₁ = 0.1, θ₂ = 0.1, σ₁ = 0.1, σ₂ = 0.1)

  function f(du, u, p, t)
      @unpack κ₁, κ₂, θ₁, θ₂ = p

      du[1] = κ₁ * (θ₁ - u[1])
      du[2] = κ₂ * (θ₂ - u[2])
  end

  function g(du, u, p, t)
      @unpack σ₁, σ₂ = p

      du[1] = σ₁ * sqrt(u[1])
      du[2] = σ₂ * sqrt(u[2])
  end

  ls_iip = SDEProblem{true}(f, g, u0, tspan, params)
  
```

```julia

 u01 = @SVector [0.1, 0.1]

 function f(u, p, t)
      @unpack κ₁, κ₂, θ₁, θ₂ = p

      du1 = κ₁ * (θ₁ - u[1])
      du2 = κ₂ * (θ₂ - u[2])

      @SVector [du1, du2]
  end

  function g(u, p, t)
      @unpack σ₁, σ₂ = p

      du1 = σ₁ * sqrt(u[1])
      du2 = σ₂ * sqrt(u[2])

      @SVector [du1, du2]
  end 

  ls_oop = SDEProblem{false}(f, g, u0, tspan, params, kwargs...)

```

## Benchmarking 

```julia

@btime solve(ls_iip)
@btime solve(ls_oop)

@btime solve(ls_iip, dt=1/252, adaptive=false)
@btime solve(ls_oop, dt=1/252, adaptive=false)

@btime solve(ls_iip, EM(), dt = 1 / 252)
@btime solve(ls_oop, EM(), dt = 1 / 252)

```

## Conclusions




