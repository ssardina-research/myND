# myND FOND Planner

An improved codebase for the [myND](https://github.com/robertmattmueller/myND) non-deterministic planner (2010).

## Build

The system requires:

* Java JDK 8:
  * [junit 4](https://junit.org/junit4/). Obtained via Maven but also available under `lib/` (for manual compile).
* [Maven](https://maven.apache.org/) for building the system (and further development, if needed).
* Python 3.6+ for [translator-fond](/translator-fond/) and [translator-pond](/translator-pond/) tools to translate FOND/POND PDDL files to SAS.

> [!NOTE]
> Library [args4j](https://args4j.kohsuke.org/) is used but not via a Maven dependency. Instead its source code is copied and extended to have groups of options and hidden options.

Get the dependencies in Linux as follows:

```shell
$ sudo apt-get install openjdk-8-jdk maven
```

To build it, first set-up `JAVA_HOME` to Java 8:

```shell
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/ 
$ mvn package -Dmaven.test.skip=true  # build!
```

The above will produce JAR file `target/mynd-1.0-SNAPSHOT.jar` and all executable classes in `target/classes`.

> [!WARNING]
> Maven will **not** compile the system with Java 17+ (Open JDK). Still it seem to *run* with Java 17+ once compiled (last test run with Java 21.0.6)

## Run it

The easiest way to use the planner is via its entry shell script `mynd`:

```shell
$ ./mynd -h 
```

The script already includes:

* Java options `-Xmx4g` to set the maximum heap space to 4GB. We could also add `-ea` to enable assertions.
* Usage of new `-translatePath` option to indicate where the Python translators are located. This allows the planner to be called from any directory, not just the root of the planner.

The planner can take SAS encodings or original PDDL domain and problem files. For example, to run over a SAS file and output the solution to file `plan`:

```shell
$ ./mynd data/benchmarks-fond/blocksworld_p10.sas -exportPlan plan
Base dir for myND: /home/ssardina/PROJECTS/planning/FOND/myND.git
java -Xmx4g -cp /home/ssardina/PROJECTS/planning/FOND/myND.git/target/mynd-1.0-SNAPSHOT.jar mynd.MyNDPlanner -translatorPath /home/ssardina/PROJECTS/planning/FOND/myND.git/ data/benchmarks-fond/blocksworld_p10.sas -exportPlan plan

MyND FOND Planner (improved/cleanup version by ssardina - 2022)

Heuristic: FF-heuristic.
Algorithm: LAO*-search
INITIAL IS PROVEN!

Result: Strong cyclic plan found.

Time needed for preprocess (Parsing, PDBs, ...):    0.01 seconds.
Time needed for search:                             0.036 seconds.
Time needed:                                        0.046 seconds.
Total Garbage Collections: 1
Total Garbage Collection Time: 0 seconds.

Out of 58 nodes, 19 are proven
Number of node expansions: 19
Policy entries: 10
Number of sensing applications in policy: 0
Plan file: plan
```

An example using PDDL files as input:

```shell
$ ./mynd -type FOND -search LAOSTAR -heuristic FF  \
    data/fond-pddl/blocksworld/domain.pddl data/fond-pddl/blocksworld/p10.pddl
```

Note that this will call the Python translator, whose output will be reported.

## Options available

Use `-h`  to get the list of options available.

We do not print help manually as in the original planner, which means options are not grouped. The (original) groups is as follows:

```shell
myND options:
 -computeCosts                          : compute and print expected costs of
                                          the plan
 -dumpPlan                              : dump plan to stdout
 -exportPlan FILENAME                   : export plan to file
 -exportDot FILENAME                    : export plan as DOT graph (GraphViz)
 -timeout N                             : set timeout in seconds

Search algorithms:
 -s (-search) [AOSTAR | LAOSTAR | FIP]  : set search algorithm [default:
                                          LAOSTAR]

Heuristics:
 -heuristic [FF | PDBS | ZERO | LMCUT   : set heuristic [default: FF]
 | HMAX]                                   

Translate:
 -t (-type) [FOND | POND]               : use fond or pond translate

General POND options:
 -aggregation [MAX | ADD | AVERAGE]     : set aggregation method for world
                                          states (POND only) [default: MAX]
 -numWorldStates N                      : set number of world states to be
                                          sampled (POND only) [default: 10]

CEGAR Options:
 -ca (-cegarAbstraction) [NONE |        : set CEGAR abstraction type [default:
 UNSOLVABILITY_OVERAPPROXIMATION |        NONE]
 UNSOLVABILITY_DOUBLEAPPROXIMATION]        

AO* Options:
 -maxNumberOfNodesToExpand N            : set maximal number of nodes to expand
                                          per iteration [default: 1]
 -senseFirst [ON | OFF]                 : prefer sensing actions (POND only)
                                          [default: ON]

LAO* Options:
 -maxNumberOfNodesToExpand N            : set maximal number of nodes to expand
                                          per iteration [default: 1]
 -senseFirst [ON | OFF]                 : prefer sensing actions (POND only)
                                          [default: ON]

FIP Options:
 -weakPlanAlgorithm [ASTAR | WASTAR5 |  : set search algorithm for weak plan
 WASTAR15 | WASTAR50 | BFS | ENFHC |      search [default: WASTAR5]
 GREEDYBFS]                                
 -searchDirection [GOAL | INTENDED |    : set search direction for fail state
 PARENT]                                  search [default: GOAL]
 -queueType [PRIORITY | FIFO]           : set order in which fail states are
                                          handled [default: PRIORITY]
 -maxDepth N                            : set max depth for fail state search
                                          [default: 100]
 -failStateHeuristic [FF | PDBS | ZERO  : specify heuristic for fail state
 | LMCUT | HMAX]                          search [default: null]

PDB options:
 -patternSearch [NONE | FO | PO]        : set type of pattern search [default:
                                          FO]
 -steps N                               : set maximal number of hill climbing
                                          iterations in pattern search
                                          [default: inf]
 -pdbTimeout N                          : set timeout in seconds for the
                                          pattern search [default: 600]
 -pdbMaxSize N                          : set maximal number of abstract states
                                          induced by a single pattern [default:
                                          -1]
 -pdbsMaxSize N                         : set maximal number of abstract states
                                          of all patterns [default: -1]
 -minImprovement X                      : set fraction of required improvers to
                                          continue pattern search [default: 0.1]
 -cachePDBs [ON | OFF]                  : set caching of PDBs on/off [default:
                                          ON]
 -randomWalkSamples N                   : set number of samples that are
                                          collected during random walks
                                          [default: 1000]
 -assumeFO                              : assume full observability for PDB
                                          heuristic and search (POND only)
                                          [default: false]
 -useDependencyGraph [ON | OFF]         : use dependency graph for
                                          preconditions of sensing actions
                                          (POND only) [default: ON]
```

## Improvements

- [haz's fork](https://github.com/haz/myND) adapted the codebase to be able to compile/run it under newer Java version (Java 8).
- [ssardina's fork](https://github.com/ssardina-planning/myND) fixed bugs with arguments, added option `--translatorPath`, and a script `myND` to run the planner from any directory. Also improved the README.

## Contact

The original `myND` is not in GitHub at: https://github.com/robertmattmueller/myND

The original authors of myND are, in alphabetical order:

 * Pascal Bercher.
 * Robert Mattmüller.
 * Manuela Ortlieb.