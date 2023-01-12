---
title: "Quadratic Programming Examples and Algorithms"
author: "Erick Jones"
date: 2020-01-07
categories: ["ORIE Techniques"]
tags: ["R Markdown", "QP", "Interior Point", "Algorithms"]
---


## Background

This post explores how to solve QPs by hand and with Gurobi. 
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
#library(matlib)
library(MASS)
```


## Gradient Descent: A first order method

`\(x_{i+1} = x_i + \lambda*d/f(x)\)`


```r
x <- c(-10:10)
fx1 <- x^2
xi <- 10
lambda <- 0.2
xit <- xi

fx <- xit^2
dfx <- 2*xit

gd <- data.frame('x' = xit, 'fx' = fx, 'dfx' = dfx)



while(dfx < -10^-50 | dfx > 10^-50){
xit <- xit - lambda*dfx
fx <- xit^2
dfx <- 2*xit
gd <- rbind(gd, c(xit,fx,dfx))

}

head(gd)
```

```
##         x          fx     dfx
## 1 10.0000 100.0000000 20.0000
## 2  6.0000  36.0000000 12.0000
## 3  3.6000  12.9600000  7.2000
## 4  2.1600   4.6656000  4.3200
## 5  1.2960   1.6796160  2.5920
## 6  0.7776   0.6046618  1.5552
```

```r
tail(gd)
```

```
##                x            fx          dfx
## 228 4.368515e-50  1.908392e-99 8.737030e-50
## 229 2.621109e-50 6.870213e-100 5.242218e-50
## 230 1.572665e-50 2.473277e-100 3.145331e-50
## 231 9.435993e-51 8.903795e-101 1.887199e-50
## 232 5.661596e-51 3.205366e-101 1.132319e-50
## 233 3.396957e-51 1.153932e-101 6.793915e-51
```

```r
qplot(x = x, y = fx1)
```

<img src="/blog/ORIE/QP Modeling/QP_algorithms_files/figure-html/gradient_descent_example-1.png" width="672" />

```r
qplot(x,fx, data = gd)
```

<img src="/blog/ORIE/QP Modeling/QP_algorithms_files/figure-html/gradient_descent_example-2.png" width="672" />




## Standard form for quadratics

`\(min\: z; z = -8x_1-16x_2+x_1^2+4x_2^2\)`

s.t. 
`\(x_1 + x_2 + x_3 = 5\)` 
`\(x_1 + x_4 = 3\)`
From Jensen and Bard Quadratic Solver notes in Book
Idea given A,b,c and intial value of x; find optimal x that minimizes c'*x


```r
constr1 <- c(1,1,1,0)
constr2 <- c(1,0,0,1)


A <- rbind(constr1,constr2)

m <- nrow(A)
n <- ncol(A)


b <- matrix(c(5,3),nrow =m)
c <- matrix(c(-8,-16,0,0), nrow = n)
Q <- rbind(c(2,0,0,0),c(0,8,0,0),c(0,0,0,0),c(0,0,0,0))
#inital x values (xi) given by slacks = RHS
xi <- matrix(c(1,1,0.5,0.5), nrow =n)

m <- nrow(A)
n <- ncol(A)

I <- diag(n)
z1 <- matrix(rep(0,n*n), nrow = n)
z2 <- matrix(rep(0,m*m), nrow = m)
z3 <- matrix(rep(0,m*n), nrow = m)
y <- matrix(rep(1,m), nrow = n)
#The complimentary slackness modifier 1/t eventually goes to 0 as t >>>> inf
t <- 9
#Step size pretty much make it up
alpha <- 0.1
#mu*x = 0 in complemntariy slackness condition , mu >0 is dual condition mu correspond to dual variables, 
#using fancy vectors this gives Xd*mu = XM1 = 1/t where t >>>> inf 
x <- xi
mu <- x/t
mu_minus_c <- mu - c - Q%*%x
#Gives lagrangian multipliers for constraints
#Solving c+A*lamda-mu = 0 gives initial lambda
lambda <- ginv(t(A))%*%(mu_minus_c)


#combined vector having values of x, lambda, and mu useful when adding the search direction
w <- rbind(x, lambda, mu)


#This is the KKT condition stationarity, at optimality this derivative should  be 0,
#Using the lagrangian cx+lambda*Ax-mu >> c+A*lambda-mu
c_plus_tA <- Q%*%x+c+t(A)%*%lambda-mu

#This is the KKT condition primal feasiblity, this should always be 0 Ax-b=0 
A_times_x_minus_b <- A%*%x-b

#This is the modfied complimentary condtion XM1 -1/t = 0 X is the diag(x) and M is diag(mu) 1/t >>> 0 as t gets larger
x_times_mu_minus_y_over_t <- x*mu-y/t

#The right hand side of the search direction iteration given from the Newton-Raphson Method
#Combines the vectors above
B <- rbind(c_plus_tA,A_times_x_minus_b,x_times_mu_minus_y_over_t)

objective <- 0.5*t(x)%*%Q%*%x+t(c)%*%x
error <- norm(B,'2')

iteration_list <- data.frame('x1' = x[1], 'x2' = x[2], 'x3' = x[3], 'x4' = x[4], 'objective' = objective, 'error' = error)

#loop



while(error > 10^-12){
t <- t*9

Xd = Diagonal(n = n, x)

Mud = Diagonal(n = n, mu) 


#The left hand side matrix of the search direction iteration, it containtes information from the A, x, and mu vectors and matricies of 1s or 0s to make the math make sense

C <- rbind(cbind(Q,t(A),-I),cbind(A,z2,z3), cbind(Mud,t(z3), Xd))

#The right hand side of the search direction iteration given from the Newton-Raphson Method
#This contains the objective function costs, the RHS values, as well as the A, x, and mu vectors. 
#It also has the complimentary condition represented by t
B <- rbind(Q%*%x+c+t(A)%*%lambda-mu,A%*%x-b,x*mu-y/t)


#solving the systems of equations with C and B gives the search direction as you move closer and closer to solving the complimentary condition in the KKT conditions
dw = solve(-C,B)


#update your w vector which is just a list of the x, mu, and lambda vectors using the search direction
w <- w + alpha*dw

x <- w[1:n]

lambda <- w[(n+1):(n+m)]

mu <- w[(n+m+1):length(w)]

#calculate the objective function from the x values and the error. Remember if this satisifies all the KKT conditions then the B vector will be 0.
objective <- 0.5*t(x)%*%Q%*%x+t(c)%*%x
error <- norm(B,'2')
iteration_list <- rbind(iteration_list,c(x,objective,error))

}

head(iteration_list)
```

```
##         x1       x2         x3         x4 objective    error
## 1 1.000000 1.000000 0.50000000 0.50000000 -19.00000 6.520410
## 2 1.269990 1.096877 0.38313300 0.38001002 -21.28452 6.520877
## 3 1.516786 1.183938 0.27427586 0.26821381 -23.16982 5.868998
## 4 1.734188 1.261642 0.18166935 0.17231180 -24.68541 5.282035
## 5 1.918400 1.330365 0.11098503 0.09745009 -25.87330 4.753769
## 6 2.066579 1.390789 0.06640714 0.04768647 -26.77733 4.278347
```

```r
tail(iteration_list)
```

```
##     x1 x2           x3           x4 objective        error
## 278  3  2 3.020886e-08 5.122922e-16       -31 1.531109e-12
## 279  3  2 2.869842e-08 4.610630e-16       -31 1.377965e-12
## 280  3  2 2.726350e-08 4.149567e-16       -31 1.239794e-12
## 281  3  2 2.590032e-08 3.734610e-16       -31 1.115852e-12
## 282  3  2 2.460531e-08 3.361149e-16       -31 1.003790e-12
## 283  3  2 2.337504e-08 3.025034e-16       -31 9.038590e-13
```






## standard form for quadratics 2

`\(min \:z; z = 4x_1^2 + 4x_2^2 - 2x_1x_2 - 12x_1 - 72x_2 + 384\)`
s.t. 
`\(2x_1 + x_2 + x_3 = 18\)`
`\(6x_1+ 5x_2 + x_4 = 60\)`
`\(2x_1 + 5x_2 + x_5 = 40\)`



```r
#Another example from http://fourier.eng.hmc.edu/e176/lectures/ch3/node19.html, but don't have answers 



#Work out matricies in Wolfram Alpha

#Idea given A,b,c and intial value of x; find optimal x that minimizes c'*x

constr1 <- c(2,1,1,0,0)
constr2 <- c(6,5,0,1,0)
constr3 <- c(2,5,0,0,1)

A <- rbind(constr1,constr2, constr3)

b <- matrix(c(18,60,40),nrow =3)
c <- matrix(c(-8,-16,0,0,0), nrow = 5)
Q <- rbind(c(2,1,0,0,0),c(1,2,0,0,0),c(0,0,0,0,0),c(0,0,0,0,0),c(0,0,0,0,0))

rm(A,b,c,Q,constr1,constr2,constr3)
```


