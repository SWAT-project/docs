---
layout: default
title: Building SWAT
parent: Get Started
nav_order: 1
---
<h1> Building SWAT </h1>
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

[Tests]: /docs/pages/tests.html





[Gradle]: https://gradle.org