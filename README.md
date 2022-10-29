# myND FOND Planner

This is a fork of the [original myND repository](https://github.com/robertmattmueller/myND). It is itself a fork of [haz's fork](https://github.com/haz/myND) to make it compatible with newer Java, and better documented here.

## Build it

myND requires Java 8 and Maven.

```shell
$ sudo apt-get install openjdk-8-jdk maven
```

To build it, first set-up `JAVA_HOME` to Java 8:

```shell
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/ 
$ mvn package -Dmaven.test.skip=true
```

**Note:** Maven will not compile the system with Java 17.

The above will produce `target/mynd-1.0-SNAPSHOT.jar`.

## Run it

**Note:** while compilation and Maven requires Java 8 (does not compile under Java 17), it seems it does run with Java 17.

**Note:** at this point the planner needs to be called from its root project folder, because it calls the PDDL `translate` Python system in a relative path wrt that location. So one needs to first cd into myND planner folder and then runt it. This means the paths to the PDDL files need to be absolute (or from the myND folder!).

To see all options available:

```shell
$ java -cp target/mynd-1.0-SNAPSHOT.jar mynd.MyNDPlanner -h help
```

The planner can take SAS encodings or original PDDL domain and problem files. For example, to run over a SAS file with 4GB of heap size and output the solution to file `plan` (use `-exportPlan`):

```shell
$ java -Xmx4g -cp target/mynd-1.0-SNAPSHOT.jar mynd.MyNDPlanner data/benchmarks-fond/blocksworld_p10.sas -exportPlan plan
Heuristic: FF-heuristic.
Algorithm: LAO*-search
INITIAL IS PROVEN!

Result: Strong cyclic plan found.

Time needed for preprocess (Parsing, PDBs, ...):    0.011 seconds.
Time needed for search:                             0.02 seconds.
Time needed:                                        0.031 seconds.
Total Garbage Collections: 1
Total Garbage Collection Time: 0 seconds.

Out of 58 nodes, 19 are proven
Number of node expansions: 19
Policy entries: 10
Number of sensing applications in policy: 0
Plan file: plan
```

The same example but now over the original PDDL input files and with some options fixed:

```shell
$ java -Xmx4g -cp target/mynd-1.0-SNAPSHOT.jar mynd.MyNDPlanner \
    -type FOND -search LAOSTAR -heuristic FF  \
    data/fond-pddl/blocksworld/domain.pddl data/fond-pddl/blocksworld/p10.pddl
```

This will also print out the translator's output (from PDDL to SAS).

There is also a script `mynd.sh` to run the planner with default options:

```shell
$ ./mynd.sh data/fond-pddl/blocksworld/domain.pddl data/fond-pddl/blocksworld/p10.pddl
```

### Running from class files

Presumably, one can run the system from the Java class files directly (folder `bin` after build) as follows:

```shell
$ java [java_options] mynd.MyNDPlanner [planner_options] <SAS-file>
```

Useful Java Options are:

* `-Xmx4g`: to set the maximum heap space to 4 GB
* `-ea`: to enable assertions

However, it does not runn when there are planner options given:

```shell
$ cd bin
$ java -ea -Xmx4g mynd.MyNDPlanner -search LAOSTAR -heuristic FF ../data/benchmarks-fond/blocksworld_p1.sas
Option search unknown.
```

## Options available

We do not print help manually as in the original planner, which means options re not grouped. The groups were as follows:

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

## Contact

The original `myND` is not in GitHub at: https://github.com/robertmattmueller/myND

The original authors of myND are, in alphabetical order:

 * Pascal Bercher.
 * Robert Mattmüller.
 * Manuela Ortlieb.