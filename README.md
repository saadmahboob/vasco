VASCO
=====

VASCO is a framework for performing precise inter-procedural data flow analysis using VAlue Sensitive COntexts.

The framework classes are present in the package `vasco` and are described in the paper: [Interprocedural Data Flow Analysis in Soot using Value Contexts](http://dl.acm.org/citation.cfm?doid=2487568.2487569).

You can use these classes directly with any program analysis toolkit or intermediate representation, although they work best with [Soot](http://www.sable.mcgill.ca/soot) and it's *Jimple* representation.

## API Documentation ##

There is a JavaDoc generated [API documentation](http://rohanpadhye.github.io/vasco/apidocs) available for the framework classes.

## Simple Examples ##

The package [`vasco.soot.examples`](https://github.com/rohanpadhye/vasco/tree/master/src/vasco/soot/examples) contains some example analyses implemented for Soot such as **copy constant propagation** and a simple **sign analysis** (the latter is the same as the example used in the research paper).

## Points-to Analysis ##

The package [`vasco.callgraph`](https://github.com/rohanpadhye/vasco/tree/master/src/vasco/callgraph) contains a sophisticated points-to analysis that builds context-sensitive call graphs on-the-fly, as detailed in the paper.

### Experiments ###

To run the experiments described in the paper, ensure that Soot is in your class path and execute:

<code>
java vasco.callgraph.CallGraphTest [-cp CLASSPATH] [-out DIR] [-k DEPTH] MAIN_CLASS
</code>

Where:

- `CLASSPATH` is used to locate application classes of the program to analyze (default: `.`)
- `DIR` is the output directory where results will be dumped (default: `.`)
- `DEPTH` is the maximum depth of call chains to count (default: `10`)
- `MAIN_CLASS` is the entry point to the program

This will generate a bunch of CSV files in the output directory containing statistics of analyzed methods, contexts, and call chains.

### Using Generated Call Graphs ###

To use the generated call graphs programatically in your own Soot-based analysis, make sure to add the `vasco.callgraph.CallGraphTransformer` to Soot's analysis `PackManager` (see code in [`CallGraphTest.main()`](https://github.com/rohanpadhye/vasco/blob/master/src/vasco/callgraph/CallGraphTest.java) for how to do this) which set's the call-graphs in the global `Scene` appropriately.

Any Soot-based analysis that you add to the pipe-line after the `CallGraphTransformer` can make use of the generated call-graphs by calling `Scene.v().getCallGraph()` or `Scene.v().getContextSensitiveCallGraph()` as you would with the results of SPARK and PADDLE respectively.


## Limitations when using with Java/Soot/Jimple ##

The VASCO framework only implements the value-context-based inter-procedural analysis algorithm, and is not customized for Java in particular. This leads to several open issues that must be addressed by any application that uses VASCO for analyzing Java programs by integrating with Soot.

- Native methods are `null` in the `DefaultJimpleRepresentation`. The handling of native methods probably depends on the client analysis, and thus you may have to write your own program representation with appropriate method stubs built in to the control-flow graph.
- The JRE is not modelled. For example, `System.in` and `System.out` have initial values of `null` in the source code of `java.lang.System`. The JVM injects values for standard input/output streams at runtime. Other such JVM behaviours are not explicitly modeled. You may have to create a wrapper around the entry point to mock any JVM behavior that is relevant to your application.
- Multi-threadeding is not handled within the framework itself. Some choose to solve this by simply linking calls to `Thread.start()` with the approriate `Runnable.run()` method, though the resulting analysis may not be sound if it does not consider interleavings of program execution. You can still use VASCO with multi-threaded programs if your analysis/application can make-do with making worst-case assumptions about shared data.
- Reflection and/or dynamic class loading is not modeled by the framework. If your candidate program has this, you probably need to use some tool such as [TamiFlex](https://github.com/secure-software-engineering/tamiflex) to resolve dynamic bindings and transform your code to use static bindings before creating a `ProgramRepresentation`.

## Contributing ##

If something seems fishy, please open an issue. If you know how to fix it, please submit a pull request.
