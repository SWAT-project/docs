---
layout: default
title: Building SWAT
parent: Get Started
nav_order: 1
---
<h1> Building SWAT </h1>
Getting _SWAT_ up and running requires two separate procedures for the _Executor_ (Java) and the _Explorer_ (Python).
<h3> Building the Symbolic Executor </h3>
The _Symbolic Executor_ (Javaagent) is build using [Gradle]. To build the agent run:

```
./gradlew build -x test
```
This uses the gradle wrapper to build the agent.
The `-x test` option is used to skip the tests. The tests are not skipped by default and cannot be run without the 
solver. The agent's `.jar` file is located in `symbolic-executor/build/lib/symbolic-executor.jar` directory.
Gradle requires a JDK 17 to build the agent. If no installed version is found it will be automatically fetched. 

After building the agent, the solver (currently Z3) must be installed. 
While a manual installation is possible, the easiest way is to use the provided gradle task:

```
./gradlew copyNativeLibs
```

A number of common z3 binaries are provided in the `libs` folder and this step will copy the appropriate one to the 
`libs/java-library-path` folder. Consequently, when running the agent, the `java.library.path` must be set to this folder.
Alternatively when installing Z3 manually, the `java.library.path` must be set to the folder containing the Z3 binary.

To test if the installation was successful, one of the test cases that omits communication with the _Symbolic Explorer_ 
can be used. Instead of sending the trace data to the _Symbolic Explorer_, one path is locally selected and solved for. 
This can be done by running the following command:

```
./gradlew runI2X
```

If the following message is printed, the installation was successful:

```
======== BEGIN OF OUTPUT FOR AUTOMATIC VALIDATION ========
de/uzl/its/tests/I2X/I2S(I)I :: Inputs: [I_35, -2147483648] -> Solutions: [I_35: -1]
========= END OF OUTPUT FOR AUTOMATIC VALIDATION =========
```

A detailed explanation of the entire output of this and similar test-cases is provided in the [Tests] section.
But generally this output indicates that the agent was able to solve the given path and that the solver was able to find a solution.
<h3> Building the Symbolic Explorer </h3>
As the _Symbolic Explorer_ is written in Python, it does not require a build process.
However, it does require a number of packages to be installed.
It is recommended to use a virtual environment to install these packages.
To create a virtual environment, run the following command in the folder `symbolic-explorer`:

```
python3 -m venv venv
```

After activating the virtual environment using `source venv/bin/activate`, the required packages can be installed using pip from the 
provided `requirements.txt` file:

```
pip install -r requirements.txt
```

To test if the installation was successful, one of the examples can be used to see if the communication with the _Symbolic Executor_ works.
This can be done by running the following command:

```
./gradew runBasic1
```

While, again, the output of this command is explained in the [Tests] section, a successful installation will print the following message:

```
[EXPLORER] Found 1 violations
[EXPLORER] Violation: ['STRING_0 = SuperSecretString\\\x80\x81\x82\x83#', 'INT_1 = 22']
```

This indicates that the symbolic exploration terminated successfully and that the desired violation was found.

[Tests]: /docs/pages/tests.html
[Gradle]: https://gradle.org