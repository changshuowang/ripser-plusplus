# Instructions on how to use Python Bindings of Ripser++
The purpose of the Python Bindings is to allow users to write their own Python scripts to run Ripser++. The user can write Python preprocessing code on the inputs of Ripser++. This can eliminate file I/O and allow for automated calling of Ripser++.

Author: Birkan Gokbag

Editor: Simon Zhang

Before using this interface, please read the README for Ripser++ from the ripser-plusplus/ directory as Python bindings requires Ripser++ requirements to be met.

Requirements:
    Python 3.x and
    NumPy

(As of January 2020, Python 2.x has been [sunset](https://www.python.org/doc/sunset-python-2/))


## File Organization

Skeleton directory structure of ripser-plusplus/python/:

```
python/
    └─ bin/ - Where all of your executables are located;
         └── libpyripser++.so
    |─ ripser_plusplus_python/ - Python Bindings package of Ripser++
    └─ working_directory/ - Contains examples on how to use the Python bindings, examples are located under under examples.py, and should be used as a working directory (where Python scripts are written) by the user
         |── run_ripser++_w_CLI.py - an example of using a file name to run analysis instead of creating user matrix
         |── run_ripser++_w_matrix.py - an example of creating a user matrix and sending it to Ripser++
         |── examples.py
         └── your_own_script.py
    |─ README.md - This file
    └─ setup.py - Installs the Python package
```

## How do the Python Bindings Work?

You will still need all of the dependencies of Ripser++ in order to build the necessary binaries. There is a CMakeLists.txt file in the ripser-plusplus/python/ folder that will create a Makefile that builds a shared object file: ripser-plusplus/python/bin/lipyripser++.so from ripser++.cu. lipyripser++.so is loaded through the ctypes foreign function library of Python. Ripser++ is then accessed through the ripser_plusplus_python package with an API of one function called ```run(-,-)``` to be called by your own custom Python script. The PYTHONPATH environment variable must be set to allow the ripser_plusplus_python package to be imported from your Python scripts. Otherwise, if you have enough privileges on your system, you can ```pip3 install .``` in the ripser-plusplus/python/ directory.

## Installation

In the project directory: ripser-plusplus/, run the command:

```
source install_w_python_bindings.sh
```

or:

from the ripser-plusplus directory follow these steps:

1) Create the ripser-plusplus/python/bin directory: ```mkdir -p python/bin``` if necessary
2) Under ripser-plusplus/python/bin/, run ``` cmake .. && make ``` to compile the project and make the executables
3) Set environment variable for the libpyripser++.so with name PYRIPSER_PP_BIN, like:
     ```export PYRIPSER_PP_BIN=$PWD/libpyripser++``` under python/bin/
4) Append to PYTHONPATH the location of the directory containing the ripser_plusplus_python package, like:
     ```export PYTHONPATH="${PYTHONPATH}:$PWD/.."```
5) (Optional) Under ripser-plusplus/python/, run ``` pip3 install . ``` to install the project as a Python package. If you are using python3, use pip3.
6) Navigate to ripser-plusplus/python/working_directory/ and check examples.py to see the usage for Python integration, and create your own script to run Ripser++ with Python. The arguments supplied to Python integration are identical to the ones supplied to Ripser++ executable; it supports user matrices in distance and lower-distance formats.

Important Notes:

* PYTHONPATH is set manually in the install_w_python_bindings.sh shell script. An alternative is to run ```pip3 install .``` in the ripser-plusplus/python/ directory. However, on many systems we have found that you need administrative privileges to run pip3 install and thus this is why the shell script edits the PYTHONPATH environment variable.
* Do not change the directory structure if Python package is not installed and if environment variables are not set.
* By default, the program assumes the directory for running your Python scripts is under ripser-plusplus/python/working_directory.

## The ripser_plusplus_python API

ripser_plusplus_python package API:
* Function to Access Ripser++:
    ```
        run(arguments_list, matrix or file_name)
    ```
    * First Argument:
        * arguments_list: Contains the command line options to be entered into Ripser++ as a string. e.g. ```"--format lower-distance --dim 2"```
    * Second Argument: Could be either of the following but not both
        * matrix: Must be a numpy array
            * e.g. ```[3,2,1]``` is a lower-distance matrix of 3 points
            * e.g. ```[[0,3,2],[3,0,1],[2,1,0]]``` is a distance matrix of 3 points 
        * file_name: Must be of type string.
            * e.g. ```"../../examples/sphere_3_192.distance_matrix.lower_triangular"```

## How to use Ripser++ with Python Bindings?

After having installed the Python bindings successfully (see the Installation section), first checkout the sample code in python/working_directory/ such as examples.py.

To create your own Python script to run Ripser++. Create a Python file (e.g. myExample.py) under ripser-plusplus/python/working_directory/.
At the top of your Python script:

Import the ripser_plusplus_python package to access Ripser++ computing engine:

```
import ripser_plusplus_python as rpp_py
```
Also import numpy, if you want to input a User Matrix:
```
import numpy as np
```
In your Python script, call ```run(arguments_list, matrix or file_name)``` with the following usages:

### Read from File

Python bindings work with file name inputs similar to ripser++ executable. Examples are located under ripser-plusplus/python/working_directory/examples.py.

e.g. ```rpp_py.run("--format point-cloud --sparse --dim 2 --threshold 1.4", "../../examples/o3_4096.point_cloud")```

### User Matrix Formats

**Note**: default user matrix format is distance in Ripser++. If you know your matrix format is different, then you must use the --format option

#### distance matrix:
* Only supports matrix with the following constraints:
    * Has only 0s at diagonals
    * Symmetric
    * Lower Triangular matrix adhears to the same constraints as lower-distance matrix

e.g. ```rpp_py.run("--format distance", np.array([[0,3,2],[3,0,1],[2,1,0]]))```

runs Ripser++ on a 3 point finite metric space.

#### lower-distance matrix:
* Only supports vectors, as either a row or column vector
* Must be the same size as a square matrix's linearized lower triangular matrix

e.g. ```rpp_py.run("--format lower-distance",np.array([3,2,1]))```

runs Ripser++ on the same data as the distance matrix given above.
#### point-cloud:
* Supports a 2-d numpy array where the number of rows are the number of points embedded in d-dimensional euclidan space and the number of columns is d
* Assumes the Euclidean distance between points

e.g. ```rpp_py.run("--format point-cloud",np.array([3,2,1],[1,2,3]))```

runs Ripser++ on a 2 point point cloud in 3 dimensional Euclidean space.

#### sparse (COO):
* Currently not supported, use file name

### Running Python scripts
To run your Python scripts, run, for example, ``` python3 myExample.py``` or ```python3 examples.py``` in the working_directory. This runs Ripser++ with its output sent to stderr and stdout as if it ran from the console. Python 2 is no longer supported, please use python3 when running your scripts.