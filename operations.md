
# Operations in Julia and C: Sum of Vectors and Matrix Multiplication

In this guide, we'll demonstrate how to perform vector summation and matrix multiplication using two programming languages: a compiled language (C) and an interpreted language (Julia).

## Step 1: Install the necessary packages inside the AlmaLinux container

First, we need to install all the necessary packages to use both Julia and C.

### Install Julia

Start by installing the `curl` package:

```bash
dnf install -y curl --allowerasing
```

Then, use `curl` to install the latest version of Julia:

```bash
curl -fsSL https://install.julialang.org | sh -s -- -y
```

After the installation is complete, verify the installed version of Julia:

```bash
julia --version
```

Now you can start Julia by typing:

```bash
julia
```

You can enter your Julia code directly in the terminal.

## Step 2: Vector Summation in Julia

To begin with, let's explore the summation operation `a*X + Y` where `X` and `Y` are vectors of size `N` with entries all equal to `x` and `y` respectively, and `a` is a constant. The following function in Julia calculates this summation:

```julia
function sum(N, xval, yval, a, s)
    #Define the vectors x and y
    x = fill(xval, N)
    y = fill(yval, N)
    #Here we sum the vectors
    z = a * x + y
    #Initialize "check" variable
    check = 0
    for i in 1:N
        check += z[i] - s
    end
    println("Verification check (should be 0): ", round(check, digits=6))
    return z
end
```

This function takes in values `N`, `xval`, `yval`, `a`, and `s`, where `s` is the expected sum value for each component of the final sum vector. You can use this function for any `N`, `xval`, and `yval` by calling it on the terminal.

## Step 3: Matrix Multiplication in Julia

Next, let's look at how to perform matrix multiplication `A*B = C`, where `A` and `B` are `NxN` matrices with entries all equal to `a` and `b`, respectively. The following Julia function computes the matrix multiplication:

```julia
function product(N, aval, bval, s)
    a = fill(aval, N, N)
    b = fill(bval, N, N)
    z = zeros(N, N)
    
    for i in 1:N, j in 1:N
        z[i, j] = sum(a[i, k] * b[k, j] for k in 1:N)
    end
    
    check = 0
    for i in 1:N, j in 1:N
        check += (z[i, j] - N * s)
    end
    
    println("Verification check (should be 0): ", round(check, digits=6))
    return z
end
```

Similarly to the sum, the `product` function takes the arguments `a`, `b`, `N`, and the expected value `s` for the components of matrix `C`. The function computes the matrix multiplication and checks the result using the verification step.

Once you finish working with Julia, you can exit the Julia environment by typing:

```bash
exit()
```

Then, return to the container's terminal.

## Step 4: Install C Compiler and Editor

Now let's proceed with C. First, you need to install the necessary packages to compile and create C programs. Install the packages by running:

```bash
dnf install gcc
```

Additionally, install `nano` to edit your C programs:

```bash
dnf install nano
```

## Step 5: Implementing the Sum in C

Create a file for your sum program by typing:

```bash
nano sum.c
```

In the editor, write the following C code to implement the vector sum:

```c
#include <stdio.h>
#include <stdlib.h>

void compute_sum(int N, double xval, double yval, double a, double s) {
    // Dynamic allocation of vectors
    double *x = (double *)malloc(N * sizeof(double));
    double *y = (double *)malloc(N * sizeof(double));
    double *z = (double *)malloc(N * sizeof(double));

    if (x == NULL || y == NULL || z == NULL) {
        printf("Memory allocation error.\n");
        exit(1);
    }

    // Initialize vectors
    for (int i = 0; i < N; i++) {
        x[i] = xval;
        y[i] = yval;
    }

    // Compute z and check the sum
    double check = 0.0;
    for (int i = 0; i < N; i++) {
        z[i] = a * x[i] + y[i];
        check += z[i] - s;  // Sum of differences
    }

    // Print check (should be 0 if the sum is correct)
    printf("Sum verification (should be 0): %.6f\n", check);

    // Print the first 10 elements of Z for verification
    printf("First 10 elements of Z:\n");
    for (int i = 0; i < (N < 10 ? N : 10); i++) {
        printf("%.2f ", z[i]);
    }
    printf("...\n");

    // Free allocated memory
    free(x);
    free(y);
    free(z);
}

int main() {
    int N;
    double xval, yval, a, s;

    // User input
    printf("Enter the value of N: ");
    scanf("%d", &N);
    printf("Enter the value of x: ");
    scanf("%lf", &xval);
    printf("Enter the value of y: ");
    scanf("%lf", &yval);
    printf("Enter the value of a: ");
    scanf("%lf", &a);
    printf("Enter the value of s: ");
    scanf("%lf", &s);

    // Call the function
    compute_sum(N, xval, yval, a, s);

    return 0;
}
```

After writing the code, save it with `^O` and press Enter. To exit `nano`, press `^X`.

Now, compile the program with:

```bash
gcc -o sum sum.c
```

And run it with:

```bash
./sum
```

This will execute the vector summation as done in Julia.

## Step 6: Implementing Matrix Multiplication in C

Similarly, create a file for the matrix multiplication program by typing:

```bash
nano product.c
```

Then, write the following code to implement matrix multiplication:

```c
#include <stdio.h>
#include <stdlib.h>

// Function to compute matrix multiplication and verify results
void matrix_multiplication(int N, double Aval, double Bval, double expected_C) {
    // Allocate memory for matrices
    double **A = (double **)malloc(N * sizeof(double *));
    double **B = (double **)malloc(N * sizeof(double *));
    double **C = (double **)malloc(N * sizeof(double *));
    
    if (A == NULL || B == NULL || C == NULL) {
        printf("Memory allocation error.\n");
        exit(1);
    }

    for (int i = 0; i < N; i++) {
        A[i] = (double *)malloc(N * sizeof(double));
        B[i] = (double *)malloc(N * sizeof(double));
        C[i] = (double *)malloc(N * sizeof(double));
        if (A[i] == NULL || B[i] == NULL || C[i] == NULL) {
            printf("Memory allocation error.\n");
            exit(1);
        }
    }

    // Initialize matrices A and B
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            A[i][j] = Aval;
            B[i][j] = Bval;
            C[i][j] = 0.0;  // Initialize result matrix C
        }
    }

    // Matrix multiplication C = A * B
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            for (int k = 0; k < N; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }

    // Verification step
    double check = 0.0;
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            check += C[i][j] - N * expected_C;
        }
    }

    // Print verification result
    printf("Verification check (should be 0): %.6f\n", check);

    // Print first few elements of C for confirmation
    printf("First few elements of matrix C:\n");
    for (int i = 0; i < (N < 3 ? N : 3); i++) {
        for (int j = 0; j < (N < 3 ? N : 3); j++) {
            printf("%.2f ", C[i][j]);
        }
        printf("\n");
    }
    printf("...\

n");

    // Free allocated memory
    for (int i = 0; i < N; i++) {
        free(A[i]);
        free(B[i]);
        free(C[i]);
    }
    free(A);
    free(B);
    free(C);
}

int main() {
    int N;
    double Aval, Bval, expected_C;

    // User input
    printf("Enter the value of N: ");
    scanf("%d", &N);
    printf("Enter the value of Aval: ");
    scanf("%lf", &Aval);
    printf("Enter the value of Bval: ");
    scanf("%lf", &Bval);
    printf("Enter the expected value of C: ");
    scanf("%lf", &expected_C);

    // Call the function
    matrix_multiplication(N, Aval, Bval, expected_C);

    return 0;
}
```

After writing the code, save it with `^O` and press Enter. To exit `nano`, press `^X`.

Now, compile the program with:

```bash
gcc -o product product.c
```

And run it with:

```bash
./product
```

This will execute the matrix multiplication as done in Julia.

With these steps, you have successfully implemented vector summation and matrix multiplication both in Julia and C. You can test the programs on your system to ensure they work as expected.


