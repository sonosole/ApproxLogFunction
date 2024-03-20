# ApproxLogFunction

A lookup table based method to calculate $y = {\rm log}_b(x)$ with a controllable error, where $b > 1$, $x > 0$, and $x$ shall be the **IEEE754** floating-point **normal** numbers.

## 💽 Installation

```julia
julia> import Pkg;
julia> Pkg.add("ApproxLogFunction");
```

## 👨‍🏫 Usage

### 🔌 Exposed Functor

The exposed constructor :

```julia
Approxlog(base::Real; abserror::Real, dtype::Type{<:AbstractFloat})
```

where  `base`  is the radix of logarithm operation,  `abserror`  is the maximum absolute error of the `output` and  `dtype`  is the data type of input and output value, for example :

```julia
alog₂  = Approxlog(2, abserror=0.1, dtype=Float32);
input  = 5.20f2;
output = alog₂(input);
```

### 📈 Error Visualization

```julia
using Plots, ApproxLogFunction

figs = []
base = 2
t  = -2f-0 : 1f-4 : 5f0;
x  = 2 .^ t;
y₁ = log.(base, x)
for Δout in [0.4, 0.2]
    alog = Approxlog(base, abserror=Δout, dtype=Float32)
    y₂ = alog.(x)
    fg = plot(x, y₁, linewidth=0.8, xscale=:log10);
    plot!(x, y₂, linewidth=0.7, titlefontsize=12)
    title!("abserror=$Δout")
    push!(figs, fg)
end
plot(figs..., layout=(1,2), leg=nothing, size=(999,666))
```

![error](./doc/err.jpg)

## 🥇 Benchmark

Some unimportant information was omitted when benchmarking.

### 💃🏻 Scalar Input

```julia
julia> using ApproxLogFunction, BenchmarkTools;
julia> begin
           type = Float32;
           base = type(2.0);
           Δout = 0.1;
           alog = Approxlog(base, abserror=Δout, dtype=type);
       end;
julia> x = 1.23456f0;
julia> @benchmark y₁ = log($base, $x)
 Time  (median):     21.643 ns

julia> @benchmark y₁ = log2($x)
 Time  (median):     14.800 ns

julia> @benchmark y₂ = alog($x)
 Time  (median):     74.129 ns
```

### 👨‍👨‍👧‍👧 Array Input

```julia
julia> using ApproxLogFunction, BenchmarkTools;
julia> begin
           type = Float32;
           base = type(2.0);
           Δout = 0.1;
           alog = Approxlog(base, abserror=Δout, dtype=type);
       end;
julia> x = rand(type, 1024, 1024);
julia> @benchmark y₁ = log.($base, $x)
 Time  (median):     21.422 ms

julia> @benchmark y₁ = log2.($x)
 Time  (median):     11.626 ms

julia> @benchmark y₂ = alog.($x)
 Time  (median):     5.207 ms
```

Calculating scaler input is slower, but why calculating array input is faster ? 😂😂😂

## ⚠️ Attention

Error is well controlled when using IEEE754 floating-point positive **normal** numbers, which is represented as :

$$
x = 2^{E-B} \ (1 + F × 2^{-|F|})
$$

where $0 < E < (2^{|E|} - 1)$ is the value of exponent part, $|E|$ is the bit width of $E$,  $F$ is the value of fractional part,  $|F|$ is the bit width of $F$, and $B=(2 ^ {|E| - 1} - 1)$  is the bias for $E$.  So the range for an `Approxlog` object's input is $[X_{\rm min}, X_{\rm max}]$, where

$$
\begin{aligned}
X_{\rm min} &= 2^{1-B} \ (1 + 0 × 2^{-|F|}) &&= 2^{1-B} \\
X_{\rm max} &= 2^{(2^{|E|}-2)-B} \ \big(1 + (2^{|F|}-1) × 2^{-|F|}\big) &&= 2^{B} \ \big(1 + (2^{|F|}-1) × 2^{-|F|}\big)
\end{aligned}
$$

In short, for `Float16`  input, its range is :

$$
2^{1-15} \le
X_{\rm Float16} \le 2^{15} \ \left( 1 + (2^{10} -1) × 2^{-10} \right)
$$

for `Float32` input :

$$
2^{1-127} \le
X_{\rm Float32} \le 2^{127} \ \left(1 + (2^{23} -1) × 2^{-23}\right)
$$

and for `Float64` input :

$$
2^{1-1023} \le
X_{\rm Float64} \le 2^{1023} \ \left(1 + (2^{52} -1) × 2^{-52}\right)
$$

As to positive **subnormal** numbers, the result is not reliable.

## 📁 C-Lang Files

Interface function :

```julia
toclang(dir::String, filename::String, funcname::String, f::Approxlog)
```

`dir`  is the directory to save the C language files (`dir`  would be made if it doesn't exist before),   `filename` is the name of the generated `.c` and `.h` files, `funcname` is the name of the generated approximate function and `f`  is the approximate log functor, for example :

```julia
alog₂ = Approxlog(2, abserror=0.12, dtype=Float32)
toclang("./cfolder/",  "approxlog", "alog", alog₂)
```

this would generate  `approxlog.c`  and  `approxlog.h`  files in `./cfolder/`, the function is named as `alog` .
