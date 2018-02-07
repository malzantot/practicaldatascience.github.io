---
layout: page
title: Vectors, matrices, and linear algebra
img: matrix.svg
---

[Download notes as Jupyter notebook](matrices.tar.gz)

## Introduction

Vector and matrices play a central role in data science: they are probably the most common way of representing data to be analyzed and manipulated by virtually any machine learning or analytics algorithm.  However, it is also important to understand that there really two uses to matrices within data science:

1. Matrices are the "obvious" way to store tabular data (particularly when all the data is numeric) in an efficient manner.
2. Matrices are the foundation of linear algebra, which the "language" of most machine learning and analytics algorithms.

In is important to understand these two points but also the differences between them.  Under the first interpretation, matrices are essentially 2D arrays (vectors are 1D arrays, and tensors are higher-dimensional arrays); this view is fundamentally a take on how to efficiently store data in the multi-dimensional arrays.  But matrices are also the basic unit of linear algebra, which is a mathematical language for the expression and manipulation of linear systems of equations.  There are naturally overlaps between the two, but the core operations of linear algebra, such as matrix multiplication and solving linear systems of equations, are largely orthogonal to the way in which matrices are stored as arrays in memory.  Note: It is also the case that most (but definitely not all) treatments of tensors is data science actually don't do much of the second: tensors are often a nice way to store higher-dimensional tabular data, but the actual linear algebra of tensors is relatively rare despite the recent uptick of the term "tensor" in data science.

These notes will take us first through the "array" view of vectors and matrices, then to the "linear algebra" view.  We will then learn the basics of the [numpy](http://www.numpy.org/) library to manipulate numpy arrays both as arrays and as matrices.  We will finish by discussing sparse matrices, which are particularly crucial for many data science applications.

## Vectors and matrices: the "array" view

Vectors are 1D arrays of values.  We use the notation $x \in \mathbb{R}^n$ to denote that $x$ is a vector with $n$ entries and in this case the entries are all real valued.  We can consider other types of values for vectors (and in code we will commonly create vectors of Booleans or integers), but as mathematical objects it's most common to use real numbers.  We can write the elements of a vector more explicitly like so

$$
x = \left[\begin{array}{c} x_1 \\ x_2 \\ \vdots \\ x_n \end{array} \right ]
$$

where we use $x_i$ to denote the $i$th element of the vector.  We also note that as far as the mathematical representation of vectors goes, we will consider vectors by default to be _column_ vectors (matrices with one column and many rows), as opposed to row vectors; but for example, the Numpy library doesn't make this distinction, so this will be largely for mathematical reasons.  If we want to denote a row vector, we'll use the notation $x^T$ ($x$ transposed, for the transpose operator we'll define shortly).

Matrices are 2D arrays of values, and we use the notation $A \in \mathbb{R}^{m \times n}$ to denote a matrix with $m$ rows and $n$ columns.  We can write out the elements explicitly as follows

$$
A = \left[\begin{array}{cccc} 
A_{11} & A_{12} & \cdots & A_{1n} \\ 
A_{21} & A_{22} & \cdots & A_{2n} \\ 
\vdots & \vdots & \ddots & \vdots \\ 
A_{m1} & A_{m2} & \cdots & A_{mn}
\end{array} \right ]
$$

where $A_ij$ denotes the entry of $A$ in the $i$th row and $j$th column.  We will use the notation $A_{i:}$ to refer to the $i$th row of $A$ and the $A_{:j}$ to refer to the $j$th column of $A$ (whether these represent row or column vectors depends somewhat on the situation, but we will explicitly clarify which it means whenever we use such notation).

There are also higher order generalizations of matrices (called tensors), which represented 3D or higher arrays of values.  These are in fairly common use in modern data science, though typically (but certainly not always), tensors are just used in the "multi-dimensional array" sense, not in their true linear algebra sense.  Tensors as linear operators that act on e.g. matrices or other higher-order tensors, are slightly less common most basic data science, and are a more advanced topic that is outside the scope of this course.

### Matrices for tabular data and row/column ordering

Let's start with a simple example representing tabular data using matrices, one of the more natural ways to represent such data (and as we will see in later lectures, one of the ways that lends itself well to use in machine learning).  Let's consider the "Grades" table that we previously discussed in our presentation of relational data:

| Person ID | HW1 Grade | HW2 Grade |
| :---: | :---: | :---: |
| 5 | 85 | 95 | 
| 6 | 80 | 60 |
| 100 | 100 | 100 |

Ignoring the primary key column (this is not really a numeric feature, so makes less sense to store as a real-valued number), we could represent the data in the table via the matrix 

$$
A \in \mathbb{R}^{3 \times 2} = \left[ \begin{array}{cc} 85 & 95 \\ 80 & 60 \\ 100 & 100 \end{array} \right ]
$$


While there seems to be little else that can be said at this point, there is actually a subtle but important point about how the data in this table is really laid out in memory.  Since data in memory is laid out sequentially (at least logically as far as programs are concerned, if not physically on the chip) we can opt to store the data in _row major order_, that is, storing each row sequentially

$$
(85, 95, 80, 60, 100, 100)
$$

or in _column major order_, storing each column sequentially

$$
(85, 80, 100, 95, 60, 100)
$$


Row major ordering is the default for 2D arrays in C (and the default for the Numpy library), while column major ordering is the default for FORTRAN.  There is no obvious reason to prefer one over the order, but due to the cache locality in CPU memory hierarchies, the different methods will be able to access the data more efficiently by row or by column respectively.  Most importantly, however, the real issue is that because a large amount of numerical code was originally (and still is) written in FORTRAN, if you ever want to call external numerical code, there is a good chance you'll need to worry about the ordering.


## Basics of linear algebra

In addition to serving as a method for storing tabular data, vector and matrices also provide a method for studying sets of linear equations.  These notes here are going to provide a very brief overview and summary of some of the primary linear algebra notation that you'll need for this course.  But it is really meant to be a refresher for those who already have some experience with linear algebra previously.  If you do not, then I have previously put out an online mini-course covering (with notes) some of the basics of linear algebra: [Linear Algebra Review](http://www.cs.cmu.edu/~zkolter/course/linalg/).  This course honestly goes a bit beyond what is needed for this particular course, but it can act a good reference.

Consider the following two linear equations in two variables $x_1$ and $x_2$.


$$
\begin{split}
4 x_1 - 5 x_2 & = -13 \\
-2 x_1 + 3 x_2 & = 9
\end{split}
$$


This can written compactly as the equation $A x = b$, where

$$
A \in \mathbb{R}^{2 \times 2} = \left [ \begin{array}{cc} 4 & -5 \\ -2 & 3 \end{array} \right ], 
\;\; b \in \mathbb{R}^2 = \left [ \begin{array}{c} -13 \\ 9 \end{array} \right ], 
\;\; x \in \mathbb{R}^2 = \left [ \begin{array}{c} x_1 \\ x_2 \end{array} \right ].
$$

As this hopefully illustrates, one of the entire points of linear algebra is to make the notation and math _simpler_ rather than more complex.  However, until you get used to the conventions, writing large equations where the size of matrices/vectors are always implicit can be a bit tricky, so you'll some care is needed to make sure you do not include any incorrect derivations.


### Basic operations and special matrices

**Addition and substraction**: Matrix addition and subtraction are applied elementwise over the matrices, and can only apply two two matrices of the same size.  That is, if $A \in \mathbb{R}^{m \times n}$ and $B \in \mathbb{R}^{m \times n}$ then their sum/difference $C = A + B$ is another matrix of the same size $C \in \mathbb{R}^{m \times n}$ where

$$
C_{ij} = A_{ij} + B_{ij}.
$$


**Transpose**: Transposing a matrix flips its rows and columns.  That is, if $A \in \mathbb{R}^n$, then it's transpose, denoted $C = A^T$ is a matrix $C \in \mathbb{R}^{n \times m}$ where

$$
C_{ij} = A_{ji}.
$$


**Matrix multiplication**: Matrix multiplication is a bit more involved.  Unlike addition and subtraction, matrix multiplication does not perform elementwise multiplication of the two matrices.  Instead, for a matrix $A \in \mathbb{R}^{m \times n}$, $B \in \mathbb{R}^{n \times p}$ (note these precise sizes, as they are important), their product $C = AB$ is a matrix $C \in \mathbb{R}^{m \times p}$ where

$$
C_{ij} = \sum_{k=1}^n A_{ik} B_{kj}.
$$

In order for this sum to make sense, notice that the number of columns in $A$ must equal the number of rows in $B$.  If you'd like a bit more of the intuition behind why matrix multiplication is defined this way, the notes above provide some important interpretations.  It's important to note the following properties, though:
- Matrix multiplication is associative: $(AB)C = A(BC)$ (i.e., it doesn't matter in what order you do the multiplications, though it _can_ matter from a computational perspective, as some orderings will be more efficient to compute than others)
- Matrix multiplication is distributive: $A(B+C) = AB + AC$
- Matrix multiplication is _not_ commutative: $AB \neq BA$. This is really true in two different ways.  Under the above matrix sizes, the multiplication $BA$ is not a valid expression if $m \neq p$ (since the number of columns in $B$ would not match the number of rows in $A$).  And even if the dimensions _do_ match (for instance if all the matrices were $n \times n$) the products will still not be equal in general.

**Identity matrix**: The identity matrix $I \in \mathbb{R}^{n \times n}$ is a square matrix with ones on the diagonal an zeros everywhere else

$$
I = \left [ \begin{array}{cccc} 
1 & 0 & \cdots & 0 \\
0 & 1 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\ 
0 & 0 & \cdots & 1
\end{array} \right ].
$$


It has the property that for any matrix $A \in \mathbb{R}^{m \times n}$

$$
A I = I A = A
$$

where we note that the two $I$ matrices in the above equations are actually two _different_ sizes (the first one is $ n \times n$ and the second is $m \times m$, to make the math work right).  For this reason, some people use the notation $I_n$ to explicitly denote the size of $I$, but it is not really needed, because the size that any identity must be can be inferred from the other matrices in the equation.

**Matix inverse**: For a square matrix $A \in \mathbb{R}^{n \times n}$, the matrix inverse $A^{-1} \in \mathbb{R}^{n \times n}$ is the unique matrix such that

$$
A^{-1}A = A A^{-1} = I.
$$

The matrix inverse need not exist for all square matrices (it will depend on the linear independence between rows/columns of $A$, and we will consider such possibilities a bit later in the course.

**Solving linear equations**: The matrix inverse provides an immediate method to obtain the solution to systems of linear equations.  Recall out example above of a set of linear equations $A x = b$.  If we want to find the $x$ that satisfies this equation, we multiply both sizes of the equation by $A^{-1}$ on the left to get

$$
A^{-1}A x = A^{-1}b \Longrightarrow x = A^{-1} b.
$$

The nice thing here is that as far as we are concerned in this course, the set of equations is now _solved_.  We don't have to worry about any actual operations that you may have learned about to actually obtain this solution (scaling the linear equations by some constant, adding/subtracting them to construct a solution, etc).  The linear algebra libraries we will use do all this for us, and our only concern is getting the solution into a form like the one above.

**Transpose of matrix product**: It follows immediately from the definition of matrix multiplication and the transpose that

$$
(AB)^T = B^T A^T
$$

i.e., the transpose of a matrix product is the product of the transposes, in reverse order.

**Inverse of matrix**: It also follows immediately from the definitions that for $A,B\in \mathbb{R}^{n \times n}$ both square

$$
(AB)^{-1} = B^{-1} A^{-1}
$$

i.e. the inverse of a matrix product is the product of the inverses, in reverse order.

**Inner products**: One type of matrix multiplication is common enough that it deserves special mention.  If $x,y \in \mathbb{R}^n$ are vectors of the same dimensions, then

$$
x^T y = \sum_{i=1}^n x_i y_i
$$

(the matrix product of $x$ transposed, i.e., a row vector and $y$, a column vector) is a _scalar_ quantity called the inner product of $x$ and $y$; it is simply equal to the sum of the corresponding elements of $x$ and $y$ multiplied together.

**Vector norms**: These are only slightly related to vectors matrices, but this seems like a good place to introduce it.  We will use the notation 

$$
\|x\|_2 = \sqrt{x^T x} = \sqrt{\sum_{i=1}^n x_i^2}
$$

to denote the Euclidean (also called $\ell_2$) norm of $x$.  We may occasionally also refer to the $\ell_1$ norm 

$$
\|x\|_1 = \sum_{i=1}^n |x_i|
$$

or the $\ell_\infty$ norm

$$
\|x\|_\infty = \max_{i=1,\ldots,n} |x_i|.
$$


**Complexity of operations**:  For making efficient use of matrix operations, it is extremely important to know the big-O complexity of the different matrix operations.  Immediately from the definitions of the operations, assuming $A,B \in \mathbb{R}^{n \times n}$ and $x,y \in \mathbb{R}^n$ we have the the following complexities:
- Inner product $x^Ty$: $O(n)$
- Matrix-vector product $Ax$: $O(n^2)$
- Matrix-matrix product $AB$: $O(n^3)$
- Matrix inverse $A^{-1}$, or matrix solve $A^{-1}y$ (as we will emphasize below, these are arctually done differently; they both have the same big-O omplexity, but different concstant terms on the runtime in practice): $O(n^3)$

Note that the big-O complexity along with the associative property of matrix multiplications implies very different complexities for computing the exact same term in different ways.  For example, suppose we want to compute the matrix products $ABx$.  We could compute this as $(AB)x$ (computing the $AB$ product first, then multiplying with $x$); this approach would have complexity $O(n^3)$, as the matrix-matrix product would dominate the computation.  On the other hand, if we compute the product as $A(Bx)$ (first computing the vector product $Bx$, which produces a vector, then multiplying this by $A$), the complex is only $O(n^2)$, as we just have two matrix-vector products.  As you can imagine, these orders of operations therefore make a huge difference in terms of the time complexity of linear algebra operations.

## Numpy and software for matrices

The Numpy library is the defacto standard for manipulating matrices and vectors (and higher order tensors) from within Python.  Numpy `ndarray` objects are fundamentally multi-dimensional arrays, but the library also includes a variety of functions for processing these like matrices/vectors.  A key aspect of this library is that numpy matrices vectors are stored efficiently in memory, not via Python lists, and the operations in numpy are back by efficiently implemented linear algebra libraries.

A more complete tutorial on numpy is available here: [Numpy tutorial](https://docs.scipy.org/doc/numpy-dev/user/quickstart.html).  But these notes will introduce you to some of the basic operations you're likely to see repeatedly in many data science applications.

### Specialized linear algebra libraies

As you will hopefully appreciate throughout this course, linear algebra computations underly virtually all data science and machine learning algorithms.  Thus, making these algorithms fast is extremely important in practical applications.  And despite the seeming "simplicity" of basic linear algebra operators, the naive implementation of most algorithms will perform quite poorly.  Consider the following (in C) implementation of a matrix multiplication operator.

```c
void matmul(double **A, double **B, double **C, int m, int n, int p) {
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < p; j++) {
            C[i][j] = 0.0;
            for (int k = 0; k < n; k++)
                C[i][j] += A[i][k] * B[k][j];
        }
    }
}
```

In some sense, it seems like this _has_ to be the implementation of matrix multiply; it is just a simple translation of the mathematical definition to code.  But if you compiled this code, and profile it against an optimized linear algebra library, you can expect about 10 _times_ slower performance.  How is this happening?

It turns out that (precisely because linear algebra is so crucial), specialized linear algebra libraries (these include, for instance, [OpenBLAS](http://www.openblas.net/), [Intel MKL](https://software.intel.com/en-us/mkl), [ATLAS](http://math-atlas.sourceforge.net/) or [Eigen](http://eigen.tuxfamily.org/)) have gone to great lengths to exploit the custom vector processors, plus the cache hierarchy of specific architectures, to make libraries that are well-tuned to each different CPU (for example, they typically use "chunking" methods to break down matrix multiplications into smaller pieces that are better suited to exploit cache locality).  And the difference in speed between these and the more naive algorithms are extremely striking.

This is also one of the reasons why we recommend that everyone still Anaconda as their Python distribution.  Anaconda comes with versions of Numpy that are compiled with a fast linear algebra library backing it.  Chances are, if you compile your Numpy library locally, you will not link to these fast libraries, and the speed of your matrix-based code will suffer substantially from it.

To check to see if Numpy is using specialized libraries, use the command:


```python
import numpy as np
np.__config__.show()
```

```
blas_mkl_info:
    libraries = ['mkl_rt', 'pthread']
    library_dirs = ['/Users/zkolter/anaconda3/lib']
    define_macros = [('SCIPY_MKL_H', None), ('HAVE_CBLAS', None)]
    include_dirs = ['/Users/zkolter/anaconda3/include']
blas_opt_info:
    libraries = ['mkl_rt', 'pthread']
    library_dirs = ['/Users/zkolter/anaconda3/lib']
    define_macros = [('SCIPY_MKL_H', None), ('HAVE_CBLAS', None)]
    include_dirs = ['/Users/zkolter/anaconda3/include']
lapack_mkl_info:
    libraries = ['mkl_rt', 'pthread']
    library_dirs = ['/Users/zkolter/anaconda3/lib']
    define_macros = [('SCIPY_MKL_H', None), ('HAVE_CBLAS', None)]
    include_dirs = ['/Users/zkolter/anaconda3/include']
lapack_opt_info:
    libraries = ['mkl_rt', 'pthread']
    library_dirs = ['/Users/zkolter/anaconda3/lib']
    define_macros = [('SCIPY_MKL_H', None), ('HAVE_CBLAS', None)]
    include_dirs = ['/Users/zkolter/anaconda3/include']
```

Your output may not be exactly the same as what is shown here, but you should be able to infer from this if you're using an optimized library like (int this case), Intel MKL.

### Creating numpy arrays

The `ndarray` is the basic data type in Numpy.  These can be created the `numpy.array` command, passing a 1D list of number to create a vector or a 2D list of numbers to create an array.


```python
b = np.array([-13,9])            # 1D array construction
A = np.array([[4,-5], [-2,3]])   # 2D array contruction
print(b, "\n")
print(A)
```

```
[-13   9] 

[[ 4 -5]
 [-2  3]]
```

There are also special functions for creating arrays/matrices of all zeros, all ones, or of random numbers (in this case, the `np.randon.randn` create a matrix with standard random normal entries, while `np.random.rand` creates uniform random entries).


```python
print(np.ones(4), "\n")           # 1D array of ones
print(np.zeros(4), "\n")          # 1D array of zeros
print(np.random.randn(4))         # 1D array of random normal numbers
```

```
[ 1.  1.  1.  1.] 

[ 0.  0.  0.  0.] 

[-0.65826018 -0.48547552 -0.12390373  0.51937501]
```


```python
print(np.ones((3,4)), "\n")       # 2D array of ones
print(np.zeros((3,4)), "\n")      # 2D array of zeros
print(np.random.randn(3,4))       # 2D array of random normal numbers
```

```
[[ 1.  1.  1.  1.]
 [ 1.  1.  1.  1.]
 [ 1.  1.  1.  1.]] 

[[ 0.  0.  0.  0.]
 [ 0.  0.  0.  0.]
 [ 0.  0.  0.  0.]] 

[[-0.92108207 -0.0840208  -1.49748471  0.1484692 ]
 [-0.80504092  0.47344881  0.96519561  1.02125684]
 [ 0.07350312 -0.52083043 -0.42326075  0.71938146]]
```

Note that (in a design that will forever frustrate me), the size of the array is passed as a tuple to `np.ones()` and `np.zeros()`, but as a list of arguments to `np.random.randn()`.

You can also create the indentity matrix using the `np.eye()` command, and a diagonal matrix with the `np.diag()` command.


```python
print(np.eye(3),"\n")                     # create array for 3x3 identity matrix
print(np.diag(np.random.randn(3)),"\n")   # create diagonal array
```

```
[[ 1.  0.  0.]
 [ 0.  1.  0.]
 [ 0.  0.  1.]] 

[[ 0.75084381  0.          0.        ]
 [ 0.         -0.17133185  0.        ]
 [ 0.          0.         -1.09201859]] 

```

### Indexing into numpy arrays

You can index into Numpy arrays in many different ways.  The most common is to index into them as you would a list: using single indices and using slices, with the additional consideration that using the `:` character will select the entire span along that dimension.


```python
A = np.array([[1,2,3], [4,5,6], [7,8,9], [10, 11, 12]])
print(A, "\n")
print(A[1,1],"\n")           # select singe entry
print(A[1,:],"\n")           # select entire row
print(A[1:3, :], "\n")       # slice indexing
```

```
[[ 1  2  3]
 [ 4  5  6]
 [ 7  8  9]
 [10 11 12]] 

5 

[4 5 6] 

[[4 5 6]
 [7 8 9]] 

```

Note the convention here in terms of the sizes returned: if we select a single entry, then we get back the value of that entry (not a 1D/2D array with just a singleton element).  If we select a single row or a single column from a 2D array we get a _1D array_ with that row or column.  And if we select a slice and/or the entire row/column along both dimensions, we get a 2D array.  This takes a while to get used to, but if, for example, we wanted to get a _2D array_ containing just the (1,1) element, we could use the code.


```python
print(A[1:2,1:2])  # Select A[1,1] as a singleton 2D array
```

```
[[5]]
```

Numpy also support fancier indexing with _integer_ and _Boolean_ indexing.  If we create another array or list of indices (that  is, for the rows in above array, this would be integers between 0-3 (inclusive)), then we can use this list of integers to select the rows/columns we want to include.


```python
print(A[[1,2,3],:])  # select rows 1, 2, and 3
```

```
[[ 4  5  6]
 [ 7  8  9]
 [10 11 12]]
```

Note that these integer indices do not need to be in order, nor do they have to include at most once instance of each row/column; we can use this notation to repeat rows/columns too.


```python
print(A[[2,1,2],:])  # select rows 2, 1, and 2 again
```

```
[[7 8 9]
 [4 5 6]
 [7 8 9]]
```

Note that we can also use an array of integers instead of a list, for the same purpose.


```python
print(A[np.array([2,1,2]),:])  # select rows 2, 1, and 2 again
```

```
[[7 8 9]
 [4 5 6]
 [7 8 9]]
```

Last, we can also use Boolean indexing.  If we specify a list or array of booleans that is the _same size_ as the corresponding row/column, then the Boolean values specify a "mask" over which values are taken.


```python
print(A[[False, True, False, True],:])  # Select 1st and 3rd rows
```

```
[[ 4  5  6]
 [10 11 12]]
```

As a final note, be careful if you try to use integer or boolean indexing for both dimensions.  This will attempt to select generate a 1D array of entries with both those locations.


```python
print(A[[2,1,2],[1,2,0]])    # the same as np.array([A[2,1], A[1,2], A[2,0]])
```

```
array([8, 6, 7])
```

If you actually want to first select based upon rows, then upon columns, you'll do it like the following, essentially doing each indexing separately.


```python
A[[2,1,2],:][:,[1,2,0]]
```

```
array([[8, 9, 7],
       [5, 6, 4],
       [8, 9, 7]])
```

### Basic operations on arrays

Arrays can be added/subtracted, multiplied/divided, and transposed, but these are _not_ all the same as their linear algebra counterparts.


```python
B = np.array([[1, 1, 1], [1,2,1], [3, 1, 3], [1, 4, 1]])
print(A+B, "\n") # add A and B elementwise (same as "standard" matrix addition)
print(A-B) # subtract B from A elementwise (same as "standard" matrix subtraction)
```

```
[[ 2  3  4]
 [ 5  7  7]
 [10  9 12]
 [11 15 13]] 

[[ 0  1  2]
 [ 3  3  5]
 [ 4  7  6]
 [ 9  7 11]]
```

Array multiplication and division are done _elementwise_, they are _not_ matrix multiplication or anything related to matrix inversion.


```python
print(A*B, "\n") # elementwise multiplication, _not_ matrix multiplication
print(A/B, "\n") # elementwise division, _not_ matrix inversion
```

```
[[ 1  2  3]
 [ 4 10  6]
 [21  8 27]
 [10 44 12]] 

[[  1.           2.           3.        ]
 [  4.           2.5          6.        ]
 [  2.33333333   8.           3.        ]
 [ 10.           2.75        12.        ]] 

```

You can transpose arrays, but note this _only_ has meaning for 2D (or higher) arrays.  Transposing a 1D array doesn't do anything, since Numpy has no notion of column vectors vs. row vectors for 1D arrays.


```python
x = np.array([1,2,3,4])
print(A.T, "\n")
print(x, "\n")
print(x.T)
```

```
[[ 1  4  7 10]
 [ 2  5  8 11]
 [ 3  6  9 12]] 

[1 2 3 4] 

[1 2 3 4]
```

### Broadcasting

Things start to get very fun when you add/substract/multiply/divide array of _different_ sizes.  Rather than throw an error, Numpy will try to make sense of your operation using the [Numpy broadcasting rules](https://docs.scipy.org/doc/numpy-1.13.0/user/basics.broadcasting.html).  This is an advanced topic, which often really throws off newcomers to Numpy, but with a bit of practice the rules become quite intuitive.  

Broadcasting generally refers to how entries from the small array are "broadcast" to the larger array.  The basic rule is as follows: first, suppose that two Numpy arrays `A` and `B` have the same number of dimensions (e.g., they are both 2D arrays).  But suppose that along one of these dimensions `B` is only size 1. In that case, the the dimension of size 1 is `B` is automatically expanded (repeating all entries along that dimension), and then the operation between the two arrays is applied.  Let's look at a simple example:


```python
A = np.ones((4,3))          # A is 4x3
x = np.array([[1,2,3]])      # x is 1x3
A*x                          # repeat x along dimension 4 (repeat four times), and add to A
```

```
array([[ 1.,  2.,  3.],
       [ 1.,  2.,  3.],
       [ 1.,  2.,  3.],
       [ 1.,  2.,  3.]])
```

Effectively, the (1,3) array `x` (observer that it is actually a 2D array) is "resized" to (4,3), repeating its entries along the first dimension, and it is them multiplied elementwise by `A`.  The effective result of this is that the columns of `A` are rescaled by the values of `x`.  Note that no actual additional memory allocation happens, and the resize here is entirely from a conceptual perspective.  Alternatively, the following code would rescale the _rows_ of `A` by `x` (where here we need to construct a (4,1) sized array is order for the broadcasting to work.


```python
x = np.array([[1],[2],[3],[4]])
A*x
```

```
array([[ 1.,  1.,  1.],
       [ 2.,  2.,  2.],
       [ 3.,  3.,  3.],
       [ 4.,  4.,  4.]])
```

Here `x` has size (4,1), so it is effectively resized to (4,3) along the second dimension, repeating values along the columns.  This has the effect of scaling the rows of `A` by `x`.

As a final note, the rule for numpy is that if the two arrays being operated upon have _different_ numbers of dimensions, we extend the dimensions in the _leading_ dimensions to all implicitly be 1.  Thus, the following code will implicitly consider `x` (which is a 1D array of size 3), to be a (1,3) array, and then apply the broadcasting rules, which thus has the same effect as our first broadcasting example.


```python
x = np.array([1,2,3])
A*x
```

```
array([[ 1.,  2.,  3.],
       [ 1.,  2.,  3.],
       [ 1.,  2.,  3.],
       [ 1.,  2.,  3.]])
```

If we want to implicitly "cast" a n sized 1D array to a (n,1) sized array, we can use the notation `x[:,None]` (we put "None" for the dimensions we want to define to be 1).


```python
x = np.array([1,2,3,4])
print(x[:,None], "\n")
print(A*x[:,None])
```

```
[[1]
 [2]
 [3]
 [4]] 

[[ 1.  1.  1.]
 [ 2.  2.  2.]
 [ 3.  3.  3.]
 [ 4.  4.  4.]]
```

These rules can be confusing, and it takes some time to get used to them, but the advantage of broadcasting is that you can compute many operations quite efficiently, and once you get used to the notation, it is actually not the difficult to understand what is happening.  For example, from a "linear algebra" perspective, the right way to scale the column of a matrix is a matrix multiplication by a diagonal matrix, like in the following code.


```python
D = np.diag(np.array([1,2,3]))
A @ D
```

```
array([[ 1.,  2.,  3.],
       [ 1.,  2.,  3.],
       [ 1.,  2.,  3.],
       [ 1.,  2.,  3.]])
```

(we will cover the matrix multiplication operator `@` in a moment).  However, actually constructing the `np.diag()` matrix is wasteful: it explicitly constructs an $n \times n$ matrix that only has non-zero elements on the diagonal, then performs a dense matrix multiplication.  It is much more efficient to simply scale `A` using the broadcasting method above, as no additional storage will be allocated, and the actual scaling operation only requires $O(n^2)$ time as opposed to the $O(n^3)$ time for a full matrix multiplication.

### Linear algebra operations

Starting with Python 3, there is now a matrix multiplication operator `@` defined between numpy arrays (previously one had to use the more cumbersome `np.dot()` function to accomplish the same thing).  Note that in the following example, all the array sizes are created such that the matrix multiplications work.


```python
A = np.random.randn(5,4)
C = np.random.randn(4,3)
x = np.random.randn(4)
y = np.random.randn(5)
z = np.random.randn(4)

print(A @ C, "\n")       # matrix-matrix multiply (returns 2D array)
print(A @ x, "\n")       # matrix-vector multiply (returns 1D array)
print(x @ z)       # inner product (scalar)
```

```
[[-2.349365    0.31307737  0.43701076]
 [-3.01521936  2.33512524 -0.59099322]
 [-0.02346425  0.0118288  -2.68179453]
 [ 0.58286024 -1.36334426  0.35011801]
 [ 0.56680928 -1.83411679  1.29601818]] 

[-0.19595591  0.22193364  0.88633042  0.40914083 -0.87358333] 

1.11337305084
```

There is an important point to note, though, here. Depending on the sizes of the arrays passed to the `@` operator, numpy will return results of different sizes: two 2D arrays result in a 2D array (matrix-matrix product), a 2D array and a 1D array result in a 1D array (matrix-vector product), and two 1D arrays result in a scalar (just a floating point number, not an `ndarray` at all).  This can cause some issues if, for instance, your code always assumes that the result of a `@` operation between two `ndarray` objects will also return an `ndarray`: depending on the size of the arrays you pass (i.e., if they are both 1D arrays), you will actually get a floating point object, not an `ndarray` at all.

The rules can be especially confusing when we think about multiplying vectors on the left of matrices, i.e., forming a matrix-vector product $y^T A$ for $y \in \mathbb{R}^m$, $A \in \mathbb{R}^{m \times n}$.  This is a valid matrix product, but since Numpy has no distinction between column and row vectors, both the following operations compute the same 1D result (i.e., which performs the above left-multplication, but return the result just as a 1D array): 


```python
print(A.T @ y, "\n")
print(y.T @ A)
```

```
[-0.20969054  1.62940281 -0.89696956 -2.29205352] 

[-0.20969054  1.62940281 -0.89696956 -2.29205352]
```

The confusing part is that because transposes have no meaning to for 1D arrays, the following code _also_ returns the same result, despite $y A$ not being a valid linear algebra expression.


```python
print(y @ A)
```

```
[-0.20969054  1.62940281 -0.89696956 -2.29205352]
```

On the other hand, trying to do the multiplication in the other order $Ay$ (which is also not a valid linear algebra expression), does throw an error.


```python
print(A @ y)
```

These are oddities that you will get used to, and while I initially thought that the notation for everything here was rather counter-intuitive, it actually does make sense (in some respect) why everything was implemented this way.

**Note**: in an attempt to "fix" these problems, the Numpy library also contains an `np.matrix` class (where everything is represented as a 2D matrix, with row and column vectors explicit, and the multiplication operator `*` overloaded to perform matrix multiplication).  Don't use this class.  The issue is that 1) when you want to perform _non-matrix_ operations, you need to cast them back to `np.ndarray` objects, which creates very cumbersome code; and 2) most function and external libraries return `ndarray` objects anyway.  There are, somewhat annoyingly, some collection of Numpy functions (and especially Scipy functions) that _do_ return `np.matrix` objects, and the one function you'll need to know is `np.asarray()`, which casts them back to arrays while not performing any matrix copies.  For example, the sparse matrix routines that we will see shortly have a function `.todense()` that returns dense version of the matrix but as an `np.matrix` object.


```python
import scipy.sparse as sp
A = sp.coo_matrix(np.eye(5))
A.todense()
```

```
matrix([[ 1.,  0.,  0.,  0.,  0.],
        [ 0.,  1.,  0.,  0.,  0.],
        [ 0.,  0.,  1.,  0.,  0.],
        [ 0.,  0.,  0.,  1.,  0.],
        [ 0.,  0.,  0.,  0.,  1.]])
```

You can cast it to an `ndarray` using the code.


```python
np.asarray(A.todense())
```

```
array([[ 1.,  0.,  0.,  0.,  0.],
       [ 0.,  1.,  0.,  0.,  0.],
       [ 0.,  0.,  1.,  0.,  0.],
       [ 0.,  0.,  0.,  1.,  0.],
       [ 0.,  0.,  0.,  0.,  1.]])
```

### Order of matrix multiplication operators

Let's also look at what we mentioned above, considering the order of matrix multiplications in terms of time complexity.  By default the `@` operator will be applied left-to-right, which may result is very inefficient orders for the matrix multiplication.


```python
A = np.random.randn(1000,1000)
B = np.random.randn(1000,2000)
x = np.random.randn(2000)
%timeit A @ B @ x
```

```
67.9 ms ± 3.91 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

This performs the matrix products $(AB)x$, which computes the inefficient matrix multiplication first.  If we want to compute the product in the much more efficient order $A(Bx)$, we would use the command


```python
%timeit A @ (B @ x)
```

```
1.44 ms ± 73.3 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

The later operation can be about 50x faster that the first version, and the difference only gets larger for larger matrices.  Be _very_ careful about this point when you are multiplying big matrices and vectors together.

### Inverses and linear solves

Finally, Numpy includes the routine `np.linalg.inv()` for computing the matrix inverse $A^{-1}$ and `np.linalg.solve()` for computing the matrix solve $A^{-1}b$, for $A \in \mathbb{R}^{n \times n}$, $b \in \mathbb{R}^n$.


```python
b = np.array([-13,9])
A = np.array([[4,-5], [-2,3]])

print(np.linalg.inv(A), "\n")   # explicitly form inverse
print(np.linalg.solve(A,b))     # compute solution A^{-1}b
```

```
[[ 1.5  2.5]
 [ 1.   2. ]] 

[ 3.  5.]
```

Obviously the `np.linalg.solve()` routine is also equivalent to the matrix-vector product with the inverse.


```python
print(np.linalg.inv(A) @ b)    # don't do this
```

```
[ 3.  5.]
```

However, you should _not_ do this.  In general, actually computing the inverse and then multiplying by a vector is both slower and less numerically stable than just solving the linear system.  For those who are curious (you won't need to know this for this class, but it can be useful to understand), this is because performing the solve internally actually computes a _factorization_ of the matrix called the LU factorization: it decomposes $A$ into the matrix product $A = LU$ where $L$ is a lower triangular matrix (all entries above the diagonal are zero), and $U$ is an upper triangular matrix (all entries below the diagonal are zero); if we want to be even more precise it actually computes the LU factorization on a version of $A$ with permuted rows and columns, but that is definitely beyond the scope of this course.  After factorizing $A$ in this manner, it computes the inverse

$$
A^{-1} b = (L U)^{-1} b = U^{-1} L^{-1} b
$$

In turns out that computing the product $L^{-1} b$ (again, not actually explicitly computing the inverse of $A$, just computing the solve) for a triangular matrix is very efficient, it just takes $O(n^2)$ operations instead of $O(n^3)$ operations as for a generic matrix.  This means that once we compute the factorization $A = LU$ (which does itself take O(n^3) time, as a note, which is why matrix solves are O(n^3) complexity), solving for the right hand size $b$ is "easy".

In fact the way that we compute the inverse is by first computing the LU factorization and then "solving" for the right hand side $I$.  But obviously if we just want to ultimately solve for a single right hand size $b$, this is an unnecessary step, and it introduces additional error into the computation.  For this reason, you will almost ways prefer to use `np.linalg.solve()` unless to really need elements of the inverse itself (and not just to multiply the inverse by some expression).



## References

- [Numpy library](http://www.numpy.org)
- [Linear Algebra Review](http://www.cs.cmu.edu/~zkolter/course/linalg/)
- [Numpy broadcasting rules](https://docs.scipy.org/doc/numpy-1.13.0/user/basics.broadcasting.html)
