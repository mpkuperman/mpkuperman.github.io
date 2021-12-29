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

## Longstaff-Schwartz SDEs system

$\begin{align}
\frac{dx(t)}{dt} =& \kappa{_1} \left( \theta{_1} - x\left( t \right) \right) \\
\frac{dy(t)}{dt} =& \kappa{_2} \left( \theta{_2} - y\left( t \right) \right)
\end{align}$

$\begin{equation}
\left[
\begin{array}{c}
\sigma{_1} \sqrt{x\left( t \right)} \\
\sigma{_2} \sqrt{y\left( t \right)} \\
\end{array}
\right]
\end{equation}$

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




