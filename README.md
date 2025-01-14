# Hungarian

[![CI](https://github.com/Gnimuc/Hungarian.jl/actions/workflows/ci.yml/badge.svg)](https://github.com/Gnimuc/Hungarian.jl/actions/workflows/ci.yml)
[![TagBot](https://github.com/Gnimuc/Hungarian.jl/actions/workflows/TagBot.yml/badge.svg)](https://github.com/Gnimuc/Hungarian.jl/actions/workflows/TagBot.yml)
[![pkgeval](https://juliahub.com/docs/Hungarian/pkgeval.svg)](https://juliahub.com/ui/Packages/Hungarian/effdR)
[![codecov.io](http://codecov.io/github/Gnimuc/Hungarian.jl/coverage.svg?branch=master)](http://codecov.io/github/Gnimuc/Hungarian.jl?branch=master)
[![version](https://juliahub.com/docs/Hungarian/version.svg)](https://juliahub.com/ui/Packages/Hungarian/effdR)
[![deps](https://juliahub.com/docs/Hungarian/deps.svg)](https://juliahub.com/ui/Packages/Hungarian/effdR?t=2)
[![Downloads](https://shields.io/endpoint?url=https://pkgs.genieframework.com/api/v1/badge/Hungarian)](https://pkgs.genieframework.com?packages=Hungarian)

The package provides one implementation of the **[Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm)** (*Kuhn-Munkres algorithm*) based on its matrix interpretation. This implementation uses a sparse matrix to keep tracking those marked zeros, so it costs less time and memory than [Munkres.jl](https://github.com/FugroRoames/Munkres.jl). Benchmark details can be found [here](https://github.com/Gnimuc/Hungarian.jl/tree/master/benchmark).

## Installation
```julia
pkg> add Hungarian
```

## Quick start
Let's say we have 5 workers and 3 tasks with the following cost matrix:

```julia
weights = [ 24     1     8;
             5     7    14;
             6    13    20;
            12    19    21;
            18    25     2]
```

We can solve the assignment problem by:

```julia
julia> using Hungarian

julia> assignment, cost = hungarian(weights)
([2,1,0,0,3],8)

# worker 1 => task 2 with weights[1,2] = 1
# worker 2 => task 1 with weights[2,1] = 5
# worker 5 => task 3 with weights[5,3] = 2
# the minimal cost is 1 + 5 + 2 = 8  
```

Since each worker can perform only one task and each task can be assigned to only one worker, those `0`s in the `assignment` mean that no task is assigned to those workers.

If a job-worker assignment is not possible, use the special `missing` value to indicate which pairs are disallowed:

```julia
julia> using Hungarian

julia> weights = [missing 1 1; 1 0 1; 1 1 0]
3×3 Matrix{Union{Missing, Int64}}:
  missing  1  1
 1         0  1
 1         1  0

julia> assignment, cost = hungarian(weights)
([2, 1, 3], 2)
```

## Usage
When solving a canonical assignment problem, namely, the cost matrix is square, one can directly get the matching via `Hungarian.munkres(x)` instead of `hungarian(x)`:

```julia
julia> using Hungarian

julia> matching = Hungarian.munkres(rand(5,5))
5×5 SparseArrays.SparseMatrixCSC{Int8,Int64} with 7 stored entries:
  [1, 1]  =  1
  [5, 1]  =  2
  [1, 2]  =  2
  [2, 3]  =  2
  [2, 4]  =  1
  [3, 4]  =  2
  [4, 5]  =  2

# 0 => non-zero
# 1 => zero
# 2 => STAR
julia> Matrix(matching)
5×5 Array{Int8,2}:
 1  2  0  0  0
 0  0  2  1  0
 0  0  0  2  0
 0  0  0  0  2
 2  0  0  0  0

julia> [findfirst(matching[i,:].==Hungarian.STAR) for i = 1:5]
5-element Array{Int64,1}:
 2
 3
 4
 5
 1

julia> [findfirst(matching[:,i].==Hungarian.STAR) for i = 1:5]
5-element Array{Int64,1}:
 5
 1
 2
 3
 4
```

If a job-worker assignment is not possible, use the special `missing` value to indicate which pairs are disallowed:

```julia
julia> using Hungarian

julia> weights = [missing 1 1; 1 0 1; 1 1 0]
3×3 Matrix{Union{Missing, Int64}}:
  missing  1  1
 1         0  1
 1         1  0

julia> matching = Hungarian.munkres(weights)
3×3 SparseArrays.SparseMatrixCSC{Int8, Int64} with 6 stored entries:
 ⋅  2  1
 2  1  ⋅
 1  ⋅  2
```

> **Note**
> If some jobs or workers cannot be assigned to any worker or job, respectively, i.e. if some rows or columns only have `missing` or infinite values (`typemax(T)` where `T` is the type of the elements of the cost matrix), the matrix returned by `Hungarian.munkres` will have zeroes for these rows and columns.

## References
1. J. Munkres, "Algorithms for the Assignment and Transportation Problems", Journal of the Society for Industrial and Applied Mathematics, 5(1):32–38, 1957 March.

2. http://csclab.murraystate.edu/bob.pilgrim/445/munkres.html
