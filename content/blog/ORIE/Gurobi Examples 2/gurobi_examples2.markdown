---
title: "Gurboi's R Examples 2"
author: "Erick Jones"
date: 2020-01-06
categories: ["ORIE Techniques"]
tags: ["R Markdown", "QP", "Gurboi", "Algorithms"]
---

This post explores how to use Gurobi to solve more advanced LPs, MIPs, and QPs. 
I have written these using Gurobi as a solver and as the mathematical formulation software.
This is a reproducible example if you have R Studio just  make sure you have installed the correct packages.





```r
library(gurobi)
```

```
## Warning: package 'slam' was built under R version 4.1.2
```

```r
library(Matrix)
```
This example formulates and solves the following simple QP model:
 
 `\(min:     x^2 + xy + y^2 + yz + z^2 + 2 x\)`
 
 subject to
       
       `\(x + 2 y + 3z \geq 4\)`
       
       `\(x +   y      \geq 1\)`
       x, y, z non-negative

```r
model <- list()

model$A     <- matrix(c(1,2,3,1,1,0), nrow=2, byrow=T)
model$Q     <- matrix(c(1,0.5,0,0.5,1,0.5,0,0.5,1), nrow=3, byrow=T)
model$obj   <- c(2,0,0)
model$rhs   <- c(4,1)
model$sense <- c('>', '>')

result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 5 nonzeros
## Model fingerprint: 0xe6f007c4
## Model has 5 quadratic objective terms
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [2e+00, 2e+00]
##   QObjective range [2e+00, 2e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 4e+00]
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 5 nonzeros
## Presolved model has 5 quadratic objective terms
## Ordering time: 0.00s
## 
## Barrier statistics:
##  Free vars  : 2
##  AA' NZ     : 6.000e+00
##  Factor NZ  : 1.000e+01
##  Factor Ops : 3.000e+01 (less than 1 second per iteration)
##  Threads    : 1
## 
##                   Objective                Residual
## Iter       Primal          Dual         Primal    Dual     Compl     Time
##    0   1.68862999e+05 -1.66862803e+05  1.50e+03 4.63e-07  9.99e+05     0s
##    1   3.32288030e+05 -3.31121401e+05  1.50e-03 4.55e-13  1.33e+05     0s
##    2   4.88215027e+04 -4.83744738e+04  1.50e-09 2.84e-14  1.94e+04     0s
##    3   7.20552197e+03 -7.03403484e+03  3.55e-14 1.42e-14  2.85e+03     0s
##    4   1.07582166e+03 -1.00982226e+03  1.78e-14 1.07e-14  4.17e+02     0s
##    5   1.65319400e+02 -1.39657698e+02  3.55e-15 3.55e-15  6.10e+01     0s
##    6   2.72141305e+01 -1.68504217e+01  1.33e-15 4.44e-16  8.81e+00     0s
##    7   5.34776479e+00 -4.13214640e-01  2.22e-16 2.22e-16  1.15e+00     0s
##    8   2.27046251e+00  2.04615758e+00  2.22e-16 4.44e-16  4.49e-02     0s
##    9   2.11217859e+00  2.11101837e+00  7.77e-15 1.67e-16  2.32e-04     0s
##   10   2.11111218e+00  2.11111102e+00  5.55e-16 3.29e-16  2.32e-07     0s
##   11   2.11111111e+00  2.11111111e+00  3.33e-15 3.33e-16  2.32e-10     0s
## 
## Barrier solved model in 11 iterations and 0.00 seconds
## Optimal objective 2.11111111e+00
```

```r
print(result$objval)
```

```
## [1] 2.111111
```

```r
print(result$x)
```

```
## [1] 3.584007e-10 1.000000e+00 6.666667e-01
```

```r
model$vtype <- c('I', 'I', 'I')

result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 5 nonzeros
## Model fingerprint: 0x2458258b
## Model has 5 quadratic objective terms
## Variable types: 0 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [2e+00, 2e+00]
##   QObjective range [2e+00, 2e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 4e+00]
## Found heuristic solution: objective 2.000000e+19
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 5 nonzeros
## Presolved model has 5 quadratic objective terms
## Variable types: 0 continuous, 3 integer (0 binary)
## 
## Root relaxation: objective 2.111111e+00, 5 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0    2.11111    0    1 2.0000e+19    2.11111   100%     -    0s
## H    0     0                       3.0000000    2.11111  29.6%     -    0s
##      0     0    2.11111    0    1    3.00000    2.11111  29.6%     -    0s
## 
## Explored 1 nodes (5 simplex iterations) in 0.00 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 2: 3 2e+19 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 3.000000000000e+00, best bound 3.000000000000e+00, gap 0.0000%
```

```r
print(result$objval)
```

```
## [1] 3
```

```r
print(result$x)
```

```
## [1] 0 1 1
```

```r
# Clear space
rm(model, result)
```

This example formulates and solves the following simple QCP model:
 `\(max: x\)`
 
 subject to
 
  `\(x + y + z   =  1\)`
       
  `\(x^2 + y^2 \leq z^2\)`  (second-order cone)
  $ x^2 \leq yz$         (rotated second-order cone)
       x, y, z non-negative


```r
model <- list()

model$A          <- matrix(c(1,1,1), nrow=1, byrow=T)
model$modelsense <- 'max'
model$obj        <- c(1,0,0)
model$rhs        <- c(1)
model$sense      <- c('=')

# First quadratic constraint: x^2 + y^2 - z^2 <= 0
qc1 <- list()
qc1$Qc <- spMatrix(3, 3, c(1, 2, 3), c(1, 2, 3), c(1.0, 1.0, -1.0))
qc1$rhs <- 0.0

# Second quadratic constraint: x^2 - yz <= 0
qc2 <- list()
qc2$Qc <- spMatrix(3, 3, c(1, 2), c(1, 3), c(1.0, -1.0))
qc2$rhs <- 0.0

model$quadcon <- list(qc1, qc2)

result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0xc90fd276
## Model has 2 quadratic constraints
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   QMatrix range    [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 1e+00]
## Presolve time: 0.00s
## Presolved: 6 rows, 6 columns, 13 nonzeros
## Presolved model has 2 second-order cone constraints
## Ordering time: 0.00s
## 
## Barrier statistics:
##  AA' NZ     : 1.500e+01
##  Factor NZ  : 2.100e+01
##  Factor Ops : 9.100e+01 (less than 1 second per iteration)
##  Threads    : 1
## 
##                   Objective                Residual
## Iter       Primal          Dual         Primal    Dual     Compl     Time
##    0   2.38095238e-01  2.38095238e-01  1.11e-16 4.33e-01  9.23e-02     0s
##    1   3.20481543e-01  3.62123302e-01  5.55e-17 1.39e-02  7.95e-03     0s
##    2   3.26649101e-01  3.28651430e-01  1.15e-14 5.44e-04  3.46e-04     0s
##    3   3.26797051e-01  3.27019441e-01  1.37e-13 5.98e-10  2.78e-05     0s
##    4   3.26990986e-01  3.26994814e-01  6.54e-13 3.46e-13  4.78e-07     0s
##    5   3.26992304e-01  3.26992876e-01  3.23e-11 1.94e-14  7.15e-08     0s
## 
## Barrier solved model in 5 iterations and 0.00 seconds
## Optimal objective 3.26992304e-01
## 
## Warning: to get QCP duals, please set parameter QCPDual to 1
```

```r
print(result$objval)
```

```
## [1] 0.3269923
```

```r
print(result$x)
```

```
## [1] 0.3269923 0.2570664 0.4159413
```

```r
# Clear space
rm(model, result)
```
This example considers the following separable, convex problem:

 `\(min:  f(x) - y + g(z)\)`
 
 subject to
 
  `\(x + 2 y + 3 z   \leq 4\)`
  `\(x +   y         \geq 1\)`
  `\(x,    y,    z \geq 0\)`

where `\(f(u) = e^{-u} \text{ and} \: g(u) = 2 u^2 - 4u\: \forall \text{ real}\: u\)`

It formulates and solves a simpler LP model by approximating f and
g with piecewise-linear functions.  Then it transforms the model
into a MIP by negating the approximation for f, which gives
a non-convex piecewise-linear function, and solves it again.


```r
library(gurobi)

model <- list()

model$A     <- matrix(c(1,2,3,1,1,0), nrow=2, byrow=T)
model$obj   <- c(0,-1,0)
model$ub    <- c(1,1,1)
model$rhs   <- c(4,1)
model$sense <- c('<', '>')

# Uniformly spaced points in [0.0, 1.0]
u <- seq(from=0, to=1, by=0.01)

# First piecewise-linear function: f(x) = exp(-x)
pwl1     <- list()
pwl1$var <- 1
pwl1$x   <- u
pwl1$y   <- sapply(u, function(x) exp(-x))

# Second piecewise-linear function: g(z) = 2 z^2 - 4 z
pwl2     <- list()
pwl2$var <- 3
pwl2$x   <- u
pwl2$y   <- sapply(u, function(z) 2 * z * z - 4 * z)

model$pwlobj <- list(pwl1, pwl2)

result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 5 nonzeros
## Model fingerprint: 0xc12b5aad
## Model has 2 piecewise-linear objective terms
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [1e+00, 4e+00]
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 5 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0   -2.6321206e+00   5.000000e-01   0.000000e+00      0s
##        2   -1.9346239e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 2 iterations and 0.00 seconds
## Optimal objective -1.934623931e+00
```

```r
print(result$objval)
```

```
## [1] -1.934624
```

```r
print(result$x)
```

```
## [1] 0.690 0.725 0.620
```

```r
# Negate piecewise-linear function on x, making it non-convex

model$pwlobj[[1]]$y <- sapply(u, function(x) -exp(-x))

result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 5 nonzeros
## Model fingerprint: 0x3229e670
## Model has 2 piecewise-linear objective terms
## Variable types: 3 continuous, 0 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [1e+00, 4e+00]
## Found heuristic solution: objective -1.3678794
## Presolve time: 0.00s
## Presolved: 202 rows, 302 columns, 603 nonzeros
## Variable types: 203 continuous, 99 integer (99 binary)
## 
## Root relaxation: objective -3.777733e+00, 1 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
## *    0     0               0      -3.7777333   -3.77773  0.00%     -    0s
## 
## Explored 0 nodes (1 simplex iterations) in 0.00 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 2: -3.77773 -1.36788 
## No other solutions better than -3.77773
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective -3.777733333333e+00, best bound -3.777733333333e+00, gap 0.0000%
```

```r
gurobi_write(model, "pwl.lp")
```

```
## NULL
```

```r
print(result$objval)
```

```
## [1] -3.777733
```

```r
print(result$x)
```

```
## [1] 0.0000000 1.0000000 0.6666667
```

```r
# Clear space
rm(model, pwl1, pwl2, result)
```

Want to cover three different sets but subject to a common budget of
elements allowed to be used. However, the sets have different priorities to
be covered; and we tackle this by using multi-objective optimization.


```r
# define primitive data
groundSetSize     <- 20
nSubSets          <- 4
Budget            <- 12
Set               <- list(
    c( 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ),
    c( 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1 ),
    c( 0, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 0 ),
    c( 0, 0, 0, 1, 1, 1, 0, 0, 0, 1, 1, 1, 0, 0, 0, 1, 1, 1, 0, 0 ) )
SetObjPriority    <- c(3, 2, 2, 1)
SetObjWeight      <- c(1.0, 0.25, 1.25, 1.0)

# Initialize model
model             <- list()
model$modelsense  <- 'max'
model$modelname   <- 'multiobj'

# Set variables, all of them are binary, with 0,1 bounds.
model$vtype       <- 'B'
model$lb          <- 0
model$ub          <- 1
model$varnames    <- paste(rep('El', groundSetSize), 1:groundSetSize, sep='')

# Build constraint matrix
model$A           <- spMatrix(1, groundSetSize,
                              i = rep(1,groundSetSize),
                              j = 1:groundSetSize,
                              x = rep(1,groundSetSize))
model$rhs         <- c(Budget)
model$sense       <- c('<')
model$constrnames <- c('Budget')

# Set multi-objectives
model$multiobj          <- list()
for (m in 1:nSubSets) {
  model$multiobj[[m]]          <- list()
  model$multiobj[[m]]$objn     <- Set[[m]]
  model$multiobj[[m]]$priority <- SetObjPriority[m]
  model$multiobj[[m]]$weight   <- SetObjWeight[m]
  model$multiobj[[m]]$abstol   <- m
  model$multiobj[[m]]$reltol   <- 0.01
  model$multiobj[[m]]$name     <- sprintf('Set%d', m)
  model$multiobj[[m]]$con      <- 0.0
}

# Save model
gurobi_write(model,'multiobj_R.lp')
```

```
## NULL
```

```r
# Set parameters
params               <- list()
params$PoolSolutions <- 100

# Optimize
result <- gurobi(model, params)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 20 columns and 20 nonzeros
## Model fingerprint: 0x82f8d485
## Variable types: 0 continuous, 20 integer (20 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [1e+01, 1e+01]
## 
## ---------------------------------------------------------------------------
## Multi-objectives: starting optimization with 4 objectives (3 combined) ...
## ---------------------------------------------------------------------------
## 
## Multi-objectives: applying initial presolve ...
## ---------------------------------------------------------------------------
## 
## Presolve time: 0.00s
## Presolved: 1 rows and 20 columns
## ---------------------------------------------------------------------------
## 
## Multi-objectives: optimize objective 1 (Set1) ...
## ---------------------------------------------------------------------------
## 
## Found heuristic solution: objective 10.0000000
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 1: 10 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 1.000000000000e+01, best bound 1.000000000000e+01, gap 0.0000%
## ---------------------------------------------------------------------------
## 
## Multi-objectives: optimize objective 2 (weighted) ...
## ---------------------------------------------------------------------------
## 
## 
## Loaded user MIP start with objective 6.25
## 
## Presolve removed 2 rows and 20 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 2: 10.5 6.25 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 1.050000000000e+01, best bound 1.050000000000e+01, gap 0.0000%
## ---------------------------------------------------------------------------
## 
## Multi-objectives: optimize objective 3 (Set4) ...
## ---------------------------------------------------------------------------
## 
## 
## Loaded user MIP start with objective 6
## 
## Presolve removed 3 rows and 20 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 2: 7 6 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 7.000000000000e+00, best bound 7.000000000000e+00, gap 0.0000%
## 
## ---------------------------------------------------------------------------
## Multi-objectives: solved in 0.00 seconds, solution count 3
```

```r
# Capture solution information
if (result$status != 'OPTIMAL') {
  cat('Optimization finished with status', result$status, '\n')
  stop('Stop now\n')
}

# Print best solution
cat('Selected elements in best solution:\n')
```

```
## Selected elements in best solution:
```

```r
for (e in 1:groundSetSize) {
  if(result$x[e] < 0.9) next
  cat(' El',e,sep='')
}
```

```
##  El2 El3 El4 El5 El6 El7 El8 El9 El10 El11 El12 El17
```

```r
cat('\n')
```

```r
# Iterate over the best 10 solutions
if ('pool' %in% names(result)) {
  solcount <- length(result$pool)
  cat('Number of solutions found:', solcount, '\n')
  if (solcount > 10) {
    solcount <- 10
  }
  cat('Objective values for first', solcount, 'solutions:\n')
  for (k in 1:solcount) {
    cat('Solution', k, 'has objective:', result$pool[[k]]$objval[1], '\n')
  }
} else {
  solcount <- 1
  cat('Number of solutions found:', solcount, '\n')
  cat('Solution 1 has objective:', result$objval, '\n')
}
```

```
## Number of solutions found: 3 
## Objective values for first 3 solutions:
## Solution 1 has objective: 9 
## Solution 2 has objective: 9 
## Solution 3 has objective: 10
```

```r
# Clean up
rm(model, params, result)
```

Assign workers to shifts; each worker may or may not be available on a
particular day.  We use Pareto optimization to solve the model:
first, we minimize the linear sum of the slacks. Then, we constrain
the sum of the slacks, and we minimize a quadratic objective that
tries to balance the workload among the workers.


```r
# define data
nShifts       <- 14
nWorkers      <-  7
nVars         <- (nShifts + 1) * (nWorkers + 1) + nWorkers + 1
varIdx        <- function(w,s) {s+(w-1)*nShifts}
shiftSlackIdx <- function(s) {s+nShifts*nWorkers}
totShiftIdx   <- function(w) {w + nShifts * (nWorkers+1)}
avgShiftIdx   <- ((nShifts+1)*(nWorkers+1))
diffShiftIdx  <- function(w) {w + avgShiftIdx}
totalSlackIdx <- nVars


Shifts  <- c('Mon1', 'Tue2', 'Wed3', 'Thu4', 'Fri5', 'Sat6', 'Sun7',
             'Mon8', 'Tue9', 'Wed10', 'Thu11', 'Fri12', 'Sat13', 'Sun14')
Workers <- c( 'Amy', 'Bob', 'Cathy', 'Dan', 'Ed', 'Fred', 'Gu' )

shiftRequirements <- c(3, 2, 4, 4, 5, 6, 5, 2, 2, 3, 4, 6, 7, 5 )

availability <- list( c( 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 0, 1, 0 ),
                      c( 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1 ),
                      c( 1, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 1 ),
                      c( 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 ) )

# Function to display results
solveandprint <- function(model, env) {
  result <- gurobi(model, env = env)
  if(result$status == 'OPTIMAL') {
    cat('The optimal objective is',result$objval,'\n')
    cat('Schedule:\n')
    for (s in 1:nShifts) {
      cat('\t',Shifts[s],':')
      for (w in 1:nWorkers) {
        if (result$x[varIdx(w,s)] > 0.9) cat(Workers[w],' ')
      }
      cat('\n')
    }
    cat('Workload:\n')
    for (w in 1:nWorkers) {
      cat('\t',Workers[w],':',result$x[totShiftIdx(w)],'\n')
    }
  } else {
    cat('Optimization finished with status',result$status)
  }
  result
}

# Set-up environment
env <- list()
env$logfile <- 'workforce4.log'

# Build model
model            <- list()
model$modelname  <- 'workforce4'
model$modelsense <- 'min'

# Initialize assignment decision variables:
#    x[w][s] == 1 if worker w is assigned to shift s.
#    This is no longer a pure assignment model, so we must
#    use binary variables.
model$vtype    <- rep('C', nVars)
model$lb       <- rep(0, nVars)
model$ub       <- rep(1, nVars)
model$obj      <- rep(0, nVars)
model$varnames <- rep('',nVars)
for (w in 1:nWorkers) {
  for (s in 1:nShifts) {
    model$vtype[varIdx(w,s)]    = 'B'
    model$varnames[varIdx(w,s)] = paste0(Workers[w],'.',Shifts[s])
    if (availability[[w]][s] == 0) model$ub[varIdx(w,s)] = 0
  }
}

# Initialize shift slack variables
for (s in 1:nShifts) {
  model$varnames[shiftSlackIdx(s)] = paste0('ShiftSlack',Shifts[s])
  model$ub[shiftSlackIdx(s)] = Inf
}

# Initialize worker slack and diff variables
for (w in 1:nWorkers) {
  model$varnames[totShiftIdx(w)] = paste0('TotalShifts',Workers[w])
  model$ub[totShiftIdx(w)]       = Inf
  model$varnames[diffShiftIdx(w)]  = paste0('DiffShifts',Workers[w])
  model$ub[diffShiftIdx(w)]        = Inf
  model$lb[diffShiftIdx(w)]        = -Inf
}

#Initialize average shift variable
model$ub[avgShiftIdx]      = Inf
model$varnames[avgShiftIdx] = 'AvgShift'

#Initialize total slack variable
model$ub[totalSlackIdx]      = Inf
model$varnames[totalSlackIdx] = 'TotalSlack'
model$obj[totalSlackIdx]     = 1

# Set-up shift-requirements constraints
model$A           <- spMatrix(nShifts,nVars,
                      i = c(c(mapply(rep,1:nShifts,nWorkers)),
                            c(1:nShifts)),
                      j = c(mapply(varIdx,1:nWorkers,
                                 mapply(rep,1:nShifts,nWorkers)),
                            shiftSlackIdx(1:nShifts)),
                      x = rep(1,nShifts * (nWorkers+1)))
model$sense       <- rep('=',nShifts)
model$rhs         <- shiftRequirements
model$constrnames <- Shifts

# Set TotalSlack equal to the sum of each shift slack
B <- spMatrix(1, nVars,
        i = rep(1,nShifts+1),
        j = c(shiftSlackIdx(1:nShifts),totalSlackIdx),
        x = c(rep(1,nShifts),-1))
model$A           <- rbind(model$A, B)
model$rhs         <- c(model$rhs,0)
model$sense       <- c(model$sense,'=')
model$constrnames <- c(model$constrnames, 'TotalSlack')

# Set total number of shifts for each worker
B <- spMatrix(nWorkers, nVars,
          i = c(mapply(rep,1:nWorkers,nShifts),
                1:nWorkers),
          j = c(mapply(varIdx,c(mapply(rep,1:nWorkers,nShifts)),1:nShifts),
                totShiftIdx(1:nWorkers)),
          x = c(rep(1,nShifts*nWorkers),rep(-1,nWorkers)))
model$A           <- rbind(model$A, B)
model$rhs         <- c(model$rhs,rep(0,nWorkers))
model$sense       <- c(model$sense,rep('=',nWorkers))
model$constrnames <- c(model$constrnames, sprintf('TotalShifts%s',Workers[1:nWorkers]))

# Save initial model
gurobi_write(model,'workforce4.lp', env)
```

```
## NULL
```

```r
# Optimize
result <- solveandprint(model, env)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 22 rows, 128 columns and 232 nonzeros
## Model fingerprint: 0x829a1cb8
## Variable types: 30 continuous, 98 integer (98 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Found heuristic solution: objective 58.0000000
## Presolve removed 22 rows and 128 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 2: 6 58 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 6.000000000000e+00, best bound 6.000000000000e+00, gap 0.0000%
## The optimal objective is 6 
## Schedule:
## 	 Mon1 :Bob  Fred  Gu  
## 	 Tue2 :Amy  Ed  
## 	 Wed3 :Amy  Cathy  Fred  Gu  
## 	 Thu4 :Cathy  Ed  
## 	 Fri5 :Amy  Bob  Cathy  Ed  Gu  
## 	 Sat6 :Bob  Dan  Fred  Gu  
## 	 Sun7 :Amy  Cathy  Ed  Gu  
## 	 Mon8 :Fred  Gu  
## 	 Tue9 :Amy  Ed  
## 	 Wed10 :Amy  Cathy  Gu  
## 	 Thu11 :Amy  Dan  Ed  Gu  
## 	 Fri12 :Amy  Cathy  Dan  Fred  Gu  
## 	 Sat13 :Amy  Bob  Cathy  Dan  Ed  Fred  Gu  
## 	 Sun14 :Amy  Cathy  Dan  Fred  Gu  
## Workload:
## 	 Amy : 10 
## 	 Bob : 4 
## 	 Cathy : 8 
## 	 Dan : 5 
## 	 Ed : 7 
## 	 Fred : 7 
## 	 Gu : 11
```

```r
if (result$status != 'OPTIMAL') stop('Stop now\n')

# Constraint the slack by setting its upper and lower bounds
totalSlack <- result$x[totalSlackIdx]
model$lb[totalSlackIdx] = totalSlack
model$ub[totalSlackIdx] = totalSlack

# Link average number of shifts worked and difference with average
B <- spMatrix(nWorkers+1, nVars,
        i = c(1:nWorkers,
              1:nWorkers,
              1:nWorkers,
              rep(nWorkers+1,nWorkers+1)),
        j = c(totShiftIdx(1:nWorkers),
              diffShiftIdx(1:nWorkers),
              rep(avgShiftIdx,nWorkers),
              totShiftIdx(1:nWorkers),avgShiftIdx),
        x = c(rep(1, nWorkers),
              rep(-1,nWorkers),
              rep(-1,nWorkers),
              rep(1,nWorkers),-nWorkers))
model$A           <- rbind(model$A, B)
model$rhs         <- c(model$rhs,rep(0,nWorkers+1))
model$sense       <- c(model$sense,rep('=',nWorkers+1))
model$constrnames <- c(model$constrnames,
                       sprintf('DiffShifts%s',Workers[1:nWorkers]),
                       'AvgShift')

# Objective: minimize the sum of the square of the difference from the
# average number of shifts worked
model$obj <- 0
model$Q   <- spMatrix(nVars,nVars,
                i = c(diffShiftIdx(1:nWorkers)),
                j = c(diffShiftIdx(1:nWorkers)),
                x = rep(1,nWorkers))

# Save modified model
gurobi_write(model,'workforce4b.lp', env)
```

```
## NULL
```

```r
# Optimize
result <- solveandprint(model, env)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 30 rows, 128 columns and 261 nonzeros
## Model fingerprint: 0x377bf6f1
## Model has 7 quadratic objective terms
## Variable types: 30 continuous, 98 integer (98 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 7e+00]
##   Objective range  [0e+00, 0e+00]
##   QObjective range [2e+00, 2e+00]
##   Bounds range     [1e+00, 6e+00]
##   RHS range        [2e+00, 7e+00]
## Found heuristic solution: objective 37.7142857
## Presolve removed 6 rows and 63 columns
## Presolve time: 0.00s
## Presolved: 24 rows, 65 columns, 136 nonzeros
## Presolved model has 7 quadratic objective terms
## Variable types: 7 continuous, 58 integer (50 binary)
## 
## Root relaxation: objective 2.142857e-01, 221 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
## H    0     0                      25.7142857    0.00000   100%     -    0s
##      0     0    0.21429    0   12   25.71429    0.21429  99.2%     -    0s
## H    0     0                       7.7142857    0.21429  97.2%     -    0s
## H    0     0                       1.7142857    0.21429  87.5%     -    0s
##      0     0    0.21429    0   12    1.71429    0.21429  87.5%     -    0s
##      0     2    0.21429    0   12    1.71429    0.21429  87.5%     -    0s
## 
## Explored 19 nodes (275 simplex iterations) in 0.02 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 4: 1.71429 7.71429 25.7143 37.7143 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 1.714285714285e+00, best bound 1.714285714285e+00, gap 0.0000%
## The optimal objective is 1.714286 
## Schedule:
## 	 Mon1 :Bob  Ed  Fred  
## 	 Tue2 :Bob  Fred  
## 	 Wed3 :Amy  Cathy  Dan  Ed  
## 	 Thu4 :Cathy  Ed  
## 	 Fri5 :Amy  Bob  Cathy  Dan  Gu  
## 	 Sat6 :Bob  Dan  Fred  Gu  
## 	 Sun7 :Amy  Cathy  Ed  Gu  
## 	 Mon8 :Bob  Cathy  
## 	 Tue9 :Dan  Fred  
## 	 Wed10 :Amy  Dan  Gu  
## 	 Thu11 :Bob  Dan  Ed  Gu  
## 	 Fri12 :Amy  Cathy  Dan  Fred  Gu  
## 	 Sat13 :Amy  Bob  Cathy  Dan  Ed  Fred  Gu  
## 	 Sun14 :Amy  Cathy  Ed  Fred  Gu  
## Workload:
## 	 Amy : 7 
## 	 Bob : 7 
## 	 Cathy : 8 
## 	 Dan : 8 
## 	 Ed : 7 
## 	 Fred : 7 
## 	 Gu : 8
```

```r
if (result$status != 'OPTIMAL') stop('Stop now\n')

#Clear space
rm(model, env, availability, Shifts, Workers, shiftRequirements, result)
```


```r
# Assign workers to shifts; each worker may or may not be available on a
# particular day. We use multi-objective optimization to solve the model.
# The highest-priority objective minimizes the sum of the slacks
# (i.e., the total number of uncovered shifts). The secondary objective
# minimizes the difference between the maximum and minimum number of
# shifts worked among all workers.  The second optimization is allowed
# to degrade the first objective by up to the smaller value of 10% and 2



# define data
nShifts       <- 14
nWorkers      <-  8
nVars         <- (nShifts + 1) * (nWorkers + 1) + 2
varIdx        <- function(w,s) {s+(w-1)*nShifts}
shiftSlackIdx <- function(s) {s+nShifts*nWorkers}
totShiftIdx   <- function(w) {w + nShifts * (nWorkers+1)}
minShiftIdx   <- ((nShifts+1)*(nWorkers+1))
maxShiftIdx   <- (minShiftIdx+1)
totalSlackIdx <- nVars


Shifts  <- c('Mon1', 'Tue2', 'Wed3', 'Thu4', 'Fri5', 'Sat6', 'Sun7',
             'Mon8', 'Tue9', 'Wed10', 'Thu11', 'Fri12', 'Sat13', 'Sun14')
Workers <- c( 'Amy', 'Bob', 'Cathy', 'Dan', 'Ed', 'Fred', 'Gu', 'Tobi' )

shiftRequirements <- c(3, 2, 4, 4, 5, 6, 5, 2, 2, 3, 4, 6, 7, 5 )

availability <- list( c( 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 0, 1, 0 ),
                      c( 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1 ),
                      c( 1, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 1 ),
                      c( 0, 1, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 1, 1 ),
                      c( 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 ) )

# Function to display results
solveandprint <- function(model, env) {
  result <- gurobi(model, env = env)
  if(result$status == 'OPTIMAL') {
    cat('The optimal objective is',result$objval,'\n')
    cat('Schedule:\n')
    for (s in 1:nShifts) {
      cat('\t',Shifts[s],':')
      for (w in 1:nWorkers) {
        if (result$x[varIdx(w,s)] > 0.9) cat(Workers[w],' ')
      }
      cat('\n')
    }
    cat('Workload:\n')
    for (w in 1:nWorkers) {
      cat('\t',Workers[w],':',result$x[totShiftIdx(w)],'\n')
    }
  } else {
    cat('Optimization finished with status',result$status)
  }
  result
}

# Set-up environment
env <- list()
env$logfile <- 'workforce5.log'

# Build model
model            <- list()
model$modelname  <- 'workforce5'
model$modelsense <- 'min'

# Initialize assignment decision variables:
#    x[w][s] == 1 if worker w is assigned to shift s.
#    This is no longer a pure assignment model, so we must
#    use binary variables.
model$vtype    <- rep('C', nVars)
model$lb       <- rep(0, nVars)
model$ub       <- rep(1, nVars)
model$varnames <- rep('',nVars)
for (w in 1:nWorkers) {
  for (s in 1:nShifts) {
    model$vtype[varIdx(w,s)]    = 'B'
    model$varnames[varIdx(w,s)] = paste0(Workers[w],'.',Shifts[s])
    if (availability[[w]][s] == 0) model$ub[varIdx(w,s)] = 0
  }
}

# Initialize shift slack variables
for (s in 1:nShifts) {
  model$varnames[shiftSlackIdx(s)] = paste0('ShiftSlack',Shifts[s])
  model$ub[shiftSlackIdx(s)] = Inf
}

# Initialize worker slack and diff variables
for (w in 1:nWorkers) {
  model$varnames[totShiftIdx(w)] = paste0('TotalShifts',Workers[w])
  model$ub[totShiftIdx(w)]       = Inf
}

#Initialize min/max shift variables
model$ub[minShiftIdx]       = Inf
model$varnames[minShiftIdx] = 'MinShift'
model$ub[maxShiftIdx]       = Inf
model$varnames[maxShiftIdx] = 'MaxShift'

#Initialize total slack variable
model$ub[totalSlackIdx]      = Inf
model$varnames[totalSlackIdx] = 'TotalSlack'

# Set-up shift-requirements constraints
model$A           <- spMatrix(nShifts,nVars,
                      i = c(c(mapply(rep,1:nShifts,nWorkers)),
                            c(1:nShifts)),
                      j = c(mapply(varIdx,1:nWorkers,
                                 mapply(rep,1:nShifts,nWorkers)),
                            shiftSlackIdx(1:nShifts)),
                      x = rep(1,nShifts * (nWorkers+1)))
model$sense       <- rep('=',nShifts)
model$rhs         <- shiftRequirements
model$constrnames <- Shifts

# Set TotalSlack equal to the sum of each shift slack
B <- spMatrix(1, nVars,
        i = rep(1,nShifts+1),
        j = c(shiftSlackIdx(1:nShifts),totalSlackIdx),
        x = c(rep(1,nShifts),-1))
model$A           <- rbind(model$A, B)
model$rhs         <- c(model$rhs,0)
model$sense       <- c(model$sense,'=')
model$constrnames <- c(model$constrnames, 'TotalSlack')

# Set total number of shifts for each worker
B <- spMatrix(nWorkers, nVars,
          i = c(mapply(rep,1:nWorkers,nShifts),
                1:nWorkers),
          j = c(mapply(varIdx,c(mapply(rep,1:nWorkers,nShifts)),1:nShifts),
                totShiftIdx(1:nWorkers)),
          x = c(rep(1,nShifts*nWorkers),rep(-1,nWorkers)))
model$A           <- rbind(model$A, B)
model$rhs         <- c(model$rhs,rep(0,nWorkers))
model$sense       <- c(model$sense,rep('=',nWorkers))
model$constrnames <- c(model$constrnames, sprintf('TotalShifts%s',Workers[1:nWorkers]))

# Set minShift / maxShift general constraints
model$genconmin <- list(list(resvar = minShiftIdx,
                             vars   = c(totShiftIdx(1:nWorkers)),
                             name   = 'MinShift'))
model$genconmax <- list(list(resvar = maxShiftIdx,
                             vars   = c(totShiftIdx(1:nWorkers)),
                             name   = 'MaxShift'))

# Set multiobjective
model$multiobj <- list(1:2)
model$multiobj[[1]]          <- list()
model$multiobj[[1]]$objn     <- c(rep(0,nVars))
model$multiobj[[1]]$objn[totalSlackIdx] = 1
model$multiobj[[1]]$priority <- 2
model$multiobj[[1]]$weight   <- 1
model$multiobj[[1]]$abstol   <- 2
model$multiobj[[1]]$reltol   <- 0.1
model$multiobj[[1]]$name     <- 'TotalSlack'
model$multiobj[[1]]$con      <- 0.0
model$multiobj[[2]]          <- list()
model$multiobj[[2]]$objn     <- c(rep(0,nVars))
model$multiobj[[2]]$objn[minShiftIdx] = -1
model$multiobj[[2]]$objn[maxShiftIdx] =  1
model$multiobj[[2]]$priority <- 1
model$multiobj[[2]]$weight   <- 1
model$multiobj[[2]]$abstol   <- 0
model$multiobj[[2]]$reltol   <- 0
model$multiobj[[2]]$name     <- 'Fairness'
model$multiobj[[2]]$con      <- 0.0


# Save initial model
gurobi_write(model,'workforce5.lp', env)
```

```
## NULL
```

```r
# Optimize
result <- solveandprint(model, env)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 23 rows, 137 columns and 261 nonzeros
## Model fingerprint: 0x68a3b7b1
## Model has 2 general constraints
## Variable types: 25 continuous, 112 integer (112 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## 
## ---------------------------------------------------------------------------
## Multi-objectives: starting optimization with 2 objectives ... 
## ---------------------------------------------------------------------------
## 
## Multi-objectives: applying initial presolve ...
## ---------------------------------------------------------------------------
## 
## Presolve added 13 rows and 0 columns
## Presolve removed 0 rows and 3 columns
## Presolve time: 0.00s
## Presolved: 36 rows and 134 columns
## ---------------------------------------------------------------------------
## 
## Multi-objectives: optimize objective 1 (TotalSlack) ...
## ---------------------------------------------------------------------------
## 
## Presolve added 8 rows and 0 columns
## Presolve removed 0 rows and 20 columns
## Presolve time: 0.00s
## Presolved: 44 rows, 114 columns, 224 nonzeros
## Presolved model has 8 SOS constraint(s)
## Variable types: 18 continuous, 96 integer (81 binary)
## Found heuristic solution: objective 7.0000000
## Found heuristic solution: objective 6.0000000
## 
## Root relaxation: objective 3.000000e+00, 30 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0    3.00000    0    4    6.00000    3.00000  50.0%     -    0s
## H    0     0                       3.0000000    3.00000  0.00%     -    0s
##      0     0    3.00000    0    4    3.00000    3.00000  0.00%     -    0s
## 
## Explored 1 nodes (30 simplex iterations) in 0.01 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 3: 3 6 7 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 3.000000000000e+00, best bound 3.000000000000e+00, gap 0.0000%
## ---------------------------------------------------------------------------
## 
## Multi-objectives: optimize objective 2 (Fairness) ...
## ---------------------------------------------------------------------------
## 
## 
## Loaded user MIP start with objective 4
## 
## Presolve added 8 rows and 0 columns
## Presolve removed 0 rows and 10 columns
## Presolve time: 0.00s
## Presolved: 45 rows, 124 columns, 273 nonzeros
## Presolved model has 8 SOS constraint(s)
## Variable types: 18 continuous, 106 integer (81 binary)
## 
## Root relaxation: objective 0.000000e+00, 74 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0    0.00000    0   16    4.00000    0.00000   100%     -    0s
## H    0     0                       2.0000000    0.00000   100%     -    0s
## H    0     0                       1.0000000    0.00000   100%     -    0s
##      0     0    0.14286    0   20    1.00000    0.14286  85.7%     -    0s
##      0     0    0.14286    0   19    1.00000    0.14286  85.7%     -    0s
##      0     0    0.14286    0   21    1.00000    0.14286  85.7%     -    0s
##      0     0    0.14286    0   20    1.00000    0.14286  85.7%     -    0s
##      0     0    0.14286    0   19    1.00000    0.14286  85.7%     -    0s
##      0     2    0.14286    0   19    1.00000    0.14286  85.7%     -    0s
## 
## Explored 3265 nodes (9169 simplex iterations) in 0.12 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 4: 1 1 2 4 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 1.000000000000e+00, best bound 1.000000000000e+00, gap 0.0000%
## 
## ---------------------------------------------------------------------------
## Multi-objectives: solved in 0.12 seconds, solution count 6
## 
## The optimal objective is 4 1 
## Schedule:
## 	 Mon1 :Bob  Fred  Tobi  
## 	 Tue2 :Amy  Bob  
## 	 Wed3 :Cathy  Dan  Fred  Gu  
## 	 Thu4 :Cathy  Ed  Gu  
## 	 Fri5 :Amy  Bob  Cathy  Dan  Ed  
## 	 Sat6 :Bob  Dan  Fred  Gu  Tobi  
## 	 Sun7 :Amy  Cathy  Ed  Gu  Tobi  
## 	 Mon8 :Bob  Ed  
## 	 Tue9 :Dan  Fred  
## 	 Wed10 :Amy  Cathy  Dan  
## 	 Thu11 :Amy  Bob  Ed  Gu  
## 	 Fri12 :Cathy  Dan  Fred  Tobi  
## 	 Sat13 :Amy  Bob  Cathy  Dan  Ed  Fred  Tobi  
## 	 Sun14 :Amy  Ed  Fred  Gu  Tobi  
## Workload:
## 	 Amy : 7 
## 	 Bob : 7 
## 	 Cathy : 7 
## 	 Dan : 7 
## 	 Ed : 7 
## 	 Fred : 7 
## 	 Gu : 6 
## 	 Tobi : 6
```

```r
if (result$status != 'OPTIMAL') stop('Stop now\n')

#Clear space
rm(model, env, availability, Shifts, Workers, shiftRequirements, result)
```

Add a new chunk by clicking the *Insert Chunk* button on the toolbar or by pressing *Ctrl+Alt+I*.

When you save the notebook, an HTML file containing the code and output will be saved alongside it (click the *Preview* button or press *Ctrl+Shift+K* to preview the HTML file).

The preview shows you a rendered HTML copy of the contents of the editor. Consequently, unlike *Knit*, *Preview* does not run any R code chunks. Instead, the output of the chunk when it was last run in the editor is displayed.
