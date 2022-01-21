---
title: "Decompositions Algorithms Broken Down and Explained"
author: "Erick Jones"
date: 2020-01-10
categories: ["ORIE Techniques"]
tags: ["R Markdown", "Benders", "Column Generation", "Decomposition", "LP", "MIP", "Algorithms"]
---

This post explores how to use various decomposition techniques to solve LPs and MIPs. 
I have written these using Gurobi as a solver and as the mathematical formulation software.
This is a reproducible example if you have R Studio just  make sure you have installed the correct packages.









```r
library(gurobi)
```

```
## Warning: package 'slam' was built under R version 4.1.2
```

```r
library(tictoc)
library(Matrix)
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 4.1.2
```


```r
#Using homogenous equations to generate extreme points for (optimality) and extreme rays for (feasiblity)
#max 2x1 + x2 + 13x3 + 7y1 + 5y2
#s.t. 9x1+4x2+14x3+35y1+24y2 <= 80; -x1-2x2+3x3-3y1+4y2 <= 10
#x >=0, y>= 0, x is int

#RMP
#max z + 2x1 + x2 + 13x3
# z >= + (80-9x1-4x2-14x3)*u1 + (10+x1+2x2-3x3)*u2 u == 0 initial guess
u <- c(0,0)
RMP <- list()

RMP$A          <- matrix(c(1,(u[1]*9-u[2]),(u[1]*4-2*u[2]),(14*u[1]+3*u[2])), nrow=1, byrow=T)
RMP$obj        <- c(1,2,1,13)
RMP$modelsense <- 'max'
RMP$rhs        <- c((80*u[1]+10*u[2]))
RMP$sense      <- c('<')
RMP$vtype      <- c('C','I','I','I')

result <- gurobi(RMP)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 4 columns and 1 nonzeros
## Model fingerprint: 0xd1b6d0fc
## Variable types: 1 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [0e+00, 0e+00]
## Found heuristic solution: objective 1.600000e+31
## Presolve time: 0.00s
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 1: 1.6e+31 
## No other solutions better than 0
## 
## Model is unbounded
## Warning: some integer variables take values larger than the maximum
##          supported value (2000000000)
## Best objective 1.600000000000e+31, best bound -, gap -
```

```r
UB <- result$objval
print(paste('RMP Objective Value and New Upper bound:', result$objval))
```

```
## [1] "RMP Objective Value and New Upper bound: 1.6e+31"
```

```r
print(paste('Value of z:', result$x[1]))
```

```
## [1] "Value of z: 0"
```

```r
x <- result$x[-1]
print(paste('Value of x:', t(as.matrix(x))))
```

```
## [1] "Value of x: 1e+30" "Value of x: 1e+30" "Value of x: 1e+30"
```

```r
if(result$status == "INF_OR_UNBD" | result$status == "UNBOUNDED"){
  UB <- 999999
  x <- c(0,0,0)
}

rm(result)

#Primal Subproblem
# 2x1 + x2 + 13x3 + max 7y1 + 5y2
#s.t. 35y1+24y2 <= 80-(9x1+4x2+14x3); -3y1+4y2 <= 10-(-x1-2x2+3x3)

LB <- -999999

LB_list <- LB
UB_list <- UB
x_list <- x
u_list <- u
y <- c(0,0)
y_list <- y

#Keeps adding Benders Cuts to Problem
while(LB != UB){

#Dual Subproblem
#min (80-(9x1+4x2+14x3))u1 + (10-(-x1-2x2+3x3))u2
#s.t. 35u1 -3u2 >= 7; 24u1+4u2 >= 5

DSB <- list()

DSB$A          <- matrix(c(35,-3,24,4), nrow=2, byrow=T)
DSB$obj        <- c((80-9*x[1]-4*x[2]-14*x[3]),(10+x[1]+2*x[2]-3*x[3]))
DSB$modelsense <- 'min'
DSB$rhs        <- c(7,5)
DSB$sense      <- c('>', '>')

result <- gurobi(DSB)

LB <- result$objval + 2 * x[1] + x[2] + 13*x[3]
print(paste('DSB Objective Value and New Lower bound:', LB))
u <- result$x
y <- result$pi
print(paste('Value of u:', u))
print(paste('Value of y:', y))



#add new constraint
#z >= u[1]*(3-y1) + u[2]*(4-3y1)
#z + (u[1]*y1) + u[2]*3y1 >= u[1]*3+u[2]*4

B <- matrix(c(1,(u[1]*9-u[2]),(u[1]*4-2*u[2]),(14*u[1]+3*u[2])), nrow=1, byrow=T)
b <- u[1]*80+u[2]*10

if(result$status == "INF_OR_UNBD" | result$status == 'UNBOUNDED'){
  DSB$A <- rbind(DSB$A,c(1,1))
  DSB$rhs <- c(0,0,1)
  DSB$sense <- c(DSB$sense, '=')
  result <- gurobi(DSB)
  u <- result$x
  LB <- LB_list[length(LB_list)]
  B <- matrix(c(0,(u[1]*9-u[2]),(u[1]*4-2*u[2]),(14*u[1]+3*u[2])), nrow=1, byrow=T)
  b <- u[1]*80+u[2]*10
  
  
}

u_list <- rbind(u_list,u)
y_list <- rbind(y_list,y)
LB_list <- c(LB_list, LB)

rm(result)


RMP$A <- rbind(RMP$A, B)
RMP$rhs <- c(RMP$rhs,b)
RMP$sense <- c(RMP$sense, '<')


result <- gurobi(RMP)

UB <- result$objval
print(paste('RMP Objective Value and New Upper bound:', result$objval))
print(paste('Value of z:', result$x[1]))
x <- result$x[-1]
print(paste('Value of x:', t(as.matrix(x))))

if(result$status == "INF_OR_UNBD"){
  UB <- 999999
  x <- c(0,0,0)
}

UB_list <- c(UB_list, UB)
x_list <- rbind(x_list,x)
rm(result)

}
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 2 columns and 4 nonzeros
## Model fingerprint: 0xfecd2aa1
## Coefficient statistics:
##   Matrix range     [3e+00, 4e+01]
##   Objective range  [1e+01, 8e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [5e+00, 7e+00]
## Presolve time: 0.00s
## Presolved: 2 rows, 2 columns, 4 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    0.0000000e+00   1.500000e+00   0.000000e+00      0s
##        2    1.6556604e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 2 iterations and 0.01 seconds
## Optimal objective  1.655660377e+01
## [1] "DSB Objective Value and New Lower bound: 16.5566037735849"
## [1] "Value of u: 0.202830188679245"  "Value of u: 0.0330188679245282"
## [1] "Value of y: 0.377358490566038" "Value of y: 2.78301886792453" 
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 4 columns and 5 nonzeros
## Model fingerprint: 0x8b4885a1
## Variable types: 1 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [7e-01, 3e+00]
##   Objective range  [1e+00, 1e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+01, 2e+01]
## Found heuristic solution: objective 18.0000000
## Presolve removed 1 rows and 1 columns
## Presolve time: 0.00s
## Presolved: 1 rows, 3 columns, 3 nonzeros
## Variable types: 0 continuous, 3 integer (0 binary)
## 
## Root relaxation: objective 6.750000e+01, 1 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0   67.50000    0    1   18.00000   67.50000   275%     -    0s
## H    0     0                      67.0000000   67.50000  0.75%     -    0s
##      0     0   67.50000    0    1   67.00000   67.50000  0.75%     -    0s
## 
## Explored 1 nodes (1 simplex iterations) in 0.01 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 2: 67 18 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 6.700000000000e+01, best bound 6.700000000000e+01, gap 0.0000%
## [1] "RMP Objective Value and New Upper bound: 67"
## [1] "Value of z: 0"
## [1] "Value of x: 0" "Value of x: 2" "Value of x: 5"
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 2 columns and 4 nonzeros
## Model fingerprint: 0xe467fd43
## Coefficient statistics:
##   Matrix range     [3e+00, 4e+01]
##   Objective range  [1e+00, 2e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [5e+00, 7e+00]
## Presolve removed 2 rows and 2 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0   -8.2857143e+29   0.000000e+00   1.657143e+00      0s
## Extra simplex iterations after uncrush: 2
## 
## Solved in 2 iterations and 0.00 seconds
## Unbounded model
## [1] "DSB Objective Value and New Lower bound: "
## [1] "Value of u: "
## [1] "Value of y: "
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 2 columns and 6 nonzeros
## Model fingerprint: 0x9e81c4c8
## Coefficient statistics:
##   Matrix range     [1e+00, 4e+01]
##   Objective range  [1e+00, 2e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 1e+00]
## Presolve removed 3 rows and 2 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0   -7.6315789e-01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective -7.631578947e-01
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 4 columns and 8 nonzeros
## Model fingerprint: 0x349c4518
## Variable types: 1 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [2e-01, 4e+00]
##   Objective range  [1e+00, 1e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+01, 2e+01]
## Found heuristic solution: objective 18.0000000
## Presolve removed 1 rows and 1 columns
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 6 nonzeros
## Variable types: 0 continuous, 3 integer (0 binary)
## 
## Root relaxation: objective 6.750000e+01, 1 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0   67.50000    0    1   18.00000   67.50000   275%     -    0s
## H    0     0                      58.0000000   67.50000  16.4%     -    0s
##      0     0   58.00000    0    2   58.00000   58.00000  0.00%     -    0s
## 
## Cutting planes:
##   Gomory: 1
##   MIR: 1
## 
## Explored 1 nodes (3 simplex iterations) in 0.01 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 2: 58 18 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 5.800000000000e+01, best bound 5.800000000000e+01, gap 0.0000%
## [1] "RMP Objective Value and New Upper bound: 58"
## [1] "Value of z: 0"
## [1] "Value of x: 1" "Value of x: 4" "Value of x: 4"
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 2 columns and 4 nonzeros
## Model fingerprint: 0xb674b004
## Coefficient statistics:
##   Matrix range     [3e+00, 4e+01]
##   Objective range  [1e+00, 7e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [5e+00, 7e+00]
## Presolve time: 0.00s
## 
## Solved in 0 iterations and 0.00 seconds
## Infeasible or unbounded model
## [1] "DSB Objective Value and New Lower bound: "
## [1] "Value of u: "
## [1] "Value of y: "
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 2 columns and 6 nonzeros
## Model fingerprint: 0x56bfae6b
## Coefficient statistics:
##   Matrix range     [1e+00, 4e+01]
##   Objective range  [1e+00, 7e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 1e+00]
## Presolve removed 3 rows and 2 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0   -1.0000000e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective -1.000000000e+00
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 4 rows, 4 columns and 11 nonzeros
## Model fingerprint: 0x6b07280a
## Variable types: 1 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [2e-01, 1e+01]
##   Objective range  [1e+00, 1e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+01, 8e+01]
## Found heuristic solution: objective 18.0000000
## Presolve removed 1 rows and 1 columns
## Presolve time: 0.00s
## Presolved: 3 rows, 3 columns, 9 nonzeros
## Variable types: 0 continuous, 3 integer (0 binary)
## 
## Root relaxation: objective 6.750000e+01, 1 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0   67.50000    0    1   18.00000   67.50000   275%     -    0s
## H    0     0                      57.0000000   67.50000  18.4%     -    0s
## H    0     0                      58.0000000   67.50000  16.4%     -    0s
## 
## Cutting planes:
##   Gomory: 1
##   MIR: 1
## 
## Explored 1 nodes (1 simplex iterations) in 0.00 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 3: 58 57 18 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 5.800000000000e+01, best bound 5.800000000000e+01, gap 0.0000%
## [1] "RMP Objective Value and New Upper bound: 58"
## [1] "Value of z: 0"
## [1] "Value of x: 0" "Value of x: 6" "Value of x: 4"
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 2 columns and 4 nonzeros
## Model fingerprint: 0x5530e933
## Coefficient statistics:
##   Matrix range     [3e+00, 4e+01]
##   Objective range  [1e+01, 1e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [5e+00, 7e+00]
## Presolve removed 2 rows and 2 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    0.0000000e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  0.000000000e+00
## [1] "DSB Objective Value and New Lower bound: 58"
## [1] "Value of u: 0.208333333333333" "Value of u: 0"                
## [1] "Value of y: 0" "Value of y: 0"
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 5 rows, 4 columns and 15 nonzeros
## Model fingerprint: 0x34e5ff52
## Variable types: 1 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [2e-01, 1e+01]
##   Objective range  [1e+00, 1e+01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+01, 8e+01]
## Found heuristic solution: objective 18.0000000
## Presolve removed 2 rows and 1 columns
## Presolve time: 0.00s
## Presolved: 3 rows, 3 columns, 9 nonzeros
## Variable types: 0 continuous, 3 integer (0 binary)
## 
## Root relaxation: objective 6.750000e+01, 1 iterations, 0.00 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0   67.50000    0    1   18.00000   67.50000   275%     -    0s
## H    0     0                      57.0000000   67.50000  18.4%     -    0s
## H    0     0                      58.0000000   67.50000  16.4%     -    0s
## 
## Cutting planes:
##   Gomory: 1
##   MIR: 1
## 
## Explored 1 nodes (1 simplex iterations) in 0.00 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 3: 58 57 18 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 5.800000000000e+01, best bound 5.800000000000e+01, gap 0.0000%
## [1] "RMP Objective Value and New Upper bound: 58"
## [1] "Value of z: 0"
## [1] "Value of x: 0" "Value of x: 6" "Value of x: 4"
```

```r
LB_list
```

```
## [1] -999999.0000      16.5566      16.5566      16.5566      58.0000
```

```r
UB_list
```

```
## [1] 999999     67     58     58     58
```

```r
u_list
```

```
##              [,1]       [,2]
## u_list 0.00000000 0.00000000
## u      0.20283019 0.03301887
## u      0.07894737 0.92105263
## u      1.00000000 0.00000000
## u      0.20833333 0.00000000
```

```r
y_list
```

```
##             [,1]     [,2]
## y_list 0.0000000 0.000000
## y      0.3773585 2.783019
## y      0.0000000 0.000000
```

```r
x_list
```

```
##        [,1] [,2] [,3]
## x_list    0    0    0
## x         0    2    5
## x         1    4    4
## x         0    6    4
## x         0    6    4
```

```r
rm(b,B,DSB,LB,LB_list,RMP,u,u_list,UB,UB_list,x,x_list,y,y_list)
```


```r
#Column Generation Algorithm
#Cutting Stock Problem
#minimize number of rods used (x). Satisfy demand for 44 81 cm pieces, 3 70 cm pieces, and 48 68 cm pieces
#min x1 + x2 + x3
#s.t. x1 >= 44; x2 >=3; x3 >= 48
#x >=0,


LMP <- list()

LMP$A          <- matrix(c(1,0,0,
                           0,1,0,
                           0,0,1), nrow=3, byrow=T)
LMP$obj        <- c(1,1,1)
LMP$modelsense <- 'min'
LMP$rhs        <- c(44,3,48)
LMP$sense      <- c('>')

result <- gurobi(LMP)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0xcd2bfab3
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Presolve removed 3 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    9.5000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  9.500000000e+01
```

```r
k <- -1

while(k < 0){


KSP <- list()

KSP$A          <- matrix(c(81,70,68), nrow=1, byrow=T)
KSP$obj        <- result$pi
KSP$modelsense <- 'max'
KSP$rhs        <- c(218)
KSP$sense      <- c('<')
KSP$vtype      <- c('I','I','I')

result <- gurobi(KSP)

k <- 1 - sum(result$x*KSP$obj)

B <- as.matrix(result$x)

LMP$A <- cbind(LMP$A,B)

LMP$obj <- c(LMP$obj,1)

result <- gurobi(LMP)

print(paste('LMP Objective Value', result$objval))

print(paste('Sum of Reduced Cost:', k))

print(LMP$A)

print(t(as.matrix(result$x)))

}
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0x6bafdfd7
## Variable types: 0 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [7e+01, 8e+01]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+02, 2e+02]
## Found heuristic solution: objective 2.0000000
## Presolve removed 1 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 2: 3 2 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 3.000000000000e+00, best bound 3.000000000000e+00, gap 0.0000%
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 4 columns and 4 nonzeros
## Model fingerprint: 0x4bac7a68
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Presolve removed 3 rows and 4 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    6.3000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  6.300000000e+01
## [1] "LMP Objective Value 63"
## [1] "Sum of Reduced Cost: -2"
##      [,1] [,2] [,3] [,4]
## [1,]    1    0    0    0
## [2,]    0    1    0    0
## [3,]    0    0    1    3
##      [,1] [,2] [,3] [,4]
## [1,]   44    3    0   16
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0xe2a49abb
## Variable types: 0 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [7e+01, 8e+01]
##   Objective range  [3e-01, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+02, 2e+02]
## Found heuristic solution: objective 2.0000000
## Presolve removed 1 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 2: 3 2 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 3.000000000000e+00, best bound 3.000000000000e+00, gap 0.0000%
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 5 columns and 5 nonzeros
## Model fingerprint: 0x46707112
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Presolve removed 3 rows and 5 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    6.1000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  6.100000000e+01
## [1] "LMP Objective Value 61"
## [1] "Sum of Reduced Cost: -2"
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    0    0    0    0
## [2,]    0    1    0    0    3
## [3,]    0    0    1    3    0
##      [,1] [,2] [,3] [,4] [,5]
## [1,]   44    0    0   16    1
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0x0bae0166
## Variable types: 0 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [7e+01, 8e+01]
##   Objective range  [3e-01, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+02, 2e+02]
## Found heuristic solution: objective 2.0000000
## Presolve removed 1 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 1: 2 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 2.000000000000e+00, best bound 2.000000000000e+00, gap 0.0000%
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 6 columns and 6 nonzeros
## Model fingerprint: 0x461ec1df
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Presolve removed 3 rows and 6 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    3.9000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  3.900000000e+01
## [1] "LMP Objective Value 39"
## [1] "Sum of Reduced Cost: -1"
##      [,1] [,2] [,3] [,4] [,5] [,6]
## [1,]    1    0    0    0    0    2
## [2,]    0    1    0    0    3    0
## [3,]    0    0    1    3    0    0
##      [,1] [,2] [,3] [,4] [,5] [,6]
## [1,]    0    0    0   16    1   22
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0x1cca384f
## Variable types: 0 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [7e+01, 8e+01]
##   Objective range  [3e-01, 5e-01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+02, 2e+02]
## Found heuristic solution: objective 1.0000000
## Presolve removed 1 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 2: 1.16667 1 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 1.166666666667e+00, best bound 1.166666666667e+00, gap 0.0000%
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 7 columns and 8 nonzeros
## Model fingerprint: 0xaf053821
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Presolve removed 1 rows and 4 columns
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 4 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    1.0000000e+00   3.400000e+01   0.000000e+00      0s
##        2    3.5000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 2 iterations and 0.00 seconds
## Optimal objective  3.500000000e+01
## [1] "LMP Objective Value 35"
## [1] "Sum of Reduced Cost: -0.166666666666667"
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7]
## [1,]    1    0    0    0    0    2    1
## [2,]    0    1    0    0    3    0    0
## [3,]    0    0    1    3    0    0    2
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7]
## [1,]    0    0    0    0    1   10   24
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 1 rows, 3 columns and 3 nonzeros
## Model fingerprint: 0x3553a8a2
## Variable types: 0 continuous, 3 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [7e+01, 8e+01]
##   Objective range  [3e-01, 5e-01]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [2e+02, 2e+02]
## Found heuristic solution: objective 1.0000000
## Presolve removed 1 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 1 (of 20 available processors)
## 
## Solution count 1: 1 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 1.000000000000e+00, best bound 1.000000000000e+00, gap 0.0000%
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 8 columns and 9 nonzeros
## Model fingerprint: 0xebe33484
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Presolve removed 1 rows and 5 columns
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 4 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    1.0000000e+00   3.400000e+01   0.000000e+00      0s
##        2    3.5000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 2 iterations and 0.00 seconds
## Optimal objective  3.500000000e+01
## [1] "LMP Objective Value 35"
## [1] "Sum of Reduced Cost: 0"
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8]
## [1,]    1    0    0    0    0    2    1    2
## [2,]    0    1    0    0    3    0    0    0
## [3,]    0    0    1    3    0    0    2    0
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8]
## [1,]    0    0    0    0    1   10   24    0
```

```r
LMP$vtype      <- rep('I', ncol(LMP$A))

result <- gurobi(LMP)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 8 columns and 9 nonzeros
## Model fingerprint: 0xe1a5582b
## Variable types: 0 continuous, 8 integer (0 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [3e+00, 5e+01]
## Found heuristic solution: objective 35.0000000
## Presolve removed 1 rows and 5 columns
## Presolve time: 0.00s
## Presolved: 2 rows, 3 columns, 4 nonzeros
## Variable types: 0 continuous, 3 integer (0 binary)
## 
## Root relaxation: cutoff, 0 iterations, 0.00 seconds
## 
## Explored 0 nodes (0 simplex iterations) in 0.00 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 1: 35 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 3.500000000000e+01, best bound 3.500000000000e+01, gap 0.0000%
```

```r
print(paste('Integer Objective Value', result$objval))
```

```
## [1] "Integer Objective Value 35"
```

```r
LMP$A
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8]
## [1,]    1    0    0    0    0    2    1    2
## [2,]    0    1    0    0    3    0    0    0
## [3,]    0    0    1    3    0    0    2    0
```

```r
t(as.matrix(result$x))
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8]
## [1,]    0    0    0    0    1    0   24   10
```

```r
rm(B,k,KSP,LMP,result)
```



Add a new chunk by clicking the *Insert Chunk* button on the toolbar or by pressing *Ctrl+Alt+I*.

When you save the notebook, an HTML file containing the code and output will be saved alongside it (click the *Preview* button or press *Ctrl+Shift+K* to preview the HTML file).

The preview shows you a rendered HTML copy of the contents of the editor. Consequently, unlike *Knit*, *Preview* does not run any R code chunks. Instead, the output of the chunk when it was last run in the editor is displayed.
