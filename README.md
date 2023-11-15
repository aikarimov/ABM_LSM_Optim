# ABM_LSM_Optim
This repository contains codes for reconstructing dynamical systems using the least square method. 

## Overview

Suppose, the problem is to find a description of a continuous dynamical system in a form of an autonomous odrinary differential equation:
$$\dot{\mathbf{x}} = \mathbf{f}(\mathbf{x}).$$

Let us have a number of sample points of the trajectiory $\mathbf{x}(t_i)$, but the mathematical description of the function $\mathbf{f}(\mathbf{x})$ is unknown. 

Suppose, every line of a function $\mathbf{f}(\mathbf{x})$ is a sum of monomials with coefficients, such as

$$f_j(\mathbf{x}) = 2x + 3y^2 + 17yz \dots, $$

where $\mathbf{x} = (x,y,z, \dots) ^\top$ is a phase vector. Any combination of its entries like $x,y^2,yz$ is called a monomial, and number by them like $2,3,17$ are coefficients. Then, we can use ABM_LSM_Optim to find $\mathbf{f}(\mathbf{x})$ in such a form.

For example, we have a recorded three-dimensional trajectory $\mathbf{x} = (x,y,z)^\top$ as in the left pane below, shown blue. 

![Fig1](https://github.com/aikarimov/ABM_LSM_Optim/blob/main/scheme.drawio.png)

We randomly select some sample points, shown green-yellow in the middle pane, and reconstruct sparse, readable equations of the system, obtaining:

$$\begin{cases}
\begin{aligned}
& \dot{x} = -10x + 10y \\
& \dot{y} = 28x - y - xz\\
& \dot{z} = -2.6667z - xy\\
\end{aligned}
\end{cases}$$

Then, we can solve this reconstructed system with a standard matlab solver like `ode45`. The obtained trajectory is shown yellow in the right pane.

## Installation
Download a zip file or via git, and then add the ABM_LSM_Optim directory to your search path:

```matlab
>> addpath('C:\Users\...\Downloads\ABM_LSM_Optim')  
>> savepath
```
## How to use

Let us find equations of the Lorenz system. First, generate a full trajectory with a stepsize $h=0.01$ from the initial point $(0.1,0,-0.1)^\top$:
```matlab
%simulate Lorenz system
Tmax = 45;
h = 0.01;
[t,y] = ode45(@Lorenz,[0:h:Tmax],[0.1,0,-0.1]); %solve ODE
w = transpose(Lorenz(0,transpose(y))); %find derivatives
```
Then, select $N$ random points:

```matlab
%get uniformly distributed points from the simulated attractor
N = 19; %data points
M = 3; %dimension
[Ns, ~] = size(y); %get the number of data point
W = zeros(N,M); %sample derivatives
Y = zeros(N,M); %sample phase coordinates

for i = 1:N %take random points from trajectory
    id = ceil(rand*Ns);  
    W(i,:) = w(id,:); 
    Y(i,:) = y(id,:);
end
```
Then, we obtain the Lorenz equations from these 19 points using ABM_LSM_Optim. First, we use a function `PolyRegression` to obtain two cell arrays $T$ and $H$, containing all necessary information about the reconstructed system (see the section [Algorithm](https://github.com/aikarimov/ABM_LSM_Optim/tree/main#algorithm) for details):

```matlab
dmax = 2; % maximum power of the monomial
[H,T] = PolyRegression(Y,W,dmax);
```
To show the reconstruction result, we use a function `prettyABM`:

```matlab
prettyABM(H,T)
```
Which outputs into the console:
```
f1 = -10*x1 + 10*x2
f2 = 28*x1 - x2 - x1*x3
f3 = -2.6667*x3 + x1*x2
```
Then, we simulate the results using a function `oderecon`:
```matlab
[~,y] = ode45(@(t,x)oderecon(H,T,t,x),[0:h:Tmax],[0.1,0,-0.1]); %solve ODE
```
## Algorithm

First, introduce some formalism. Representation of an arbitrary $M$-dimensional function $\mathbf{f}(\mathbf{x})$ is contained in two cell arrays $H$ and $T$. Entries of $H$ are $L_i \times 1$ matrices (column vectors) of coefficients by monomials in $i$-th line of $\mathbf{f}(\mathbf{x})$, where $L_i$ is a number of terms in $i$-th line. Entries of $T$ are $L_i \times M$ matrices containing powers of variables in each monomial, ordered degree-lexicographically.

On the first stage of the algorithm, for each line, full matrices $H_i$ and $T_i$ are created, containing all possible variants of powers up to $d_{max}$. Example of full degree-lexicographic ordering $\sigma$ is shown in the left of the figure, and example of the Lorenz system represented in such a way is given in the right of the figure. 

![Fig2](https://github.com/aikarimov/ABM_LSM_Optim/blob/main/handt.drawio.png)

Ordering $\sigma$ is generated with the function `deglexord(dmin,dmax,M)`. Sparse reconstruction of the equations needs eliminating all excessive terms in $T_i$ and $H_i$ and setting correct values to entries of $H_i$. 

Suppose, we have a trajectory $Y$ which is represented as $N \times M$ matrix, with $N$ sample points and $M$ dimensions, and a derivative to the trajectory $W = \dot{Y}$ which is also represented as $N \times M$ matrix. 

First, the approximate Buchberger-Moller (ABM) algorithm is run, which excludes all monomials in $T_i$ that vanish on a given set $Y$:

```matlab
[~, O] = ApproxBM(Y, eps, sigma); %use approximate Buchberger-Moller algorithm
```
This returns a border basis `O` which is then used as an initial guess for each $T_i$. 


## Literature
The ABM and delMinorTerms routines are written following pseudocodes provided in the work

Kera, H.; Hasegawa, Y. Noise-tolerant algebraic method for reconstruction of nonlinear dynamical systems. Nonlinear Dynamics 2016, 85(1), 675-692,  https://doi.org/10.1007/s11071-016-2715-3

If you use this code or its parts in scientific work, please, cite the following papers:

1. Karimov, A.; Nepomuceno, E.G.; Tutueva, A.; Butusov, D. Algebraic Method for the Reconstruction of Partially Observed Nonlinear Systems Using Differential and Integral Embedding. Mathematics 2020, 8, 300. https://doi.org/10.3390/math8020300

2. Karimov, A.; Rybin, V.; Kopets, E.; Karimov, T.; Nepomuceno, E.; Butusov, D. Identifying empirical equations of chaotic circuit from data. Nonlinear Dyn. 2023, 111:871–886 https://doi.org/10.1007/s11071-022-07854-0

## License
MIT License
