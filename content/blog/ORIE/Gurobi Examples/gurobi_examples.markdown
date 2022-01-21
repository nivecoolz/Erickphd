---
title: "Gurobi Basic LP/MIP Examples"
author: "Erick Jones"
date: 2020-01-05
categories: ["ORIE Basics"]
tags: ["R Markdown", "LP", "MIP", "Gurobi"]
--- 


This post explores how to use Gurobi to solve LPs and MIPs. 
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

This example formulates and solves the following simple LP model:
`\(max: x + 2y + 3z\)`
 
  subject to
      `\(x +   y  \leq 1\)`
      `\(y +   z \leq 1\)`


```r
model <- list()

model$A          <- matrix(c(1,1,0,0,1,1), nrow=2, byrow=T)
model$obj        <- c(1,2,3)
model$modelsense <- 'max'
model$rhs        <- c(1,1)
model$sense      <- c('<', '<')

result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 4 nonzeros
## Model fingerprint: 0x8d9b7076
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 3e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 1e+00]
## Presolve removed 2 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    4.0000000e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  4.000000000e+00
```

```r
print(result$objval)
```

```
## [1] 4
```

```r
print(result$x)
```

```
## [1] 1 0 1
```

```r
# Second option for A - as a sparseMatrix (using the Matrix package)...

model$A <- spMatrix(2, 3, c(1, 1, 2, 2), c(1, 2, 2, 3), c(1, 1, 1, 1))

params <- list(Method=2, TimeLimit=100)

result <- gurobi(model, params)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 4 nonzeros
## Model fingerprint: 0x8d9b7076
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 3e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 1e+00]
## Presolve removed 2 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    4.0000000e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  4.000000000e+00
```

```r
print(result$objval)
```

```
## [1] 4
```

```r
print(result$x)
```

```
## [1] 1 0 1
```

```r
# Third option for A - as a sparse triplet matrix (using the slam package)...

model$A <- simple_triplet_matrix(c(1, 1, 2, 2), c(1, 2, 2, 3), c(1, 1, 1, 1))

params <- list(Method=3, TimeLimit=100)

result <- gurobi(model, params)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 2 rows, 3 columns and 4 nonzeros
## Model fingerprint: 0x8d9b7076
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 3e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [1e+00, 1e+00]
## Presolve removed 2 rows and 3 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    4.0000000e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  4.000000000e+00
```

```r
print(result$objval)
```

```
## [1] 4
```

```r
print(result$x)
```

```
## [1] 1 0 1
```

```r
# Clear space
rm(result, params, model)
```

This example formulates and solves the following simple MIP model:
`\(max: x +   y + 2 z\)`
subject to
  `\(x + 2 y + 3 z \leq 4\)`
  `\(x +   y       \geq 1\)`
  x, y, z binary


```r
model <- list()

model$A          <- matrix(c(1,2,3,1,1,0), nrow=2, ncol=3, byrow=T)
model$obj        <- c(1,1,2)
model$modelsense <- 'max'
model$rhs        <- c(4,1)
model$sense      <- c('<', '>')
model$vtype      <- c('B','B','B')

params <- list(OutputFlag=0)

result <- gurobi(model, params)

print('Solution:')
```

```
## [1] "Solution:"
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
## [1] 1 0 1
```

```r
# Clear space
rm(model, result, params)
```



```r
# Solve the classic diet model, showing how to add constraints
# to an existing model.


# define primitive data
Categories      <- c('calories', 'protein', 'fat', 'sodium')
nCategories     <- length(Categories)
minNutrition    <- c(     1800 ,       91 ,    0 ,       0 )
maxNutrition    <- c(     2200 ,      Inf ,   65 ,    1779 )

Foods           <- c('hamburger', 'chicken', 'hot dog', 'fries', 'macaroni',
                     'pizza', 'salad', 'milk', 'ice cream')
nFoods          <- length(Foods)
cost            <- c(2.49, 2.89, 1.50, 1.89, 2.09, 1.99, 2.49, 0.89, 1.59)
nutritionValues <- c( 410, 24, 26 ,  730,
                      420, 32, 10 , 1190,
                      560, 20, 32 , 1800,
                      380,  4, 19 ,  270,
                      320, 12, 10 ,  930,
                      320, 15, 12 ,  820,
                      320, 31, 12 , 1230,
                      100,  8, 2.5,  125,
                      330,  8, 10 ,  180 )

#Each constraint is basically the Nutrion = sum(food*nut/food)
#Could have just made nutrition the RHS, but it works as a bounded variable because you need both upper and lower and it shrinks the amount of equations
#Objective min cost of food

# Build model
model     <- list()
#spMatrix tells you where to put the non zero values in matrix i,j is the location and x are teh values for each pair
model$A   <- spMatrix(nCategories, nCategories + nFoods,
               i = c(mapply(rep,1:4,1+nFoods)),
               j = c(1, (nCategories+1):(nCategories+nFoods),
                     2, (nCategories+1):(nCategories+nFoods),
                     3, (nCategories+1):(nCategories+nFoods),
                     4, (nCategories+1):(nCategories+nFoods) ),
               x = c(-1.0, nutritionValues[1 + nCategories*(0:(nFoods-1))],
                     -1.0, nutritionValues[2 + nCategories*(0:(nFoods-1))],
                     -1.0, nutritionValues[3 + nCategories*(0:(nFoods-1))],
                     -1.0, nutritionValues[4 + nCategories*(0:(nFoods-1))] ))
model$obj         <- c(rep(0, nCategories), cost)
model$lb          <- c(minNutrition, rep(0, nFoods))
model$ub          <- c(maxNutrition, rep(Inf, nFoods))
model$varnames    <- c(Categories, Foods)
model$rhs         <- rep(0,nCategories)
model$sense       <- rep('=',nCategories)
model$constrnames <- Categories
model$modelname   <- 'diet'
model$modelsense  <- 'min'

# display results
printSolution <- function(model, res, nCategories, nFoods) {
  if (res$status == 'OPTIMAL') {
    cat('\nCost: ',res$objval,'\nBuy:\n')
    for (j in nCategories + 1:nFoods) {
      if (res$x[j] > 1e-4) {
        cat(format(model$varnames[j],justify='left',width=10),':',
            format(res$x[j],justify='right',width=10,nsmall=2),'\n')
      }
    }
    cat('\nNutrition:\n')
    for (j in 1:nCategories) {
      cat(format(model$varnames[j],justify='left',width=10),':',
          format(res$x[j],justify='right',width=10,nsmall=2),'\n')
    }
  } else {
    cat('No solution\n')
  }
}

# Optimize
res <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 4 rows, 13 columns and 40 nonzeros
## Model fingerprint: 0xff20f824
## Coefficient statistics:
##   Matrix range     [1e+00, 2e+03]
##   Objective range  [9e-01, 3e+00]
##   Bounds range     [7e+01, 2e+03]
##   RHS range        [0e+00, 0e+00]
## Presolve removed 0 rows and 3 columns
## Presolve time: 0.00s
## Presolved: 4 rows, 10 columns, 37 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    0.0000000e+00   1.472500e+02   0.000000e+00      0s
##        4    1.1828861e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 4 iterations and 0.00 seconds
## Optimal objective  1.182886111e+01
```

```r
printSolution(model, res, nCategories, nFoods)
```

```
## 
## Cost:  11.82886 
## Buy:
## hamburger  :  0.6045139 
## milk       :   6.970139 
## ice cream  :   2.591319 
## 
## Nutrition:
## calories   :    1800.00 
## protein    :      91.00 
## fat        :    59.0559 
## sodium     :    1779.00
```

```r
# Adding constraint: at most 6 servings of dairy
# this is the matrix part of the constraint
B <- spMatrix(1, nCategories + nFoods,
              i = rep(1,2),
              j = (nCategories+c(8,9)),
              x = rep(1,2))
# append B to A
model$A           <- rbind(model$A,       B)
# extend row-related vectors
model$constrnames <- c(model$constrnames, 'limit_dairy')
model$rhs         <- c(model$rhs,         10)
model$sense       <- c(model$sense,       '<')

# Optimize
res <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 5 rows, 13 columns and 42 nonzeros
## Model fingerprint: 0xc012cadd
## Coefficient statistics:
##   Matrix range     [1e+00, 2e+03]
##   Objective range  [9e-01, 3e+00]
##   Bounds range     [7e+01, 2e+03]
##   RHS range        [1e+01, 1e+01]
## Presolve removed 0 rows and 3 columns
## Presolve time: 0.00s
## Presolved: 5 rows, 10 columns, 39 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    0.0000000e+00   1.472500e+02   0.000000e+00      0s
##        4    1.1828861e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 4 iterations and 0.00 seconds
## Optimal objective  1.182886111e+01
```

```r
printSolution(model, res, nCategories, nFoods)
```

```
## 
## Cost:  11.82886 
## Buy:
## hamburger  :  0.6045139 
## milk       :   6.970139 
## ice cream  :   2.591319 
## 
## Nutrition:
## calories   :    1800.00 
## protein    :      91.00 
## fat        :    59.0559 
## sodium     :    1779.00
```

```r
# Clear space
#rm(res, model)
```



```r
#Facility Location Problem (MIP)

# define primitive data
nPlants     <- 5
nWarehouses <- 4
# Warehouse demand in thousands of units
Demand      <- c(15, 18, 14, 20)
# Plant capacity in thousands of units 
Capacity    <- c(20, 22, 17, 19, 18)
# Fixed costs for each plant 
FixedCosts  <- c( 12000, 15000, 17000, 13000, 16000)
# Transportation costs per thousand units 
TransCosts  <- c(4000, 2000, 3000, 2500, 4500,
                 2500, 2600, 3400, 3000, 4000,
                 1200, 1800, 2600, 4100, 3000,
                 2200, 2600, 3100, 3700, 3200 )

flowidx <- function(w, p) {nPlants * (w-1) + p}

# Build model
model <- list()
model$modelname <- 'facility'
model$modelsense <- 'min'

# initialize data for variables
model$lb       <- 0
model$ub       <- c(rep(1, nPlants),   rep(Inf, nPlants * nWarehouses))
model$vtype    <- c(rep('B', nPlants), rep('C', nPlants * nWarehouses))
model$obj      <- c(FixedCosts, TransCosts)
model$varnames <- c(paste0(rep('Open',nPlants),1:nPlants),
                    sprintf('Trans%d,%d',
                            c(mapply(rep,1:nWarehouses,nPlants)),
                            1:nPlants))

# build production constraint matrix
#uses custom functions to fill out matrix, a bit out my wheelhouse
A1 <- spMatrix(nPlants, nPlants, i = c(1:nPlants), j = (1:nPlants), x = -Capacity)
A2 <- spMatrix(nPlants, nPlants * nWarehouses,
               i = c(mapply(rep, 1:nPlants, nWarehouses)),
               j = mapply(flowidx,1:nWarehouses,c(mapply(rep,1:nPlants,nWarehouses))),
               x = rep(1, nWarehouses * nPlants))
A3 <- spMatrix(nWarehouses, nPlants)
A4 <- spMatrix(nWarehouses, nPlants * nWarehouses,
               i = c(mapply(rep, 1:nWarehouses, nPlants)),
               j = mapply(flowidx,c(mapply(rep,1:nWarehouses,nPlants)),1:nPlants),
               x = rep(1, nPlants * nWarehouses))
model$A           <- rbind(cbind(A1, A2), cbind(A3, A4))
model$rhs         <- c(rep(0, nPlants),   Demand)
model$sense       <- c(rep('<', nPlants), rep('=', nWarehouses))
model$constrnames <- c(sprintf('Capacity%d',1:nPlants),
                       sprintf('Demand%d',1:nWarehouses))

# Save model
gurobi_write(model,'facilityR.lp')
```

```
## NULL
```

```r
# Guess at the starting point: close the plant with the highest fixed
# costs; open all others first open all plants
model$start <- c(rep(1,nPlants),rep(NA, nPlants * nWarehouses))

# find most expensive plant, and close it in mipstart
cat('Initial guess:\n')
```

```
## Initial guess:
```

```r
worstidx <- which.max(FixedCosts)
model$start[worstidx] <- 0
cat('Closing plant',worstidx,'\n')
```

```
## Closing plant 3
```

```r
# set parameters
params <- list()
params$method <- 2

# Optimize
res <- gurobi(model, params)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 9 rows, 25 columns and 45 nonzeros
## Model fingerprint: 0x36b45dc0
## Variable types: 20 continuous, 5 integer (5 binary)
## Coefficient statistics:
##   Matrix range     [1e+00, 2e+01]
##   Objective range  [1e+03, 2e+04]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [1e+01, 2e+01]
## 
## User MIP start produced solution with objective 210500 (0.00s)
## Loaded user MIP start with objective 210500
## 
## Presolve time: 0.00s
## Presolved: 9 rows, 25 columns, 45 nonzeros
## Variable types: 20 continuous, 5 integer (5 binary)
## Root barrier log...
## 
## Ordering time: 0.00s
## 
## Barrier statistics:
##  AA' NZ     : 2.000e+01
##  Factor NZ  : 4.500e+01
##  Factor Ops : 2.850e+02 (less than 1 second per iteration)
##  Threads    : 1
## 
##                   Objective                Residual
## Iter       Primal          Dual         Primal    Dual     Compl     Time
##    0   7.94290841e+05 -2.24842916e+05  7.25e+00 3.75e+03  2.69e+04     0s
##    1   2.34432856e+05  7.59319096e+04  1.78e-15 3.64e-12  3.17e+03     0s
##    2   2.10232015e+05  1.89880475e+05  8.88e-16 4.01e-12  4.07e+02     0s
##    3   2.00964341e+05  1.98582137e+05  9.77e-15 2.79e-12  4.76e+01     0s
##    4   1.99878036e+05  1.99804970e+05  2.46e-13 3.19e-12  1.46e+00     0s
##    5   1.99833638e+05  1.99832960e+05  3.14e-13 1.82e-12  1.36e-02     0s
##    6   1.99833333e+05  1.99833333e+05  1.47e-14 2.86e-12  1.39e-08     0s
##    7   1.99833333e+05  1.99833333e+05  7.10e-15 2.73e-12  1.39e-14     0s
## 
## Barrier solved model in 7 iterations and 0.01 seconds
## Optimal objective 1.99833333e+05
## 
## 
## Root relaxation: objective 1.998333e+05, 6 iterations, 0.01 seconds
## 
##     Nodes    |    Current Node    |     Objective Bounds      |     Work
##  Expl Unexpl |  Obj  Depth IntInf | Incumbent    BestBd   Gap | It/Node Time
## 
##      0     0 199833.333    0    1 210500.000 199833.333  5.07%     -    0s
##      0     0 200252.941    0    1 210500.000 200252.941  4.87%     -    0s
##      0     0 208000.000    0    1 210500.000 208000.000  1.19%     -    0s
## 
## Cutting planes:
##   MIR: 1
##   Flow cover: 3
## 
## Explored 1 nodes (11 simplex iterations) in 0.02 seconds
## Thread count was 20 (of 20 available processors)
## 
## Solution count 1: 210500 
## 
## Optimal solution found (tolerance 1.00e-04)
## Best objective 2.105000000000e+05, best bound 2.105000000000e+05, gap 0.0000%
```

```r
# Print solution
if (res$status == 'OPTIMAL') {
  cat('\nTotal Costs:',res$objval,'\nsolution:\n')
  cat('Facilities:', model$varnames[which(res$x[1:nPlants]>1e-5)], '\n')
  active <- nPlants + which(res$x[(nPlants+1):(nPlants*(nWarehouses+1))] > 1e-5)
  cat('Flows: ')
  cat(sprintf('%s=%g ',model$varnames[active], res$x[active]), '\n')
  rm(active)
} else {
  cat('No solution\n')
}
```

```
## 
## Total Costs: 210500 
## solution:
## Facilities: Open1 Open2 Open4 Open5 
## Flows: Trans1,2=14  Trans1,4=1  Trans2,4=18  Trans3,1=14  Trans4,1=6  Trans4,2=8  Trans4,5=6
```

```r
# Clear space
rm(res, model, params, A1, A2, A3, A4)
```


```r
# Assign workers to shifts; each worker may or may not be available on a
# particular day. If the problem cannot be solved, use IIS iteratively to
# find all conflicting constraints.


# Function to display results
printsolution <- function(result) {
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
  }
}

# define data
nShifts  <- 14
nWorkers <-  7
nVars    <- nShifts * nWorkers
varIdx   <- function(w,s) {s+(w-1)*nShifts}

Shifts  <- c('Mon1', 'Tue2', 'Wed3', 'Thu4', 'Fri5', 'Sat6', 'Sun7',
             'Mon8', 'Tue9', 'Wed10', 'Thu11', 'Fri12', 'Sat13', 'Sun14')
Workers <- c( 'Amy', 'Bob', 'Cathy', 'Dan', 'Ed', 'Fred', 'Gu' )

pay     <- c(10, 12, 10, 8, 8, 9, 11 )

shiftRequirements <- c(3, 2, 4, 4, 5, 6, 5, 2, 2, 3, 4, 6, 7, 5 )

availability <- list( c( 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 0, 1, 0 ),
                      c( 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1 ),
                      c( 1, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 1 ),
                      c( 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 ) )

# Set-up environment
env <- list()
env$logfile <- 'workforce2.log'

# Build model
model            <- list()
model$modelname  <- 'workforce2'
model$modelsense <- 'min'

# Initialize assignment decision variables:
#    x[w][s] == 1 if worker w is assigned
#    to shift s. Since an assignment model always produces integer
#    solutions, we use continuous variables and solve as an LP.
model$lb       <- 0
model$ub       <- rep(1, nVars)
model$obj      <- rep(0, nVars)
model$varnames <- rep('',nVars)
for (w in 1:nWorkers) {
  for (s in 1:nShifts) {
    model$varnames[varIdx(w,s)] = paste0(Workers[w],'.',Shifts[s])
    model$obj[varIdx(w,s)]      = pay[w]
    if (availability[[w]][s] == 0) model$ub[varIdx(w,s)] = 0
  }
}

# Set-up shift-requirements constraints
model$A           <- spMatrix(nShifts,nVars,
                      i = c(mapply(rep,1:nShifts,nWorkers)),
                      j = mapply(varIdx,1:nWorkers,
                                 mapply(rep,1:nShifts,nWorkers)),
                      x = rep(1,nShifts * nWorkers))
model$sense       <- rep('=',nShifts)
model$rhs         <- shiftRequirements
model$constrnames <- Shifts

# Save model
gurobi_write(model,'workforce2.lp', env)
```

```
## NULL
```

```r
# Optimize
result <- gurobi(model, env = env)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 14 rows, 98 columns and 98 nonzeros
## Model fingerprint: 0xbddc1063
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [8e+00, 1e+01]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 1 rows and 33 columns
## Presolve time: 0.00s
## 
## Solved in 0 iterations and 0.00 seconds
## Infeasible model
```

```r
# Display results
if (result$status == 'OPTIMAL') {
# The code may enter here if you change some of the data... otherwise
# this will never be executed.
  printsolution(result);
} else if (result$status == 'INFEASIBLE') {
# We will loop until we reduce a model that can be solved
  numremoved <- 0 
  while(result$status == 'INFEASIBLE') {
    iis               <- gurobi_iis(model, env = env)
    keep              <- (!iis$Arows)
    cat('Removing rows',model$constrnames[iis$Arows],'...\n')
    model$A           <- model$A[keep,,drop = FALSE]
    model$sense       <- model$sense[keep]
    model$rhs         <- model$rhs[keep]
    model$constrnames <- model$constrnames[keep]
    numremoved        <- numremoved + 1
    gurobi_write(model, paste0('workforce2-',numremoved,'.lp'), env)
    result            <- gurobi(model, env = env)
  }
  printsolution(result)
  rm(iis)
} else {
# Just to handle user interruptions or other problems
  cat('Unexpected status',result$status,'\nEnding now\n')
}
```

```
## 
## IIS computed: 1 constraints and 7 bounds
## IIS runtime: 0.00 seconds
## Removing rows Thu4 ...
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 13 rows, 98 columns and 91 nonzeros
## Model fingerprint: 0xd0134630
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [8e+00, 1e+01]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 1 rows and 35 columns
## Presolve time: 0.00s
## 
## Solved in 0 iterations and 0.00 seconds
## Infeasible model
## 
## IIS computed: 1 constraints and 7 bounds
## IIS runtime: 0.00 seconds
## Removing rows Sat6 ...
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 12 rows, 98 columns and 84 nonzeros
## Model fingerprint: 0x50426357
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [8e+00, 1e+01]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 1 rows and 39 columns
## Presolve time: 0.00s
## 
## Solved in 0 iterations and 0.00 seconds
## Infeasible model
## 
## IIS computed: 1 constraints and 7 bounds
## IIS runtime: 0.00 seconds
## Removing rows Sun7 ...
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 11 rows, 98 columns and 77 nonzeros
## Model fingerprint: 0x23a14e24
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [8e+00, 1e+01]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 1 rows and 43 columns
## Presolve time: 0.00s
## 
## Solved in 0 iterations and 0.00 seconds
## Infeasible model
## 
## IIS computed: 1 constraints and 7 bounds
## IIS runtime: 0.00 seconds
## Removing rows Fri12 ...
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 10 rows, 98 columns and 70 nonzeros
## Model fingerprint: 0xa8fcf019
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [8e+00, 1e+01]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 10 rows and 98 columns
## Presolve time: 0.00s
## Presolve: All rows and columns removed
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    3.3500000e+02   0.000000e+00   1.480000e+02      0s
## Extra simplex iterations after uncrush: 5
##        5    3.3500000e+02   0.000000e+00   0.000000e+00      0s
## 
## Solved in 5 iterations and 0.00 seconds
## Optimal objective  3.350000000e+02
## The optimal objective is 335 
## Schedule:
## 	 Mon1 :Ed  Fred  Gu  
## 	 Tue2 :Dan  Ed  
## 	 Wed3 :Amy  Dan  Ed  Fred  
## 	 Thu4 :
## 	 Fri5 :Amy  Cathy  Dan  Ed  Gu  
## 	 Sat6 :
## 	 Sun7 :
## 	 Mon8 :Dan  Ed  
## 	 Tue9 :Dan  Ed  
## 	 Wed10 :Amy  Cathy  Dan  
## 	 Thu11 :Amy  Cathy  Dan  Ed  
## 	 Fri12 :
## 	 Sat13 :Amy  Bob  Cathy  Dan  Ed  Fred  Gu  
## 	 Sun14 :Amy  Cathy  Dan  Ed  Fred
```

```r
#Clear space
rm(model, env, availability, Shifts, Workers, pay, shiftRequirements, result)
```


```r
# Assign workers to shifts; each worker may or may not be available on a
# particular day. If the problem cannot be solved, relax the model
# to determine which constraints cannot be satisfied, and how much
# they need to be relaxed.


# Function to display results
printsolution <- function(result) {
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
  }
}

# define data
nShifts  <- 14
nWorkers <-  7
nVars    <- nShifts * nWorkers
varIdx   <- function(w,s) {s+(w-1)*nShifts}

Shifts  <- c('Mon1', 'Tue2', 'Wed3', 'Thu4', 'Fri5', 'Sat6', 'Sun7',
             'Mon8', 'Tue9', 'Wed10', 'Thu11', 'Fri12', 'Sat13', 'Sun14')
Workers <- c( 'Amy', 'Bob', 'Cathy', 'Dan', 'Ed', 'Fred', 'Gu' )

pay     <- c(10, 12, 10, 8, 8, 9, 11 )

shiftRequirements <- c(3, 2, 4, 4, 5, 6, 5, 2, 2, 3, 4, 6, 7, 5 )

availability <- list( c( 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 0, 1, 0 ),
                      c( 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1 ),
                      c( 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1 ),
                      c( 1, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 1 ),
                      c( 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 ) )

# Set-up environment
env <- list()
env$logfile <- 'workforce3.log'

# Build model
model            <- list()
model$modelname  <- 'workforce3'
model$modelsense <- 'min'

# Initialize assignment decision variables:
#    x[w][s] == 1 if worker w is assigned
#    to shift s. Since an assignment model always produces integer
#    solutions, we use continuous variables and solve as an LP.
model$lb       <- 0
model$ub       <- rep(1, nVars)
model$obj      <- rep(0, nVars)
model$varnames <- rep('',nVars)
for (w in 1:nWorkers) {
  for (s in 1:nShifts) {
    model$varnames[varIdx(w,s)] = paste0(Workers[w],'.',Shifts[s])
    model$obj[varIdx(w,s)]      = pay[w]
    if (availability[[w]][s] == 0) model$ub[varIdx(w,s)] = 0
  }
}

# Set-up shift-requirements constraints
model$A           <- spMatrix(nShifts,nVars,
                      i = c(mapply(rep,1:nShifts,nWorkers)),
                      j = mapply(varIdx,1:nWorkers,
                                 mapply(rep,1:nShifts,nWorkers)),
                      x = rep(1,nShifts * nWorkers))
model$sense       <- rep('=',nShifts)
model$rhs         <- shiftRequirements
model$constrnames <- Shifts

# Save model
gurobi_write(model,'workforce3.lp', env)
```

```
## NULL
```

```r
# Optimize
result <- gurobi(model, env = env)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 14 rows, 98 columns and 98 nonzeros
## Model fingerprint: 0xbddc1063
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [8e+00, 1e+01]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 1 rows and 33 columns
## Presolve time: 0.00s
## 
## Solved in 0 iterations and 0.00 seconds
## Infeasible model
```

```r
# Display results
if (result$status == 'OPTIMAL') {
# The code may enter here if you change some of the data... otherwise
# this will never be executed.
  printsolution(result);
} else if (result$status == 'INFEASIBLE') {
# Use gurobi_feasrelax to find out which copnstraints should be relaxed
# and by how much to make the problem feasible.
  penalties     <- list()
  penalties$lb  <- Inf
  penalties$ub  <- Inf
  penalties$rhs <- rep(1,length(model$rhs))
  feasrelax     <- gurobi_feasrelax(model, 0, FALSE, penalties, env = env)
  result        <- gurobi(feasrelax$model, env = env)
  if (result$status == 'OPTIMAL') {
    printsolution(result)
    cat('Slack values:\n')
    for (j in (nVars+1):length(result$x)) {
      if(result$x[j] > 0.1)
        cat('\t',feasrelax$model$varnames[j],result$x[j],'\n')
    }
  } else {
    cat('Unexpected status',result$status,'\nEnding now\n')
  }
  rm(penalties, feasrelax)
} else {
# Just to handle user interruptions or other problems
  cat('Unexpected status',result$status,'\nEnding now\n')
}
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 14 rows, 126 columns and 126 nonzeros
## Model fingerprint: 0xa5484b98
## Coefficient statistics:
##   Matrix range     [1e+00, 1e+00]
##   Objective range  [1e+00, 1e+00]
##   Bounds range     [1e+00, 1e+00]
##   RHS range        [2e+00, 7e+00]
## Presolve removed 5 rows and 99 columns
## Presolve time: 0.00s
## Presolved: 9 rows, 27 columns, 27 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    6.0000000e+00   0.000000e+00   0.000000e+00      0s
##        0    6.0000000e+00   0.000000e+00   0.000000e+00      0s
## 
## Solved in 0 iterations and 0.00 seconds
## Optimal objective  6.000000000e+00
## The optimal objective is 6 
## Schedule:
## 	 Mon1 :Ed  Fred  Gu  
## 	 Tue2 :Bob  Ed  
## 	 Wed3 :Amy  Cathy  Fred  Gu  
## 	 Thu4 :Cathy  Ed  
## 	 Fri5 :Amy  Cathy  Dan  Ed  Gu  
## 	 Sat6 :Bob  Dan  Fred  Gu  
## 	 Sun7 :Amy  Cathy  Ed  Gu  
## 	 Mon8 :Dan  Ed  
## 	 Tue9 :Dan  Gu  
## 	 Wed10 :Amy  Dan  Gu  
## 	 Thu11 :Amy  Bob  Ed  Gu  
## 	 Fri12 :Amy  Cathy  Dan  Fred  Gu  
## 	 Sat13 :Amy  Bob  Cathy  Dan  Ed  Fred  Gu  
## 	 Sun14 :Amy  Cathy  Ed  Fred  Gu  
## Slack values:
## 	 ArtP_Thu4 2 
## 	 ArtP_Sat6 2 
## 	 ArtP_Sun7 1 
## 	 ArtP_Fri12 1
```

```r
#Clear space
rm(model, env, availability, Shifts, Workers, pay, shiftRequirements, result)
```


Add a new chunk by clicking the *Insert Chunk* button on the toolbar or by pressing *Ctrl+Alt+I*.

When you save the notebook, an HTML file containing the code and output will be saved alongside it (click the *Preview* button or press *Ctrl+Shift+K* to preview the HTML file).

The preview shows you a rendered HTML copy of the contents of the editor. Consequently, unlike *Knit*, *Preview* does not run any R code chunks. Instead, the output of the chunk when it was last run in the editor is displayed.
