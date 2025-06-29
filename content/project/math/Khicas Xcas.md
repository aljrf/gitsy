---
tags:
  - math
---
# Khicas Xcas

Factor:

x^4-1=>*

1/(x^4-1)=>+

![[Khicas Xcas_pic_20230411162413030.png]]

![[Khicas Xcas_pic_20230411162440126.png]]
## Differentiation

diff(sin(x),x)

sin(x)’
## Integration:

integrate(sin(x))

integrate(1/(t^4-1),t)

## Defined integration:

integrate(sin(x)^4,x,0,pi)

## limit

limit((cos(x)-1)/x^2,x=0)

## plot
plot(x^3-7x+5,x,-4,4)

## table of variations of an expression

tabvar(x^3-7x+5)

## taylor

taylor(sin(x),x=0,5)

## discrete summation from 1 to n

sum(k^2,k,1,n)

## solve

solve solves an equation exactly. Takes the variable to solve for as second argument,
unless it is x, for example solve(t^2-1=0,t)

If exact solving fails, run fsolve for approx solving, either with an iterative
method starting with a guess fsolve(cos(x)=x,x=0.0)

For complex solutions, run csolve.
It is possible to restrict solutions using assumptions on the variable, for example
assume(m>1)
m
then solve(m^2-4=0,m)

solve can also solve (simple) polynomial systems, enter a list of equations as
1st argument and a list of variables as 2nd argument, for example intersection of
a circle and a line:
solve([x^2+y^2+2y=3,x+y=1],[x,y])

Run linsolve to solve linear systems. enter a list of equations as 1st argument
and a list of variables as 2nd argument, example:
linsolve([x+2y=3,x-y=7],[x,y])

Run desolve to solve exactly a differential equation. for example, to solve
y' = 2y, type desolve(y’=2y).
Another example with an initial condition:
desolve([y’=2y,y(0)=1],x,y)
Run odesolve for approx solving or plotode for a graphic representation of
the approx. solution.

rsolve solves some recurrence relations un+1 = f(un; :::), for example to
solve the arithmetico-geometric recurrence un+1 = 2un + 3; u0 = 1, type:
rsolve(u(n+1)=2*u(n)+3,u(n),u(0)=1)

## Arithmetic

iquo(a,b), irem(a,b) quotient and remainder of euclidean division of two
integers.
iquo(23,13),irem(23,13)
1; 10
 isprime(n) checks whether n is prime. This is a probabilisitic test for large
values of n.
isprime(2^64+1)

ifactor(n) factorizes an integer (not too large, since algorithms used are trial
division and Pollard-, there is no space left in memory for quadratic sieve), for
example
ifactor(2^64+1)
67280421310721  274177
Shortcut !then * (=>*)
 gcd(a,b), lcm(a,b) GCD and LCM of two integers or polynomials.
gcd(25,15),lcm(25,15)

gcd(x^3-1,x^2-1),lcm(x^3-1,x^2-1)

iegcd(a,b) returns 3 integers u; v; d such that au + bv = d where d is the
GCD of a et b, juj < jbj and jvj < jaj.
u,v,d:=iegcd(23,13); 23u+13v
[4;-7; 1] ; 1

## Polynomials

From F4 catalog, select `Polynomials`. The default variable is x�, otherwise you can specify it as last optional argument. For example `degree(x^2*y)` or `degree(x^2*y,x)` return 2, `degree(x^2*y,y)` returns 1.

-   `coeff(P,n)` coefficient of xn�� in P�, `lcoeff(P)` leading coefficient of P�, for example  
    exe  
    
-   `degre(P)` degree of polynomial P� exe  
    
-   `quo(P,Q)`, `rem(P,Q)` quotient and remainder of euclidean division of `P` by `Q`  
    exe  
    
-   `proot(P)` : approx. roots of P� (all roots, real and complex)  
    exe  
    Graphic representation  
    exe  
    
-   `interp(X,Y)` : for two lists of the same size, returns the interpolating polynomial P� such that P(Xi)=Yi�(��)=��.  
      
    exe  
    Graphic representation  
    exe  
    
-   `resultant(P,Q)` : resultant of polynomials P� and Q� exe  
    
-   `hermite(x,n)` : n�-th Hermite polynomial (orthogonal for the density e−x2dx�−�2�� on RR)
-   `laguerre(x,n,a)` : n�-th Laguerre polynomial
-   `legendre(x,n)` : n�-th Legendre polynomial (orthogonal for the density dx�� on [−1,1][−1,1])
-   `tchebyshev1(n)` and `tchebyshev2(n)` Tchebyshev polynomials of 1st and 2nd kind defined by :Tn(cos(x))=cos(nx),Un(cos(x))sin(x)=sin((n+1)x)
- 
### 3.5  Linear algebra, vectors, matrices

Xcas does not make distinction between vectors and lists. For example,
v:=[1,2]; w:=[3,4]
exe[1,2],[3,4]  
defines 2 vectors v� and w�, then `dot` will compute the scalar product of v� and w�:

dot(v,w)

exe  

A matrix is a list of lists of the same size. You can enter a matrix element by element using the matrix editor (shift-MATR EXE or F6 0). Enter a new variable name to create a new matrix or the name of an existing variable to edit a matrix. The `,` key may be used to insert a line or column, and the `DEL` key erases the line or column of the selection (press `UNDO` if you want to go one step back). For small matrices, it is also convenient to enter them directly in the commandline, for example to defineA=(1234)�=(1234)type

exe[[1,2],[3,4]]  
or  
exe  
It is recommended to store matrices in variables!

If a matrix is defined by a formula, then it’s better to use the `matrix` command (shift-MATR EXE AC), for example:  
exe  
returns the matrix where coefficient line j� and column k� is 1j+k+11�+�+1 (beware, indices begin at 0).

Run `idn(n)` to get the identity matrix of order n� and `ranm(n,m,law,[parameter])` to get a matrix with random coefficients with dimensions n,m�,�. for example  
exe  
  
exe  

For basic arithmetic on matrices, use keyboard operators (`+ - *`, inverse). Otherwise, open catalog and select `Matrices`

-     
    exe  
      
    exe  
    eigenvalues and eigenvectors of matrix A�.
-     
    exe  
    finds the Jordan normal form of matrix A�, returns matrices P� and D� such that P−1AP=D�−1��=�, with D� upper triangular (diagonal if A� is diagonalizable)
-     
    exe  
    computes matrix A� to the k�-th power, where k� is symbolic.
-   `rref`: row reduction to echelon form
-   `lu`: LU�� factorization of matrix A�, returns a permutation P� and two matrices L� (lower) and U� (upper) such that PA=LU��=��. The result of exe  
    may be passed as an argument to the command exe  
    to solve a system Ax=b��=� by solving two triangular systems (in O(n2)�(�2) instead of O(n3)�(�3)).
-   `qr` QR�� factorization of matrix A�, Q� is orthogonal and R� upper triangular, A=QR�=��.
-   `svd(A)` singular value decomposition of matrix A� returns U� orthogonal, S� vector of singular values, Q� orthogonal such that `A=U*diag(S)*tran(Q)`.  
    The ratio of the largest and the smallest singular value of S� is the condition number of A� relative to the Euclidean norm.

## 4  Probabilities and statistics

### 4.1  Random numbers

From F4 catalog, select `Probabilities` then exe  
(real in [0,1)[0,1)) or exe  
(integer between 1 and n�). Other commands with prefix `rand` are available, followed by the name of the law, for example `randbinomial(n,p)` returns a random integer according to binomial law of parameters n,p�,�. For a random vector or matrix, run `ranv` or `ranm` (from `Alglin, Matrice` submenu), for example for a vector with 10 random reals according to normal law (mean 0, stddev 1), type  
  
exe  

### 4.2  Probabilities

From F4 catalog, select `Probabilities` (8). There you will find a few distribution laws: `binomial`, `normald`, `exponentiald` and `uniformd`. Other distribution must be keyed in: `chisquared`, `geometric`, `multinomial`, `studentd`, `fisherd`, `poisson`.

To get the cumulated distribution function, enter the law name then the `_cdf` suffix (shortcut: select `cdf` in the catalog at the end and press F1). Inverse cumulated distribution function follows the same principle with `_icdf` suffix (shortcut: select `cdf` in the catalog and press F2).

Example : find the centered interval I� for the normal law of mean 5000, standard deviation 200, such that the probability to be outside I� is 5% exe  

### 4.3  1-d statistics

The statistic functions are taking lists as arguments, for example

exe[9,11,6,13,17,10]  
From F4 catalog, select `Statistics`:

-   exe  
     : arithmetic mean of a list
-   exe  
     : standard deviation of a list.  
    Run exe  
    to get an unbiaised estimate of the standard deviation of a population from a sample `l`
-   exe  
    , exe  
    , exe  
    returns respectivly the median, first and third quartile of a list.

For 1-d statistics with frequencies, replace `l` by two lists of the same length, the first list being the values of the serie, the second list the frequencies. For graphic representations, open catalog, `Graphic` and select `histogram` or `barplot`.

### 4.4  2-d statistics

From F4 catalog, select `Statistics`:

-   `correlation(X,Y)`: correlation of two lists X� and Y� of the same length.
-   `covariance(X,Y)`:: covariance of two lists X� and Y� of the same length.
-   regression computations: run commands with suffix `_regression(X,Y)`, for example `linear_regression(X,Y)` returns coefficients m,p�,� of the linear regression line y=mx+p�=��+�.
-   `linear_regression_plot(X,Y)` and all commands of suffix `_regression_plot` will display the line (or curve) of the regression. These commands will also print the R2�2 coefficient that give information on the quality of adjustment (R2�2 near 1 is good).

## 5  Graphics

From F4 catalog, select `Graphics` (or type the plot shortcut F3) and choose a graphic command. If you want to have more than one curve on the same graphic, enter several commands with `;` as separator.

-   `plot(f(x),x=a..b)` plot expression f(x)�(�) for x∈[a,b]�∈[�,�]. Discretization option: `xstep=`, for example exe  
    Default is 384 evaluations per plot (one per horizontal pixel).
-   `plotseq(f(x),x=[u0,a,b])` webplot for a recurrent sequence un+1=f(un)��+1=�(��) of first term u0�0, for example if un+1=√2+un,u0=6��+1=2+��,�0=6, with a plot on [0,7][0,7] exe  
    
-   `plotparam([x(t),y(t)],t=tm..tM)` parametric plot (x(t),y(t))(�(�),�(�)) for t∈[tm,tM]�∈[��,��]. Discretization option: `tstep=`. Example exe  
    
-   `plotpolar(r(theta),theta=a..b)` polar plot of r(θ)�(�) for θ∈[a,b]�∈[�,�], for example exe  
    
-   `plotlist(l)`: plot a list `l`, i.e. draws a polygonal line with vertices (i,li)(�,��) (index i� starts at 0).  
    `plotlist([X1,Y1],[X2,Y2],...)` polygonal line with vertices the points of coordinates (Xi,Yi)(��,��)
-   `scatterplot(X,Y)`, `polygonscatterplot(X,Y)` for two lists `X,Y` of the same size, draws the points or a polygonal line of vertices (Xi,Yi)(��,��)
-   `histogram(l,class_min,class_size)` plots the histogram of data in `l`, class size `class_size`, first class starts at `class_min`. Example: check the random generator quality  
    exe  
    
-   `plotcontour(f(x,y),[x=xmin..xmax,y=ymin..ymax],[l0,l1,...])` plot implicit curves f(x,y)=l0,f(x,y)=l1,...�(�,�)=�0,�(�,�)=�1,....
-   `plotfield(f(t,y),[t=tmin..tmax,y=ymin..ymax])` plot the field of tangents for the differential equation y'=f(t,y)�′=�(�,�). Add the optional last parameter `,plotode=[t0,y0]` to plot simultaneously the solution with initial condition y(t0)=y0�(�0)=�0. Example y'=sin(ty)�′=sin(��) for t∈[−3,3]�∈[−3,3] and y∈[−2,2]�∈[−2,2]  
    exe  
    N.B.: `plotode` may be used outside of `plotfield`.
-   For simultaneous plots, write commands separated by `;`

If a command returns an arc of curve, the trace mode is active by default on the addin in 2 parts for the FXCG50 or on the addin for the monochrom model. In this mode, typing the left or right key will move a pointer on the active arc of curve, and display the pointer coordinates (and the parameter value for parametric curves), and a tangent vector (speed). If there are several curves, press up or down to change active curve. Type F2 to get info on the active curve, then EXE to see a table of value.

From F1 menu, if you select the curve study item (shortcut x,θ,t�,�,� key), you can add a normal vector pointing to the center of curvature (F4), and/or the osculating circle and radius of curvature. From the curve study menu, you can also move the pointer to a location, or to a remarkable point: root, horizontal or vertical tangent, inflexion point, intersection with another curve arc (limited to intersection of function graphs on monochrom Casio). You can also compute an arc length or the area under the curve between pointer and mark.

When a curve is active, the variables `X0,X1,X2,Y0,Y1,Y2` are set with the expressions of position, speed and acceleration. When you search a root, horizontal tangent, inflexion, arc length or area under curve, variables are set with the last value.

For display options, press the OPTN key:

-   `display=color` color option: select a color then press F2, for example exe  
    
-   `display=line_width_2` to `display=line_width_8`: change segments width (including inside polygonal line used to plot a curve). Simultaneous display options should be added with `+`. For example `display=red+line_width_2`
-   Circles and rectangles with edges parallel to the coordinate axis may be filled with `display=filled` (this attribute might be added to other attributes)
-   If you want to define the display window (overwriting the autoscale computation), select `gl_x` or/and `gl_y` and add an x� or y� interval, for example exe  
    Note that `gl_` commands must preced the plotting command.
-   If you want to remove axes, select `axes` and press F2 (`axes=0`). Like `gl_` commands, `axes=0` must preced the plotting command. Axes can be removed interactively when the graph screen is displayed by pressing SIN.

## 6  Programs

You can program either with Xcas-like syntax (English or French) or with Python-like syntax.

Example : function defined by an algebraic expression `nom_fonction(parametres):=expression` for example simple confidence interval for a frequency p� in a sample of size N�  
exe  
  
Test exe  

Second example : more precise confidence interval for a frequency p� in a sample of size N�:  
exe  
To avoid computing twice the same quantity, one can insert a local variable. The commandline is not well adapted to write these kinds of functions. For non algebraic functions, it is best to run the program editor. Press F6, select Script Editor, clear the editor if it is not empty (F6 Clear) and type with the help of test (F1), loop (F2) for programming structures the following program, in Xcas syntax: exe  
  
or in Python syntax: exe  
Type EXE to check the syntax. Once the program is correct, save it (F6 2), then type EXIT. Now you can call your program from the commandline like this  
`f(0.5,30)`

Third example : a loop printing integer squares from 1 to n� in Python syntax. Check that Python syntax is enabled in the F6 or shift-SETUP menu, if it is not checked, check it. Open F6 Script Editor, if there is some old script source, clear it (F6 Clear). Select `f(x):=` from F2 (or from F4, Program, function def), you should get `def f(x):`. Replace `x` by `n` (press F5 to lock the keyboard in alpha lowercase), move to the end of the line and press shift-EXE to input a newline. Type Shift-PRGM then `3 for`, then F5 J space alpha, then Shift-PRGM then `6 in range(a,b)`. Type `1,n+1)` then F1 (`:`). Type shift-EXE to insert a newline then Alpha SPACE, F4 (Cmds), EXE (`1 All`), P, R select `print` with the cursor then type EXE, type `j,j^2)` then EXE.

def f(n):
  for j in range(1,n+1):
    print(j,j^2)

exe  
Inside Xcas `^` means power, `**` is also accepted like in Python.

Now, type EXE (or F6, select `1. Check syntax`). If syntax is correct, you will see `Success` in the status line. Otherwise, the first error line number and token will be displayed and cursor will be positionned at the line where the error was detected. Note that the error may be before this line but it was only detected later. Note also that if you are using Python syntax compatibility, programming structures are translated into Xcas, errors are displayed after translation, therefore you might see token errors like `end` that were added by the translator.

If the program is correct, you can save it with the F6 menu (save or save as). You can run it from the commandline by pressing EXIT then for example `f(10)` should display all squares from 1 to 10.

The turtle is a nice way to learn programming. The turtle is a small robot that you can move, it handles a pen that marks its path. Type F6, Script Editor, then F6 Clear. Type shift-QUIT select `efface` which means clear the screen. You can access to the turtle commands using shift-QUIT (move the cursor to a command and press F6 for help). For example try `avance` (forward). Checking the syntax (EXE) will display the turtle window moves. You can enter several moves in your script, and organize them inside tests, loops and functions. For example:

function square(n)
  repete(4,avance n,tourne_gauche);
ffunction:;
  
efface;
for n from 1 to 10 do
  square(10*n);
od;

Another example of non algebraic function: the euclidean algorithm to compute the GCD of two integers. Press shift-EXE to insert a newline. `!` is in the submenu `Programmation_cmds` (11, shortcut X,θ,T�,�,�) or in the test F1 menu. Xcas syntax

function pgcd(a,b)
  while b!=0 do
    a,b:=b,irem(a,b);
  od;
  return a;
ffunction

exe  
Python syntax

def pgcd(a,b):
  while b!=0:
    a,b=b,a % b
  return a

exe  
Check with exe  

If your program has runtime errors or if you want to see it run step by step, run `debug` on it, for example  
`debug(pgcd(12345,3425))`

Unlike adaptations of Micro-Python by calculator manufacturers (including Casio), the Python syntax in Xcas is fully integrated. You can therefore use all Xcas commands and data types in your programs. This corresponds approximatively to importing Python modules `math`, `cmath`, `random`, `scipy`, `numpy`, `turtle`, `giacpy`. There is also a small pixelised graphic commands set (`set_pixel(x,y,c)`, `set_pixel()` to synchronize display, `clearscreen()`, `draw_line(x1,y1,x2,y2,c)`, `draw_polygon([[x1,y1],[x2,y2],...],c)`, `draw_rectangle(x,y,w,h,c)`, `draw_circle(x,y,r,c)`, the color+width+filled `c` parameter is optional, `draw_arc(x,y,rx,ry,t1,t2,c)` draws an ellipsis arc). And you can somewhat replace `matplotlib` with graphic commands of χ�CAS (`point`, `line`, `segment`, `circle`, `barplot`, `histogram` and all `...plot...` commands). Plus you have natural access to data types like rationnals or expressions, and you can run CAS commands on them. The complete list of commands available on the calculator is given in appendix. For documentation on commands not listed in the catalog categories, please refer to Xcas documentation.

## 7  The 2d editor.

If a computation returns an expression, it will be displayed in the 2d expression editor. This also happens if you press F3 when the selected level is an expression, or if you press F3 from the commandline if the line is empty or contains a syntaxically correct expression.

Once the 2d editor is open, the expression is displayed in full screen and all or part of the expression is selected. One can run a command on the selection (from the menus or from the keyboard), or edit (in 1d mode) the selection. This is an efficient way to rewrite expressions or edit them.

Example 1 : enterlimx→0sin(x)xlim�→0sin(�)�From an empty commandline, type F3 (view), you should see 0 selected. Type x then EXE, this will replace 0 by x selected. Type SIN, now sin(x)sin(�) should be selected. Type the division key (above -), you should see sin(x)0sin(�)0 with 0 selected, type x then EXE, you should now see sin(x)xsin(�)� with x (below the fraction) selected. Type the up arrow key, now sin(x)xsin(�)� should be selected. Now type F2 4 (for limit). The expression is ready to eval, type EXE to copy it to the commandline and EXE again to eval it. For the same limit at +∞+∞, before leaving the 2d editor with EXE, move the selection with the right arrow key, then type F1 8 (oo) EXE.

Example 2 :∫+∞01x4+1dx∫0+∞1�4+1��From an empty commandline, type F3 (view), then F2 3 (integrate), you should see:∫100dx∫010��with x� selected. We must modify the 1 (upper bound) and the 0 (integrand). Press left arrow key, this will select the integrand 0, type `1/(x^4+1)` EXE, then left arrow key F1 8 EXE. Type again EXE to copy to commandline, EXE again to run the computation, the result will be displayed in the 2d editor, EXE will leave the 2d editor, with the integral and its value in the history.

Example 3 : compute and simplify∫1x4+1dx∫1�4+1��From an empty commandline, type F3 (view), then F2 3 (integrate), you should see∫100dx∫010��Move the selection to the lower bound 0 (right arrow key), type DEL, you should see∫0dx∫0��selected. With the down arrow key, select 0, type `1/(x^4+1)` EXE, EXE copy to the commandline, EXE to run the compuation, the result is now displayed in the 2d editor.  
With the arrow key, select one of the arctangent, type F1 EXE (simplify), this will make a partial simplification, do the same on the second arctangent.  
For a more complete simplification, we will collect the logarithms. The first step is to exchange two terms of the main sum so that the logarithms are grouped. Select one of the logarithm with the arrow keys, then type

-   CG10,20,50 : shift-left or right arrow key
-   fx-9860GIII: F5 left or right arrow key, then ALPHA

this will exchange the selection with the right or left sibling. Now type ALPHA right or left arrow key to extend the selection adding the right or left sibling. Once the two logarithm terms are selected, press F1 2 EXE (factor), decrease the selection to the numerator, type F4 EXE (All), type the letters l, n, c, this moves in the catalog to the first command beginning with `lnc`, select `lncollect`, EXE and F6 (eval).

## 8  Managing sessions

### 8.1  Modifying a session

With the up/down cursor keys, you can move in the history, the current level is printed with reverse colors.

You can move one level in another position with ALPHA-up and ALPHA-down. You can delete a level with the DEL key (the level is copied into the clipboad).

You can modify an existing level with F3 or ALPHA-F3. With F3, the 2d editor is called if the level is an expression, with ALPHA-F3 the level is edited in the text (program) editor. Type EXIT if you want to cancel modifications, or EXE if you confirm the modifications. If you confirm the modifications, the commandlines below the current level will automatically be re-evaled. This way, if you modify for example a level like `A:=1`, all levels below that depend on the value of `A` will be up to date. If you want to do that several times, it is best to introduce a parameter with the F6 Parameter wizzard. Then pressing + or - on the `assume(...)` or `parameter` level will modify the value of the parameter (press * or / for faster move).

### 8.2  Variables

Press VARS to see which variables are assigned to a value. Select a variable name, press EXE to copy it to the commandline, DEL will input the command that erases the variable (confirm with EXE). `restart` will purge all variables at once (press `AC/ON` to clear the history and start a fresh new session). `assume` is a command to make assumptions on a variable, like `assume(x>5)` (`>` can be accessed from the shift-PRGM menu).

### 8.3  Archiving and exchanging with Xcas

On the calculator, go back to the history (type EXIT if you are in the programming editor or the 2d expression editor). From the F6 menu, you can save/restore sessions in the calculator flash memory. Files have the `xw` extensions. They can be copied to your computer (connect the calc, choose F1 USB key), and there they may be opened with Xcas or Xcas for Firefox. From Xcas, choose the File menu then Open file, then select all type of files and open the session file. From [Xcas for Firefox](https://www-fourier.univ-grenoble-alpes.fr/~parisse/xcasfr.html), press the Load button.

Conversely you can save a session from Xcas (choose File, Export to Khicas) or from Xcas for Firefox (choose Export at the right of the session



