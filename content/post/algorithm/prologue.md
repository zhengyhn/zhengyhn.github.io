---
date: 2022-07-18
title: 开幕
tags: ['Algorithm']
categories: ['Algorithm']
---

本文为"Algorithms"一书的笔记

# fibonacci

书上给出了另外一种我想不到的解法：

``` {.c}
#include <stdio.h>

#define MAXN 100

int fib[MAXN];

int get_fib(int num);

int main(int argc, char *argv[])
{
    printf("%d\n", get_fib(10));

    return 0;
}

int get_fib(int num)
{
    int i;

    fib[0] = 0;
    fib[1] = 1;

    for (i = 2; i <= num; i++) {
        fib[i] = fib[i - 1] + fib[i - 2];
    }

    return fib[num];
}
```

我以前写过递归的，记忆性递归的，迭代的，但是这种以循环代替递归的还没写过，
也没见过，虽然理解起来非常简单，但是为什么我就想不到呢？
不过这个程序，如果要求多个数的fibonacci值，还要在循环里面判断一下：

``` {.c}
if (fib[i] != 0) {
    fib[i] = fib[i - 1] + fib[i - 2];
}
```

这样可以避免每次计算，以空间换取时间。

# Big-oh notation

每一本算法书都会讲这个东西，离散数学也讲这东西，基本讲一次忘一次，这次算是
回忆，却让我理解更透彻了。 Ο(n)，英文读作：big oh of n 定义如下：

------------------------------------------------------------------------

Let f(n) and g(n) be functions from positive integers to positive reals.

We say f = Ο(g) if there is a constant c &gt; 0 such that f(n) &lt;= c
\* g(n).

------------------------------------------------------------------------

f = Ο(g)表明f增长得没有g快，是f &lt;= g的一个简单模拟。 比如：

------------------------------------------------------------------------

f1(n) = n \^ 2, f2(n) = 2 \* n + 20,

f2 = Ο(f1), 因为：

f2(n) / f1(n) = (2 \* n + 20) / n \^ 2 &lt;= 22

可以取1到22之间的任意一个常数。

而f1 != Ο(f2)，因为：

f1(n) / f2(n) = n \^ 2/ (2 \* n + 20) 趋于无穷

------------------------------------------------------------------------

有关Ο 的一些常识：

1.  常数相乘可以忽略，如：14 \* n \^ 2 --&gt; n \^ 2
2.  如果有n \^ a和n \^ b，且a &gt; b，忽略n \^ b，如：n \^ 2 + n --&gt;
    n \^ 2
3.  如果有指数和多项式，则忽略多项式，如：n \^ 3 + n \^ 5 --&gt; n \^ 3
4.  如果有多项式和对数，则忽略对数，如：n + (logn) \^ 3 --&gt; n，再如：

n \^ 2 + nlogn --&gt; n \^ 2 作为扩展，有如下定义：

------------------------------------------------------------------------

如果f = Ο(g)，则g = *Ω*(f)

如果f = Ο(g), 且f = *Ω*(g)，则f = *Θ*(g)

------------------------------------------------------------------------

# more fibonacci algorithms

第一种方法： 用矩阵来求

我们知道：

``` {.example}
0 * bullet F0 + F1 = F1
F0 + F1 = F2
```

根据线性方程组转矩阵的方法，可以变成：

$$
\begin{equation}
\left(
\begin{array}{cc}
0 & 1 \\
1 & 1
\end{array}
\right)
\bullet
\left(
\begin{array}{c}
F0 \\
F1
\end{array}
\right)
=
\left(
\begin{array}{c}
F1 \\
F2
\end{array}
\right)
\end{equation}
$$

通过数学归纳法，可以推出：
$$
\begin{equation}
\left(
\begin{array}{cc}
0 & 1 \\
1 & 1
\end{array}
\right)^{n}
\bullet
\left(
\begin{array}{c}
F0 \\
F1
\end{array}
\right)
=
\left(
\begin{array}{c}
F_{n} \\
F_{n+1}
\end{array}
\right)
\end{equation}
$$
于是，可以写代码来实现这个算法了。

main.c

``` {.c}
#include "matrix.h"

/* return the fibonacci number of NUM */
float fibonacci(int num);

int main(int argc, char *argv[])
{
    printf("%g\n", fibonacci(10));

    return 0;
}

float fibonacci(int num)
{
    Matrix *m1, *m2, *exp;
    float **arr, **vector, result;
    int i, j;

    arr = (float **)malloc(2 * sizeof(float *));
    for (i = 0; i < 2; i++) {
        arr[i] = (float *)malloc(2 * sizeof(float));
        for (j = 0; j < 2; j++) {
            arr[i][j] = 1;
        }
    }
    arr[0][0] = 0;

    vector = (float **)malloc(2 * sizeof(float *));
    for (i = 0; i < 2; i++) {
        vector[i] = (float *)malloc(sizeof(float));
    }
    vector[0][0] = 0;
    vector[1][0] = 1;

    m1 = matrix_from_array(arr, 2, 2);
    m2 = matrix_from_array(vector, 2, 1);
    exp = matrix_from_file(NULL, 2, 2);

    matrix_power(m1, num, exp);
    matrix_multiple(exp, m2, m2);
    result = *(m2->elem);

    matrix_delete(m1);
    matrix_delete(m2);
    matrix_delete(exp);
    for (i = 0; i < 2; i++) {
        free(arr[i]);
    }
    free(arr);
    for (i = 0; i < 2; i++) {
        free(vector[i]);
    }
    free(vector);

    return result;
}
```

matrix.h

``` {.c}
#ifndef _MATRIX_H_
#define _MATRIX_H_

#include <stdio.h>
#include <stdlib.h>

//matrix element's type
typedef double matrix_t;
typedef struct matrix Matrix;

struct matrix {
    int row_no;       //row number
    int col_no;       //column number
    matrix_t *elem;   //point to the elements
};

/* create a matrix from a file pointed by FP, with
 * ROW_NO rows and COL_NO columns, return the pointer
 * to the matrix.
 * [note]:if FP is NULL, return an empty matrix */
Matrix *
matrix_from_file(FILE *fp, int row_no, int col_no);

/* create a matrix from an two dimension pointer ARR with
 * ROW_NO rows and COL_NO columns, return the pointer
 * to the matrix.
 * [note]:if ARR is NULL, return an empty matrix */
Matrix *
matrix_from_array(float **arr, int row_no, int col_no);

/* free all memory */
void matrix_delete(Matrix *m);

/* initial the matrix M as identity matrix */
void matrix_identity(Matrix *m);

/* multiple matrix LEFT and RIGHT, the result is matrix RESULT.
 * with a stupid algorithm. */
void matrix_multiple(Matrix *left, Matrix *right, Matrix *result);

/* calculate the matrix's power of exponent EXP, the result is
 * the matrix RESULT. */   
void matrix_power(Matrix *m, int exp, Matrix *result);

#endif /* _MATRIX_H_ */
```

matrix.c

``` {.c}
#include "matrix.h"

static void assert_matrix(const Matrix *);
static int can_mul(const Matrix *left, const Matrix *right,
                   const Matrix *result);
static int is_square(const Matrix *m);

Matrix *
matrix_from_file(FILE *fp, int row_no, int col_no)
{
    int i;
    Matrix *m = (Matrix *)malloc(sizeof(Matrix));

    if (m == NULL) {
        fprintf(stderr, "memory no enough!");
        exit(1);
    }
    m->row_no = row_no;
    m->col_no = col_no;
    m->elem = (matrix_t *)malloc(row_no * col_no * sizeof(matrix_t));
    if (m->elem == NULL) {
        fprintf(stderr, "memory no enough!");
        exit(1);
    }
    if (fp != NULL) {
        for (i = 0; i < row_no * col_no; i++) {
            fscanf(fp, "%lf", (m->elem + i));
        }
    }

    return m;
}

Matrix *
matrix_from_array(float **arr, int row_no, int col_no)
{
    int i, j;
    Matrix *m = (Matrix *)malloc(sizeof(Matrix));

    if (m == NULL) {
        fprintf(stderr, "memory no enough!");
        exit(1);
    }
    m->row_no = row_no;
    m->col_no = col_no;
    m->elem = (matrix_t *)malloc(row_no * col_no * sizeof(matrix_t));
    if (m->elem == NULL) {
        fprintf(stderr, "memory no enough!");
        exit(1);
    }
    if (arr != NULL) {
        for (i = 0; i < row_no; i++) {
            for (j = 0; j < col_no; j++) {
                *(m->elem + i * col_no + j) = arr[i][j];
            }
        }
    }

    return m;
}

void matrix_delete(Matrix *m)
{
    assert_matrix(m);

    if (m->elem) {
        free(m->elem);
    }
    free(m);
}

void matrix_identity(Matrix *m)
{
    int i;

    assert_matrix(m);

    if (!is_square(m)) {
        fprintf(stderr, "identity: the matrix is not square!");
        exit(1);
    }
    for (i = 0; i < m->row_no * m->col_no; i++) {
        if (((i / m->col_no) * m->row_no) + (i / m->col_no) == i) {
            *(m->elem + i) = 1;
        } else {
            *(m->elem + i) = 0;
        }
    }
}

void matrix_multiple(Matrix *left, Matrix *right, Matrix *result)
{
    int i, j, k, left_elem, right_elem;
    matrix_t sum;
    Matrix *temp;

    assert_matrix(left);
    assert_matrix(right);
    assert_matrix(result);

    if (!can_mul(left, right, result)) {
        fprintf(stderr, "multiple: row number or column number not match!");
        exit(1);
    }
    if (result == left || result == right) {
        temp = matrix_from_file(NULL, result->row_no, result->col_no);
        for (i = 0; i < result->row_no * result->col_no; i++) {
            *(temp->elem + i) = *(result->elem + i);
        }
    } else {
        temp = result;
    }
    for (i = 0; i < left->row_no; i++) {
        for (j = 0; j < right->col_no; j++) {
            sum = 0.0;
            for (k = 0; k < left->col_no; k++) {
                left_elem = *(left->elem + i * left->col_no + k);
                right_elem = *(right->elem + j * right->col_no + k);
                if ((left_elem != 0) && (right_elem != 0)) {
                    sum += left_elem * right_elem;
                }
            }
            *(temp->elem + i * temp->col_no + j) = sum;
        }
    }
    if (result == left || result == right) {
        for (i = 0; i < result->row_no * result->col_no; i++) {
            *(result->elem + i) = *(temp->elem + i);
        }
    }
}

void matrix_power(Matrix *m, int exp, Matrix *result)
{
    matrix_identity(result);

    while (exp > 0) {
        if ((exp & 1) == 1) {
            matrix_multiple(m, result, result);
        }
        matrix_multiple(m, m, m);
        exp >>= 1;
    }
}

static void assert_matrix(const Matrix *m)
{
    if (m == NULL) {
        fprintf(stderr, "the matrix is NULL!");
    }
}

static int can_mul(const Matrix *left, const Matrix *right,
                   const Matrix *result)
{
    if (left->col_no == right->row_no &&
        right->row_no == result->row_no &&
        right->col_no == result->col_no) {
        return 1;
    } else {
        return 0;
    }
}

static int is_square(const Matrix *m)
{
    return m->row_no == m->col_no;
}
```

完整的矩阵类，请参见[这里](http://www.zhengyuanhang.com/article/207.html)

第二种方法：称为fast
doubling，引用自[这里](http://nayuki.eigenstate.org/page/fast-fibonacci-algorithms)
。

从上面的矩阵法中，可以推出下面这个等式：
$$
\begin{equation}
\left(
\begin{array}{cc}
0 & 1 \\
1 & 1 
\end{array}
\right)^n
=
\left(
\begin{array}{cc}
F(n-1) & F(n) \\
F(n) & F(n+1)
\end{array}
\right)^n
\end{equation}
$$

这个等式是可以通过数学归纳法证明的，也比较容易记忆（虽然我很不喜欢记公式，
但是有的东西，能记住是非常好的）。

于是，我们可以根据这个等式推出fast doubling的算法：
$$
\begin{equation}
\begin{split}
\left(
\begin{array}{cc}
F(2n-1) & F(2n) \\
F(2n) & F(2n+1)
\end{array}
\right)
=
\left(
\begin{array}{cc}
0 & 1 \\
1 & 1
\end{array}
\right)^{2n}
=
\left(
\left(
\begin{array}{cc}
0 & 1 \\
1 & 1
\end{array}
\right)^n
\right)^2
=
\left(
\begin{array}{cc}
F(n-1) & F(n) \\
F(n) & F(n+1)
\end{array}
\right)^2
\\
= 
\left(
\begin{array}{cc}
F(n-1)^2 + F(n)^2 & F(n-1)F(n) + F(n)F(n+1) \\
F(n)F(n-1) + F(n+1)F(n) & F(n)^2 + F(n+1)^2
\end{array}
\right)
\end{split}
\end{equation}
$$
于是有：

F(2n+1) = F(n)^2^ + F(n+1)^2^

F(2n) = F(n-1)F(n) + F(n)F(n+1) = F(n)( F(n-1) + F(n+1) ) = F(n)( F(n+1)
- F(n) + F(n+1) ) = F(n)( 2F(n+1) - F(n) )

所以，如果知道F(k+1)和F(k)，可以知道：

F(2k+1) = F(k)^2^ + F(k+1)^2^

F(2k) = F(k)( 2F(k+1) - F(k))

这样就得到了通用的方法。

有了算法，实现不了也是没用，马上敲代码吧！

``` {.c}
#include <stdio.h>

/* calculate the fibonacci number of n with fast doubling algorithm. */
int fibonacci(int n);

int main(int argc, char *argv[])
{
    int i;

    for (i = 0; i < 20; i++) {
        printf("%d\n", fibonacci(i));
    }

    return 0;
}

int fibonacci(int n)
{
    int fn, fn1;

    if (n == 0) {
        return 0;
    }
    if (n == 1 || n == 2) {
        return 1;
    }
    if (n & 1) {
        fn = fibonacci((n - 1) / 2);
        fn1 = fibonacci((n - 1) / 2 + 1);
        return fn * fn + fn1 * fn1;
    } else {
        fn = fibonacci(n / 2);
        fn1 = fibonacci(n / 2 + 1);
        return fn * (2 * fn1 - fn);
    }
}
```
