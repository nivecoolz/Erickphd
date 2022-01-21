---
title: "Linear Programming Examples and Applications"
author: "Erick Jones"
date: 2020-01-03
categories: ["ORIE Basics"]
tags: ["R Markdown", "LP", "Algorithms"]
---

## Background

This post explores how to use the fundamental algorithms to solve LPs. 
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
library(MASS)
```



## Problem 1

`\(max: 2x_1 + 3x_2\)`
s.t. 
`\(-x_1 + x_2 \leq 5\)`
`\(x_1+3x_2 <= 35; x_1 \leq 20\)`



```r
#library(matlib)
#https://cran.r-project.org/web/packages/matlib/vignettes/linear-equations.html
#another method using outer function https://stackoverflow.com/questions/10199547/plotting-curves-given-by-equations-in-r


A <- matrix(c(-1, 1, 1, 1, 3, 0), 3, 2)
b <- c(5,35, 20)
#showEqn(A, b)

#c( R(A), R(cbind(A,b)) )          # show ranks

#all.equal( R(A), R(cbind(A,b)) )  # consistent?

#plotEqn(A,b, xlim = c(0,60), ylim = c(0,60))

rm(A,b)
```


## Linear Program Example 

From: http://lpsolve.sourceforge.net/5.5/formulate.htm
In this script, we don't solve the linear program but plot it in 2-d space for visualization purposes. 


```r
### set up some functions to define the constraints and the profit
# money constraint

#-x_1 + x_2 <= 5;
constraint1 = function(x1){
  x2 = (5 + x1)
  return(x2)
}

# storage constraint

#x_1+3x_2 <= 35; 
constraint2 = function(x1){
  x2 = (35 - x1)/3
  return(x2)
}

# acreage constraint

#x_1 <= 20
#constraint3 = function(x1){
#  x2 = 20 - x1
#  return(x2)
#}

# profit contours - returns barley given wheat and profit. i.e. gives us the information needed to plot a line of (wheat, barley) combinations that yield a given amount of profit

#max z = 2x_1 + 3x_2

profitContour = function(x1Array, z){
  x2 <- numeric(length(x1Array))
  for (i in 0:length(x1)){
    x2[i] = (z - 2*x1[i]) / 3
  }
  return(x2)
}


### set up data frame for plotting. Data frame will put barley in terms of wheat. Wheat will be our x axis, and barley will be our y axis.
x1 = seq(0,20)
# add data for plotting the constraints. I.e. how much barley we can have in each constraint given an amount of wheat.
plotDF = data.frame(x1, constraint1(x1), constraint2(x1))
names(plotDF) = c('x1','con1','con2')
plotDF$zero = rep(0,length(x1))
# add data for plotting the profit contours. I.e. how much barlet do we need to make a certain profit given a certain amount of wheat.
for (z in c(25, 40, 55, 70, 85)){
  x2 <- data.frame(profitContour(x1, z))
  names(x2) = paste('z', z, sep="")
  plotDF <- cbind(plotDF, x2)
}
#set all negatives to zero, since you can't have negative x2
plotDF <- replace(plotDF, plotDF<0, 0)


### set up and view the charts
# plot the constraint lines
p0 = ggplot(plotDF, aes(x = x1)) + 
  coord_cartesian(ylim=c(0,25),xlim = c(0,25))+                      
  geom_line(aes(y = con1), colour = 'red', linetype = 2) +
  geom_line(aes(y = con2), colour = 'green', linetype = 2) +
  xlab('x1') +
  ylab('x2') 



# add an area plot underneath the constraint lines. This is the feasible solution space.
p1 <- p0 +  geom_area(aes(y = pmin(con1,con2)), fill = 'gray40')
# view the constraints and feasible solution space


# add the profit contour lines
p2 <- p1 +                    
  geom_line(aes(y = z25), colour = 'blue', linetype = 1) +
  geom_line(aes(y = z40), colour = 'blue', linetype = 1) +
  geom_line(aes(y = z55), colour = 'blue', linetype = 1) +
  geom_line(aes(y = z70), colour = 'blue', linetype = 1) +
  geom_line(aes(y = z85), colour = 'blue', linetype = 1)
# view the whole chart
plotDF
```

```
##    x1 con1      con2 zero       z25        z40       z55      z70      z85
## 1   0    5 11.666667    0 8.3333333 13.3333333 18.333333 23.33333 28.33333
## 2   1    6 11.333333    0 7.6666667 12.6666667 17.666667 22.66667 27.66667
## 3   2    7 11.000000    0 7.0000000 12.0000000 17.000000 22.00000 27.00000
## 4   3    8 10.666667    0 6.3333333 11.3333333 16.333333 21.33333 26.33333
## 5   4    9 10.333333    0 5.6666667 10.6666667 15.666667 20.66667 25.66667
## 6   5   10 10.000000    0 5.0000000 10.0000000 15.000000 20.00000 25.00000
## 7   6   11  9.666667    0 4.3333333  9.3333333 14.333333 19.33333 24.33333
## 8   7   12  9.333333    0 3.6666667  8.6666667 13.666667 18.66667 23.66667
## 9   8   13  9.000000    0 3.0000000  8.0000000 13.000000 18.00000 23.00000
## 10  9   14  8.666667    0 2.3333333  7.3333333 12.333333 17.33333 22.33333
## 11 10   15  8.333333    0 1.6666667  6.6666667 11.666667 16.66667 21.66667
## 12 11   16  8.000000    0 1.0000000  6.0000000 11.000000 16.00000 21.00000
## 13 12   17  7.666667    0 0.3333333  5.3333333 10.333333 15.33333 20.33333
## 14 13   18  7.333333    0 0.0000000  4.6666667  9.666667 14.66667 19.66667
## 15 14   19  7.000000    0 0.0000000  4.0000000  9.000000 14.00000 19.00000
## 16 15   20  6.666667    0 0.0000000  3.3333333  8.333333 13.33333 18.33333
## 17 16   21  6.333333    0 0.0000000  2.6666667  7.666667 12.66667 17.66667
## 18 17   22  6.000000    0 0.0000000  2.0000000  7.000000 12.00000 17.00000
## 19 18   23  5.666667    0 0.0000000  1.3333333  6.333333 11.33333 16.33333
## 20 19   24  5.333333    0 0.0000000  0.6666667  5.666667 10.66667 15.66667
## 21 20   25  5.000000    0 0.0000000  0.0000000  5.000000 10.00000 15.00000
```

```r
p2
```

<img src="/blog/ORIE/LP Algorithms/LP_algorithms_files/figure-html/unnamed-chunk-3-1.png" width="672" />

```r
rm(x1,x2,p0,p1,p2, constraint1, constraint2, plotDF, profitContour,z )
```


## Simplex Algorithm

primal simplex tableu reformulation
`\(max: z;\: z - 2x_1 - 3x_2 = 0\)`
s.t. 
`\(-x_1 + x_2 + x_3 = 5\)`
`\(x_1+ 3x_2 + x_4 = 35\)`
`\(x_1 + x_5 = 20\)`

where `\(x_3, x_4, \text{ and } x_5\)` are slack variables. Giving 3 basic variables for 3 equations. The "4th" constraint describes how z changes with the decision variables
For less than or equal constraints adding the slack variables define a basic feasible solution which we use to initialize the algorithm (note use `\(x+1\)` as basic variable instead of `\(x_5\)` for iteration reasons)



```r
tic('Simplex')
initial_tableau <- data.frame(row = c(0,1,2,3), basic = (c('z', 'x3', 'x4', 'x5')), z = c(1,0,0,0), x1 = c(-2,-1,1,1), x2 = c(-3,1,3,0), x3 = c(0,1,0,0), x4 = c(0,0,1,0), x5 = c(0,0,0,1),   RHS = c(0,5,35,20), ratio = c(0,0,0,0))

initial_tableau$basic <- as.character(initial_tableau$basic)

nvars <- 5
nrows <- 3



tableau <- initial_tableau

iters <- 1


#loop iterate until you have no negative coefficients in the first row of the tableau

maxiters <- 10
while(iters < maxiters){
  
  
  

#create a and RHS matrixes for easy calculations

ma <- as.matrix(tableau[,4:(4+nvars-1)])


#Run this only if there is a negative reduced cost
if(min(ma[1,]) < 0){
mrhs <- as.matrix(tableau[,(4+nvars)])
print(paste('iteration:',iters))
print(tableau)


#use steepest ascent to find the most negative reduced cost and that is the variable that enters the basis (sa) as seen in row 0, caluclate the rations, then determine the pivot row index (pri)
sa <- which.min(ma[1,])


ratios <- mrhs[2:(nrows+1)]/ma[2:(nrows+1),sa]
ratios[ratios<=0] <- 9999
pri <- which.min(ratios)+1





#change pivot row by pivot element (pe) using Gauss Jordan elimination (substition)
#by simply divide the row and rhs by the pe to get a new pivot row (npr) and new rhs (nrhs)
#https://www.coursera.org/lecture/solving-algorithms-discrete-optimization/3-3-1-linear-programming-rzHVE

pe <- ma[pri,sa]
npr <- ma[pri,]/pe

nrhs <- mrhs[pri]/pe

#take that row and muliply by the negative of the pivot variable's coefficent in that row column and add the result to that row for both the rhs matrix and the A matrix

for(i in 1:(nrows+1)){
  mrhs[i] <- -ma[i,sa]*nrhs+mrhs[i]
  }
mrhs[pri] <- nrhs


for(i in 1:(nrows+1)){
  ma[i,] <- -ma[i,sa]*npr+ma[i,]
  }
ma[pri,] <- npr

#rewrite the new A and RHS matricies to the tableau 

tableau[,4:(4+nvars-1)] <- ma
tableau[,(4+nvars)] <- mrhs
tableau[2:(nrows+1),(4+nvars+1)] <- ratios
print(paste('pivot row:',(pri-1)))
print(paste('new basis:', sa))
tableau[pri,2] <- paste0('x',sa)

iters <- iters + 1
}
else{
  print(paste('Final Tableau; iteration:',iters))
  print(tableau[1:(length(tableau)-1)])
  print(paste0('objective value:', mrhs[1]))
  for(j in 1:nrows+1){
    print(paste(tableau[j,2], '=', tableau[j,(4+nvars)]))}
  iters <- maxiters}
}
```

```
## [1] "iteration: 1"
##   row basic z x1 x2 x3 x4 x5 RHS ratio
## 1   0     z 1 -2 -3  0  0  0   0     0
## 2   1    x3 0 -1  1  1  0  0   5     0
## 3   2    x4 0  1  3  0  1  0  35     0
## 4   3    x5 0  1  0  0  0  1  20     0
## [1] "pivot row: 1"
## [1] "new basis: 2"
## [1] "iteration: 2"
##   row basic z x1 x2 x3 x4 x5 RHS    ratio
## 1   0     z 1 -5  0  3  0  0  15  0.00000
## 2   1    x2 0 -1  1  1  0  0   5  5.00000
## 3   2    x4 0  4  0 -3  1  0  20 11.66667
## 4   3    x5 0  1  0  0  0  1  20      Inf
## [1] "pivot row: 2"
## [1] "new basis: 1"
## [1] "iteration: 3"
##   row basic z x1 x2    x3    x4 x5 RHS ratio
## 1   0     z 1  0  0 -0.75  1.25  0  40     0
## 2   1    x2 0  0  1  0.25  0.25  0  10  9999
## 3   2    x1 0  1  0 -0.75  0.25  0   5     5
## 4   3    x5 0  0  0  0.75 -0.25  1  15    20
## [1] "pivot row: 3"
## [1] "new basis: 3"
## [1] "Final Tableau; iteration: 4"
##   row basic z x1 x2 x3         x4         x5 RHS
## 1   0     z 1  0  0  0  1.0000000  1.0000000  55
## 2   1    x2 0  0  1  0  0.3333333 -0.3333333   5
## 3   2    x1 0  1  0  0  0.0000000  1.0000000  20
## 4   3    x3 0  0  0  1 -0.3333333  1.3333333  20
## [1] "objective value:55"
## [1] "x2 = 5"
## [1] "x1 = 20"
## [1] "x3 = 20"
```

```r
toc()
```

```
## Simplex: 0.05 sec elapsed
```

```r
rm(pri,sa,npr,iters,maxiters,ma,mrhs,nrhs,nrows,nvars,pe,ratios,i,j)
rm(initial_tableau, tableau)
```

## Dual Simplex Algorithm


The dual of the previous problem is 
`\(min 5y_1+35y_2+20y_3\)` 
s.t. 
`\(-y_1+y_2+y_3 \geq 2\)`
`\(y_1+3y_2 \geq 3\)`

Switching to a max problem and adding slacks yields 
`\(z=-5y_1-35y_2-20y_3\)`
s.t. 
`\(y_1-y_2-y_3+y_4 = -2\)`
`\(-y_1-3y_2+y_5 = -3\)`


```r
tic('Dual Simplex')
initial_tableau <- data.frame(row = c(0,1,2), basic = (c('z', 'y4', 'y5')), z = c(1,0,0), y1 = c(5,1,-1), y2 = c(35,-1,-3), y3 = c(20,-1,0), y4 = c(0,1,0), y5 = c(0,0,1),   RHS = c(0,-2,-3))

initial_tableau$basic <- as.character(initial_tableau$basic)

nvars <- 5
nrows <- 2



tableau <- initial_tableau

iters <- 1

maxiters <- 10

while(iters < maxiters){
  

#create a and RHS matrixes for easy calculations

ma <- as.matrix(tableau[,4:(4+nvars-1)])

#Check to see if a RHS value is negative
if(min(tableau[2:(nrows+1),(4+nvars)]) < 0){
mrhs <- as.matrix(tableau[,(4+nvars)])
print(paste('iteration:',iters))
print(tableau)


#use steepest ascent to find the most negative RHS and that is the pivot row index (pri)
#then caluclate the ratios to determine the entering variable (ev)

pri <- which.min(mrhs[2:(nrows+1),])+1

ratios <- -ma[1,]/ma[pri,]
ratios[ratios<=0] <- 9999
ev <- which.min(ratios)


#identify the new pivot element, do the same matrix operations to make the new pivot row and the new rhs for that row
#change pivot row by pivot element (pe) using Gauss Jordan elimination (substition)
#by simply divide the row and rhs by the pe to get a new pivot row (npr) and new rhs (nrhs)
#https://www.coursera.org/lecture/solving-algorithms-discrete-optimization/3-3-1-linear-programming-rzHVE

pe <- ma[pri,ev]
npr <- ma[pri,]/pe

nrhs <- mrhs[pri]/pe

#Do the matrix operations for the rest of the tableau


#take that row and muliply by the negative of the pivot variable's coefficent in that row column and add the result to that row for both the rhs matrix and the A matrix

for(i in 1:(nrows+1)){
  mrhs[i] <- -ma[i,ev]*nrhs+mrhs[i]
  }
mrhs[pri] <- nrhs


for(i in 1:(nrows+1)){
  ma[i,] <- -ma[i,ev]*npr+ma[i,]
  }
ma[pri,] <- npr

#rewrite the new A and RHS matricies to the tableau 

tableau[,4:(3+nvars)] <- ma
tableau[,(4+nvars)] <- mrhs
print(paste('pivot row:',(pri-1)))
print(paste('entering variable:',ev))
tableau[pri,2] <- paste0('y',ev)




iters <- iters + 1
}
else{
  print(tableau)
  print(paste('objective value:', mrhs[1]))
  for(j in 1:nrows+1){
    print(paste(tableau[j,2], '=', tableau[j,(4+nvars)]))}
  iters <- maxiters}

}
```

```
## [1] "iteration: 1"
##   row basic z y1 y2 y3 y4 y5 RHS
## 1   0     z 1  5 35 20  0  0   0
## 2   1    y4 0  1 -1 -1  1  0  -2
## 3   2    y5 0 -1 -3  0  0  1  -3
## [1] "pivot row: 2"
## [1] "entering variable: 1"
## [1] "iteration: 2"
##   row basic z y1 y2 y3 y4 y5 RHS
## 1   0     z 1  0 20 20  0  5 -15
## 2   1    y4 0  0 -4 -1  1  1  -5
## 3   2    y1 0  1  3  0  0 -1   3
## [1] "pivot row: 1"
## [1] "entering variable: 2"
## [1] "iteration: 3"
##   row basic z y1 y2    y3    y4    y5    RHS
## 1   0     z 1  0  0 15.00  5.00 10.00 -40.00
## 2   1    y2 0  0  1  0.25 -0.25 -0.25   1.25
## 3   2    y1 0  1  0 -0.75  0.75 -0.25  -0.75
## [1] "pivot row: 2"
## [1] "entering variable: 3"
##   row basic z         y1 y2 y3 y4         y5 RHS
## 1   0     z 1 20.0000000  0  0 20  5.0000000 -55
## 2   1    y2 0  0.3333333  1  0  0 -0.3333333   1
## 3   2    y3 0 -1.3333333  0  1 -1  0.3333333   1
## [1] "objective value: -55"
## [1] "y2 = 1"
## [1] "y3 = 1"
```

```r
toc()
```

```
## Dual Simplex: 0.03 sec elapsed
```

```r
rm(pri,npr,iters,maxiters,ma,mrhs,nrhs,nrows,nvars,pe,ev,ratios,i,j)
rm(initial_tableau, tableau)
```

## Interior Point Algorithm

Source: http://fourier.eng.hmc.edu/e176/lectures/ch3/node19.html

KKT (via interior points) vs Simplex
https://math.stackexchange.com/questions/3422607/why-would-you-choose-simplex-over-lagrange-kkt-multipliers-methods

Standard form:

`\(max: z; z - 2x_1 - 3x_2 = 0\)`
s.t. 
`\(-x_1 + x_2 + x_3 = 5\)`
`\(x_1+ 3x_2 + x_4 = 35\)`
`\(x_1 + x_5 = 20\)`
Idea given A,b,c and intial value of x; find optimal x that minimizes c'*x



```r
tic('Interior Point: Newton Raphson')
constr1 <- c(-1,1,1,0,0)
constr2 <- c(1,3,0,1,0)
constr3 <- c(1,0,0,0,1)

A <- rbind(constr1,constr2, constr3)

b <- matrix(c(5,35,20),nrow =3)
c <- matrix(c(-2,-3,0,0,0), nrow = 5)
#inital x values (xi) just has to be a feasible solution, but give every x variable a value or there will be numerical instablity problems in the matricies
xi <- matrix(c(1,1,5,31,19), nrow =5)

m <- nrow(A)
n <- ncol(A)

I <- diag(n)
z1 <- matrix(rep(0,n*n), nrow = n)
z2 <- matrix(rep(0,m*m), nrow = m)
z3 <- matrix(rep(0,m*n), nrow = m)
y <- matrix(rep(1,5), nrow = 5)

#The complimentary slackness modifier 1/t eventually goes to 0 as t >>>> inf
t <- 9
#Step size pretty much make it up the higher the more the step changes, but it might be too quick.
#if its too quick it converges on negative values of x which is bad, 
#for an example change this to 0.3 to see a slower convergance and then to 1 to see a divergence
alpha <- .5
#mu*x = 0 in complemntariy slackness condition , mu >0 is dual condition mu correspond to dual variables, 
#using fancy vectors this gives Xd*mu = XM1 = 1/t where t >>>> inf 
x <- xi
mu <- x/t
mu_minus_c <- mu - c
#Gives lagrangian multipliers for constraints
#Solving c+A*lamda-mu = 0 gives initial lambda
lambda <- ginv(t(A))%*%(mu_minus_c)


#combined vector having values of x, lambda, and mu useful when adding the search direction
w <- rbind(x, lambda, mu)


#This is the KKT condition stationarity, at optimality this derivative should  be 0,
#Using the lagrangian cx+lambda*Ax-mu >> c+A*lambda-mu
c_plus_tA <- c+t(A)%*%lambda-mu

#This is the KKT condition primal feasiblity, this should always be 0 Ax-b=0 
A_times_x_minus_b <- A%*%x-b

#This is the modfied complimentary condtion XM1 -1/t = 0 X is the diag(x) and M is diag(mu) 1/t >>> 0 as t gets larger
x_times_mu_minus_y_over_t <- x*mu-y/t

#The right hand side of the search direction iteration given from the Newton-Raphson Method
#Combines the vectors above
B <- rbind(c_plus_tA,A_times_x_minus_b,x_times_mu_minus_y_over_t)

objective <- t(c)%*%x
error <- norm(B,'2')

iteration_list <- data.frame('x1' = x[1], 'x2' = x[2], 'x3' = x[3], 'x4' = x[4], 'x5' = x[5], 'objective' = objective, 'error' = error)

#loop



while(error > 10^-7){
t <- t*9

Xd = Diagonal(n = n, x)

Mud = Diagonal(n = n, mu) 


#The left hand side matrix of the search direction iteration, it containtes information from the A, x, and mu vectors and matricies of 1s or 0s to make the math make sense

C <- rbind(cbind(z1,t(A),-I),cbind(A,z2,z3), cbind(Mud,t(z3), Xd))

#The right hand side of the search direction iteration given from the Newton-Raphson Method
#This contains the objective function costs, the RHS values, as well as the A, x, and mu vectors. 
#It also has the complimentary condition represented by t
B <- rbind(c+t(A)%*%lambda-mu,A%*%x-b,x*mu-y/t)


#solving the systems of equations with C and B gives the search direction as you move closer and closer to solving the complimentary condition in the KKT conditions
dw = solve(-C,B)


#update your w vector which is just a list of the x, mu, and lambda vectors using the search direction
w <- w + alpha*dw

x <- w[1:n]

lambda <- w[(n+1):(n+m)]

mu <- w[(n+m+1):length(w)]

#calculate the objective function from the x values and the error. Remember if this satisifies all the KKT conditions then the B vector will be 0.
objective <- t(c)%*%x
error <- norm(B,'2')
iteration_list <- rbind(iteration_list,c(x,objective,error))

}

toc()
```

```
## Interior Point: Newton Raphson: 0.17 sec elapsed
```

```r
head(iteration_list)
```

```
##          x1       x2        x3        x4        x5 objective     error
## 1  1.000000 1.000000  5.000000 31.000000 19.000000  -5.00000 113.97801
## 2  2.815083 1.900809  5.914274 26.482489 17.184917 -11.33259 114.10755
## 3 10.549002 1.617924 13.931079 19.597227  9.450998 -25.95178  62.76629
## 4 13.970331 2.387418 16.582913 13.867413  6.029669 -35.10292  35.22695
## 5 16.877593 3.136894 18.740699  8.711726  3.122407 -43.16587  19.61963
## 6 18.419517 3.865645 19.553873  4.983549  1.580483 -48.43597  10.66207
```

```r
tail(iteration_list)
```

```
##    x1 x2 x3           x4           x5 objective        error
## 28 20  5 20 1.386952e-06 3.808029e-07       -55 2.878188e-06
## 29 20  5 20 6.934761e-07 1.904014e-07       -55 1.439094e-06
## 30 20  5 20 3.467381e-07 9.520072e-08       -55 7.195471e-07
## 31 20  5 20 1.733690e-07 4.760036e-08       -55 3.597735e-07
## 32 20  5 20 8.668451e-08 2.380018e-08       -55 1.798868e-07
## 33 20  5 20 4.334226e-08 1.190009e-08       -55 8.994339e-08
```

```r
rm(x,lambda,mu,z1,z2,z3,y,xi,Xd,Mud,t,n,I,alpha,b,c,constr1,constr2,constr3,m)
rm(c_plus_tA,mu_minus_c,A_times_x_minus_b,x_times_mu_minus_y_over_t, A,B,C,dw)
rm(iteration_list,objective,error,w)
```


## Run All Methods with Gurobi

`\(max: z = 2x_1 + 3x_2\)`

s.t. 

`\(-x_1 + x_2 \leq 5\)`
`\(x_1+3x_2 \leq 35\)`
`\(x_1 \leq 20\)`

This solver runs all the techniques above in paralel.
The Simplex, Dual Simplex, and 3 versions of the interior point method (barrier method).
This requires 5 cores. Whichever one solves the fastest produces the output.


```r
tic('Gurobi Solver')
model <- list()
model$A     <- matrix(c(-1,1,
                        1,3,
                        1,0), nrow=3, byrow=T)
model$obj   <- c(2,3)
model$rhs   <- c(5,
                 35,
                 20)
model$sense <- c('<',
                 '<',
                 '<')
model$modelsense <- 'max'
result <- gurobi(model)
```

```
## Gurobi Optimizer version 9.1.2 build v9.1.2rc0 (win64)
## Thread count: 10 physical cores, 20 logical processors, using up to 20 threads
## Optimize a model with 3 rows, 2 columns and 5 nonzeros
## Model fingerprint: 0x7d34ce81
## Coefficient statistics:
##   Matrix range     [1e+00, 3e+00]
##   Objective range  [2e+00, 3e+00]
##   Bounds range     [0e+00, 0e+00]
##   RHS range        [5e+00, 4e+01]
## Presolve removed 1 rows and 0 columns
## Presolve time: 0.00s
## Presolved: 2 rows, 2 columns, 4 nonzeros
## 
## Iteration    Objective       Primal Inf.    Dual Inf.      Time
##        0    7.0000000e+01   1.875000e+00   0.000000e+00      0s
##        1    5.5000000e+01   0.000000e+00   0.000000e+00      0s
## 
## Solved in 1 iterations and 0.00 seconds
## Optimal objective  5.500000000e+01
```

```r
#print(result$objval)
#print(result$x)



# Clear space
rm(model, result)

toc()
```

```
## Gurobi Solver: 0.02 sec elapsed
```



