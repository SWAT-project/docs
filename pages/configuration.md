---
title: Configuration
layout: default
nav_order: 5
---
# Configuration

---

This page contains all configurable options.
{: .highlight }
This page is still under construction.
## VM Options:

---

Some options have to be configures using System properties that are set using (J)VM Options.

- `javaagent: String (required)` Relative path to the agent JAR. This is the core of the symbolic-executor that attaches to the SuT. When SWAT is run from within its repository, the agent should be set to/ using `-javaagent:symbolic-executor/lib/symbolic-executor.jar`
- `(D)config.path: String (optional: default=../swat.cfg)`  If a config file is written, this *relative* path should point to the file. If the config file does not exist at the specified location or the path is not set, default values are used.
- `(D)java.library.path (recommended)` The library path is used to make `z3` and its java bindings available to the solver. When running SWAT inside its repository (using gradle) the task `copyNativeLibs` can be used to automatically add the required dependencies for common systems to the `libs/java-library-path` path. This folder then needs to be set here: `-Djava.library.path=libs/java-library-path` . Alternatively the correct version of z3 can be added to directories where java looks for libraries by default. Then this option does not need to be set.
- `(D)logging.level (optional: default=INFO)` The verbosity of logging can be configured using this option. Logging is performed using [logback](https://github.com/qos-ch/logback), allowed logging levels are `[TRACE, DEBUG, INFO, WARN, ERROR]` .
- `(D)logging.path (optional: default=./logs)` The relative path to the folder where all logging related information should be stored. This includes all log-files and instrumented classes alongside additional information.

## swat.cfg:

---

### General Options

- `exitOnError` When true, errors are only logged but not thrown, should only be used carefully as swallowed errors can quickly lead to unexpected issues.

---

### Instrumentation Options

All configuration options that modify the instrumentation behaviour.

- `instrumentation.dispatcher: String (optional: default=de.uzl.its.swat.symbolic.SymbolicInstructionDispatcher` The fully qualified name of the class that initiates the instruction handling by listening for calls from the concrete execution. Currently, only one functional dispatcher is available.
- `instrumentation.prefix: String (optional=: default=SWAT)` Prefix used to avoid naming conflicts when the instrumentation adds analysis methods to classes.
- `instrumentation.includePackages: String (optional: default=[])` Colon-separated list of fully qualified class names that should be instrumented, if not set all classes are instrumented if not otherwise configured.
- `instrumentation.excludePackages: String (optional: default=[])` Colon-separated list of fully qualified class names that should not be instrumented. Currently only works when `instrumentPackages` is not set.
- `instrumentation.instructionIDs: String (optional: default=logging.debug)` When true (default) all instrumented instruction will get IIDs assigned during instrumentation. Useful for debugging (finding the concrete instruction in the instrumented .class file who’s IID appeared last in the log before an issue)
- `instrumentation.transformer: Enum (optional: default=NONE)`: Determines what instrumentation mode should be used. This heavily influences how the engine performs and is one of the core options. The possible options (`de.uzl.its.swat.instrument.TransformerType`) include:
    - `NONE`: No symbolic tracking is performed
    - `SV_COMP`: Special mode used for the software verification competition ([SV-Comp](https://sv-comp.sosy-lab.org)).
    - `PARAMETER`: Enables symbolic tracking for method parameters, default mode for SuT’s that are not web applications
    - `WEB_SERVLET`: Automatically finds controllers that adhere to the default servlet architecture and enables symbolic tracking of user-controller values. These include but are not limited to cookies, parameters and headers. A detailed description of this mode can be found here (ToDo)
   - `SPRING_ENDPOINT`: Similar to the servlet mode but tailored to springs architecture. Not all types of endpoints are supported.

- `instrumentation.parameter.symbolicPattern (optional: default="")`: Fully qualified name of the method(s) for which the parameters should be tracked symbolically. Only applies when the `instrumentation.transformer` is set to `PARAMETER`. Supports basic regular expressions (`Pattern.*matches()*`).

---

### Logging Options

All configuration options that revolve around logging and debugging.

- `logging.directory: String (optional: default=logs/)`: Relative path where logs and instrumented `.class` files are stored.

    <aside>
    ⚠️ Currently not fully working due to logback configuration.

    </aside>

- `logging.level: Enum (optional: default=INFO)`: Logging level (`org.slf4j.event.Level`) to print to console and file. Levels include `[TRACE, DEBUG, INFO, WARN, ERROR]`.
- `logging.classes: Boolen (optional: default=False)` If set to `True` , all `.class` files that are instrumented are saved in the folder `instrumented` in the `logging.directory`.
- `logging.debug: Boolean (optional: default=False)` Enables or disables the debug mode. The debug mode groups and enables several features that are useful for diagnosing errors at the cost of increased space and time consumption. Each feature can be overwritten by its own configuration option. This mode is just a simplification for debugging. The included features are:
    - Extended instruction ID’s (`instrumentation.instructionIDs`) . Assigns each instruction a unique id during instrumentation that can be found in the log and in the instrumented `.class`file.
    - Logging of out-of-scope invocations (`invocation.logging`).
- `invocation.logging: Boolean` When enabled invocations that are out-of-scope (not instrumented) are logged to the logging folder (`logging.directory`). In detail, each invocation that returns a `PlaceHolder` and is not of type `void` is logged. This is useful to determine the completeness of the analysis/ detect when information passes through uninstrumented regions. (Default: `True`)
- `loggingFormulaLength: Integer` Limits the length of symbolic formulas printed to the log, useful for very large formulas such as when arrays are in use. (Default: `100`)

---

### Explorer Options

Configuration options to specify how the symbolic executor communicates with the symbolic explorer. Only applies when a mode is selected that does not use the `LocalSolver`.

- `explorer.host: String` : The hostname/ IP of the symbolic explorer. (Default: `localhost`)
- `explorer.port: Int`: The port on which the explorer listens for requests. (Default: `8078`)
- `explorer.traceURI: String`: The URI where the explorer listens for requests with traces. (Default: `constraints/submit`)

---

### Solver Options

- `solver.mode: Enum (optional: default=SolverMode.LOCAL)`: Solver mode (`de.uzl.its.swat.solver.SolverMode`) determines how constraints should be solved. Supported modes are:
    - `LOCAL`: Each constraint will be solved for once with all corresponding path constraints and input bounds. The solution is logged to the console and the `solution.log` file. However the solution is not relayed back into another round of executions. This mode is primarily intended for testing and debugging.
    - `HTTP`: Traces are sent to the symbolic-explorer that is in turn responsible for finding new solutions and driving the execution. From the view of the symbolic-executor this mode is fire-and-forget as the symbolic engine is done after sending the trace information.

  New modes are planned, such as a local mode with iterative testing.


---

### SV-Comp:

- `svcomp.randomInputs: Boolean (optional: default=false)`: Flag to enable random input sampling for SV-Comp. If disabled, fixed values are used for each datatype.

## Misc

- Symbolic Scope
    - `symbolicValueFunction` [0:1]
        - [0] `makeSymbolicClassPath` Only 1 usage: The ParameterTransformer exits if the class file that should be instrumented is not equal to this value
        - [1]  `symbolicFunctionPattern`  Two usages; first the ParameterClassAdapter only instruments methods who’s name (signature) matches this. Secondly, the SymbolicWrapperClassAdapter only wraps methods that match this name.
    - `symbolicStartFunction` [0:1]
    - [0] `symbolicStartPath` One usage: The SymbolicWrapperTransformer exits if the classname is not equal to this
    - [1] `symbolicStartFunctionPattern` Unused