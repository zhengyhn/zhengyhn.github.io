---
date: 2022-07-24
title: C语言实现的矩阵类
tags: ['c']
categories: ['c']
---


随着学习的深入，数学是必不可少的，最近复习到矩阵，于是用我最喜欢的C语言写
了一个矩阵类，使用面向对象的方法，只实现了简单的操作:新建，删除，加，减，
乘，求幂，转置等，以后会再增加。

matrix.h

```c
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

/* change all elements of matrix M to their opposite sign elements */
void matrix_minus(Matrix *m);

/* add two matrix M1 and M2, the result is matrix RESULT */
void matrix_add(const Matrix *m1, const Matrix *m2, Matrix *result);

/* substract matrix SUBSTRAHEND from matrix MINUEND, the result is
 * matrix RESULT */
void matrix_substract(Matrix *minuend, Matrix *substrahend, Matrix *result);

/* multiple matrix LEFT and RIGHT, the result is matrix RESULT.
 * with a stupid algorithm. */
void matrix_multiple(Matrix *left, Matrix *right, Matrix *result);

/* calculate the matrix's power of exponent EXP, the result is
 * the matrix RESULT. */   
void matrix_power(Matrix *m, int exp, Matrix *result);

/* calculate the transposition of matrix M, the result is TRANS */
void matrix_trans(Matrix *m, Matrix *trans);

/* print the matrix M */
void matrix_print(const Matrix *m);

#endif /* _MATRIX_H_ */
```

matrix.c

```c
#include "matrix.h"

static void assert_matrix(const Matrix *);
static int can_add(const Matrix *m1, const Matrix *m2, const Matrix *result);
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

void matrix_minus(Matrix *m)
{
    int i;

    assert_matrix(m);

    for (i = 0; i < m->row_no * m->col_no; i++) {
        *(m->elem + i) = -(*(m->elem + i));
    }
}

void matrix_print(const Matrix *m)
{
    int i, j;

    assert_matrix(m);

    for (i = 0; i < m->row_no; i++) {
        for (j = 0; j < m->col_no; j++) {
            printf("%5g", *(m->elem + i * m->col_no + j));
        }
        printf("\n");
    }
}

void matrix_add(const Matrix *m1, const Matrix *m2,
                Matrix *result)
{
    int i;

    assert_matrix(m1);
    assert_matrix(m2);
    assert_matrix(result);

    if (!can_add(m1, m2, result)) {
        fprintf(stderr, "add: row number or column number not match!");
        exit(1);
    }
    for (i = 0; i < result->row_no * result->col_no; i++) {
        *(result->elem + i) = *(m1->elem + i) + *(m2->elem + i);
    }
}

void matrix_substract(Matrix *minuend, Matrix *substrahend,
                      Matrix *result)
{
    matrix_minus(substrahend);
    matrix_add(minuend, substrahend, result);
    matrix_minus(substrahend);
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

void matrix_trans(Matrix *m, Matrix *trans)
{
    int i, j;

    assert_matrix(m);
    assert_matrix(trans);

    if (!(m->row_no == trans->col_no &&
          m->col_no == trans->row_no)) {
        fprintf(stderr, "row number or column number not match!");
        exit(1);
    }
    for (i = 0; i < m->row_no; i++) {
        for (j = 0; j < m->col_no; j++) {
            *(trans->elem + j * trans->col_no + i) =
                *(m->elem + i * m->row_no + j);
        }
    }
}

static void assert_matrix(const Matrix *m)
{
    if (m == NULL) {
        fprintf(stderr, "the matrix is NULL!");
    }
}

static int can_add(const Matrix *m1, const Matrix *m2,
                   const Matrix *result)
{
    if (m1->row_no == m2->row_no &&
        m1->col_no == m2->col_no &&
        m1->row_no == result->row_no &&
        m1->col_no == result->col_no) {
        return 1;
    } else {
        return 0;
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

main.c

```c
#include "matrix.h"

#define M 5
#define N 5

/* return the fibonacci number of NUM */
float fibonacci(int num);

int main(int argc, char *argv[])
{
    Matrix *m1, *m2, *result;
    FILE *fp;

    fp = fopen("input.txt", "r");
    if (fp == NULL) {
        perror("open input.txt");
        exit(1);
    }
    m1 = matrix_from_file(fp, 5, 5);
    m2 = matrix_from_file(fp, 5, 5);
    result = matrix_from_file(NULL, 5, 5);

    matrix_print(m1);
    printf("\n");

    matrix_minus(m1);
    matrix_print(m1);
    printf("\n");

    matrix_add(m1, m2, result);
    matrix_print(result);
    printf("\n");

    matrix_substract(m2, m1, result);
    matrix_print(result);
    printf("\n");

    matrix_multiple(m1, m2, result);
    matrix_print(result);
    printf("\n");

    matrix_trans(m1, result);
    matrix_print(result);
    printf("\n");

    matrix_delete(m1);
    matrix_delete(m2);
    fclose(fp);
    matrix_delete(result);

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

input.txt

``` {.example}
1 2 3 4 5
6 7 8 9 10
9 8 7 6 5
4 3 2 1 -1
-2 0 1 2 3

1 1 1 1 1
2 2 2 2 2
3 3 3 3 3
4 4 4 4 4
5 5 5 5 5

```

Makefile

``` {.example}
#for matrix

CC = gcc
CFLAGS = -Wall -o2 -g
HEADER = matrix.h
SRC = matrix.c main.c
OBJ = $(SRC:.c=.o)

main: $(OBJ)
    $(CC) $(CFLAGS) $^ -o $@

.PHONY: clean

clean:
    rm -rf *.o
```

能看到这里的，已经不是普通人了，希望多多指教！
