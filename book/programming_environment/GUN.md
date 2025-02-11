# GUN

- Author: *{{Fu}}*
- Update: *Dec 14, 2022*
- Reading: *30 min*

---


## Install

There are 2 options in Macbook for parallel computing:
- **`clang` compiler + `openmp` + `openmpi`**, clang don't have `gfortran` compiler.
- **`gcc` compiler + `openmp` + `openmpi`**, gcc has `gfortran` compiler.

### `clang` compiler + `openmp`

The clang compiler doesn't bring **`openmp`** package, and install openmp for clang compiler:

```bash
# Install
brew install openmp

# For compilers to find libomp you may need to set for Makefile
export LDFLAGS="-L/opt/homebrew/opt/libomp/lib"
export CPPFLAGS="-I/opt/homebrew/opt/libomp/include"

# Or compile directly (eg: c++ script)
clang++ openmp.cpp -o openmp -Xpreprocessor -fopenmp -lomp -I/opt/homebrew/opt/libomp/include  -L/opt/homebrew/opt/libomp/lib 
```


:::{dropdown} The example **`openmp.cpp`** file is here:
:color: info
:icon: info
```c++
#include <iostream>  
#include <omp.h>  

using namespace std;  
  
int main(){  
    omp_set_num_threads(15);
    #pragma omp parallel  
    {  
        cout << "Hello World!\n";  
    }  
    return 0;  
}  
```
:::


:::{dropdown} Compile error?
:color: info
:icon: info
The first time I compile the `openmp` code, I got an error as following:
```bash
Undefined symbols for architecture arm64:
  "___kmpc_fork_call", referenced from:
      _main in main-4f8dcc.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

Then run the following command, it worked well now:
```bash
ln -s /opt/homebrew/opt/libomp/lib/libomp.dylib /usr/local/lib/libomp.dylib
```

Refer to https://stackoverflow.com/questions/71061894/how-to-install-openmp-on-mac-m1 
:::




### `gcc` compiler + `openmp` 

After installing Command Line Tools for Xcode, M1 chip use clang to compile **`.c file`** and clang++ to compile **`.cpp file`**. 
And at this moment, M1 chip also has gcc and g++, but they are the **`soft links`** to clang compiler.

To install `gcc` in M1 chip (it will also install `gfortran`), which will bring `openmp` package which is not needed to install by yourself.

```bash
brew search gcc
brew install gcc

# To compile code directly (different from clang compiler)
g++ openmp.cpp -o openmp -fopenmp
```

Set the environment variables as following, and you can also download different versions `gcc`:

```bash
open ~/.zhsrc

# >>> brew's gcc (GNU) >>>
alias gcc='gcc-12'
alias g++='g++-12'
# <<< brew's gcc (GNU) <<<
```

Refer to https://trinhminhchien.com/install-gcc-g-on-macos-monterey-apple-m1/




### Install `openmpi`

Then install **`openmpi`**, brew will use default compiler (In my M1 chip is `clang`) to compile openmpi package.
But we can use a custom compiler(clang or gcc) when compiling code.


```bash
brew install open-mpi
```

This will install a lot of command we can use, such as 
`mpic++` `mpicxx` `mpif77` `mpifort` `mpirun` `mpicc` `mpiexec` `mpif90` and `mpioutil`.
Take `mpic++` for example:

```bash
# Check openmpi version
mpic++ --version

# Check the detail of mpic++ command
mpic++ -showme

# Compile code using mpicc
mpic++ openmpi.cpp -o openmpi

# Run code
mpirun -np 4 ./openmpi
```

:::{dropdown} The example **`openmpi.cpp`** file is here:
:color: info
:icon: info
```c++
#include <iostream>  
#include <mpi.h>

using namespace std;  
int main(int argc, char **argv) {
    int ierr, num_procs, my_id;

    /* find out MY process ID, and how many processes were started. */
    ierr = MPI_Init(&argc, &argv);
    ierr = MPI_Comm_rank(MPI_COMM_WORLD, &my_id);
    ierr = MPI_Comm_size(MPI_COMM_WORLD, &num_procs);

    cout<< "Hello world! I'm process "<< my_id << " out of " << num_procs << " processes" << endl;

    ierr = MPI_Finalize();
    return 0;
}
```
:::

Set our custom compiler (clang/gcc) to compile **`openmpi.cpp`** file.

```bash
# Set openmpi's compiler before using mpi to compile code
# [1] C++ language
export OMPI_CXX=clang++
export OMPI_CXX=g++-12
# [2] C language
export OMPI_CC=clang
export OMPI_CXX=gcc-12

# Then use mpi to compile [c/c++] code
mpic++ openmpi.cpp -o openmpi
mpicc xx.c -o xx

# Run code
mpirun -np 4 openmpi
```

:::{dropdown} Do you know `mpic++` is a compiler's wrapper?
:color: info
:icon: info
Actually mpic++ is a wrapper over the system compiler. In my case, mpicc will use the clang compiler. To find what is behind mpicc you can execute the following command:

```bash
mpic++ -showme
```

Here is my output:
```bash
clang++ -I/opt/homebrew/Cellar/open-mpi/4.1.4_2/include 
-L/opt/homebrew/Cellar/open-mpi/4.1.4_2/lib 
-L/opt/homebrew/opt/libevent/lib 
-lmpi
```

:::






## Configuration


Download the plugin, `C/C++`, in VSCode Extensions

- Manual refer to [`Miscrosoft docs`](https://code.visualstudio.com/docs/python/python-tutorial)




## Package

<style>
table th:first-of-type {
    width: 30%;
}
table th:nth-of-type(2) {
    width: 50%;
}
table th:nth-of-type(2) {
    width: 20%;
}
</style>


**Scientific computation-related package**:

|    Name       |    Purpose    |    Way       |     
| ------------  | ------------- | :----------: |
| `fftw3`   | ...       | `brew` and https://www.fftw.org/     |
| ``   | ...       |      |

**Seismology package**:

|     Name     |    Purpose    |     Way       |     
| ------------ | ------------- | :-----------: |
| ``   |        |      |
| ``   |        |      |




### FFTW3

[FFTW](http://fftw.org/), the Fastest Fourier Transform in the West, is a C subroutine library for computing the discrete Fourier transform (DFT).


#### 1.Install via brew

```bash
# install
brew install fftw

# compile
g++ fftw.cpp -o fftw -lfftw3 -lm -I/opt/homebrew/opt/fftw/include -L/opt/homebrew/opt/fftw/lib

# run
./fftw
```
If the installation was successful, you should be able to use the FFTW library in your C/C++ programs by including the `fftw3.h` header file and linking against the `libfftw3.a` library. You may need to specify the path to the `fftw3.h` header file and the `libfftw3.a` library using the `-I` and `-L` options when compiling your code.

- The `-lfftw3` option tell the compiler to link the FFTW library.
- The `-lm` option tell the compiler to the math library.
- The `-I` option to include header file
- The `-L` option to include package

Refer to https://www.runoob.com/w3cnote/gcc-parameter-detail.html

:::{dropdown} The example **`fftw.cpp`** file is here:
:color: info
:icon: info
```c++
#include <fftw3.h>
#include <iostream>

int main() {
  // Set up the input signal and allocate space for the output
  const int N = 8;
  double input[N] = {1, 2, 3, 4, 5, 6, 7, 8};
  fftw_complex output[N];

  // Set up the FFTW plan
  fftw_plan plan = fftw_plan_dft_r2c_1d(N, input, output, FFTW_ESTIMATE);

  // Execute the plan
  fftw_execute(plan);

  // Print the result
  std::cout << "Output: " << std::endl;
  for (int i = 0; i < N; i++) {
    std::cout << output[i][0] << " + " << output[i][1] << "i" << std::endl;
  }

  // Clean up
  fftw_destroy_plan(plan);

  return 0;
}
```
:::



#### 2.Install from source

**Download FFTW Package:**

```bash
FFTW_VERSION=3.3.10
FFTW_FOLDER=fftw-$(FFTW_VERSION)
wget http://fftw.org/fftw-$(FFTW_VERSION).tar.gz
tar -xvf fftw-$(FFTW_VERSION).tar.gz
rm fftw-$(FFTW_VERSION).tar.gz
```


**Install:**

:::::{tab-set}
::::{tab-item} M1
```bash
cd $(FFTW_FOLDER)

./configure CC=gcc --prefix=/Absolute/path \
    --with-pic \
    --enable-single \
    --enable-shared \
    --build=x86_64-apple-darwin20

make -j 4 
make install
```
::::


::::{tab-item} Linux
```bash
cd $(FFTW_FOLDER)

./configure CC=gcc --prefix=/Absolute/path \
    --with-pic \
    --enable-single 
    --enable-shared  \
    --enable-avx2 --enable-avx \
    --enable-sse --enable-sse2 \

make -j 4 
make install
```
::::
:::::



:::{dropdown} explaination of `configure` parameters:
:color: info
:icon: info
- The `--build` option to specify the code installed in an `M1-based Mac`.

- M1 does not support `AVX` instructions, check if your machine supports SSE:
    ```bash
    gcc -mavx -E -v - </dev/null 2>&1 | grep cc1
    ```
    If your machine supports `AVX`, you should see the `-mavx` flag in the output.

- M1 does not support `SSE` instructions, check:
    ```bash
    gcc -msse -E -v - </dev/null 2>&1 | grep cc1
    ```
    If your machine supports `SSE`, you should see the `-msse` flag in the output.

- `--enable-shared` to create shared, rather than static, libraries.

- The `--with-pic` option tells the `./configure` script to build FFTW with position-independent code (PIC). `PIC` is a form of machine code that can be relocated at runtime, and is typically used when building shared libraries. Building FFTW with PIC can allow the library to be used in shared libraries and plugins, which can be loaded and unloaded dynamically at runtime.

- The `--enable-single` option tells the `./configure` script to enable single precision (float) support in FFTW. This allows FFTW to compute discrete Fourier transforms (DFTs) of single precision floating-point arrays, in addition to the default double precision (double) support.
By default, FFTW is built with double precision support only, so you will need to use the `--enable-single` option if you want to use single precision arrays with FFTW.

More information can be found in https://www.fftw.org/fftw3_doc/Installation-on-Unix.html
:::


**Compile:**

```bash
g++ fftw.pp -o fftw -lfftw3 -lm -I/Absolute/path/include -L/Absolute/path/lib
```








## Resource


- https://betterprogramming.pub/integrating-open-mpi-with-clion-on-apple-m1-76b7815c27f2
- https://www.open-mpi.org/faq/?category=mpi-apps#override-wrappers-after-v1.0
- https://unix.stackexchange.com/questions/346263/mpicc-with-different-versions-of-gcc
- [C 语言教程](https://wangdoc.com/clang/)
- [C 语言教程](https://www.runoob.com/cprogramming/c-tutorial.html)（较全面、系统）
- [X 分钟速成 C](https://learnxinyminutes.com/docs/zh-cn/c-cn/)（简要）
- [Building programs](https://fortran-lang.org/learn/building_programs)（简要）
- [跟我一起写 Makefile](https://seisman.github.io/how-to-write-makefile/)（较全面、系统）
- [X 分钟速成 make](https://learnxinyminutes.com/docs/zh-cn/make-cn/)（简要）