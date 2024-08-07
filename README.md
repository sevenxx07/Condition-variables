# Formulation of the problem 

This program is simulating a manufacturing plant that produces 3 types of products. Each product must go through a fixed sequence of operations at different workplaces. 
Workplaces are staffed by workers, and each worker is capable of servicing just one type of workplace. The number of workers and their workplaces is different and variable 
over time – workers come and go, workplaces can be added or eliminated. Requests for production, arrivals and departures of workers, and purchase/retirement of machines 
are entered through the application's standard input with the following commands (one command per line terminated by '\n'):<br />

- make <product> – request to make the product; <product> is “A”, “B”, or “C”
- start <name> <workplace> – the arrival of a worker with an indication of his specialization
- end <name> – worker exit
- add <workplace> – adding a new workplace
- remove <workplace> – removal of a workplace

Parameters are separated by a space. If you enter an invalid command or parameter (e.g. non-existent workplace), or the wrong number of parameters, ignore the entire line.
The production process is controlled by workers. Each worker will be represented by a separate thread that will be created when the worker arrives and terminated when the worker leaves.
A worker needs a semi-finished product (intermediate product) and a free workplace for which he is specialized for his work. If he doesn't have one or the other, he waits. 
For the first workplace, the intermediate product replaces the production requirement. If the worker has everything he needs at his disposal, he writes information about his 
activity and then waits the time that is defined for each type of operation. The activity listing format is <name> <job> <step> <product> so e.g.: <br />
```
Karel vrtacka 2 A
```
If the worker completes the last operation in the process, it will print
```
done <product>
```
where <product> is product code A, B, or C.<br />
We can assume that no two workers with the same name will come. If a workplace is being removed, preferably remove vacant ones. If no workplace of the given type is free, discard any one, but only after the current operation is completed.
The worker leaves the workplace either when terminating the entire application (see below) or as a response to the end command. The worker who is to leave first finishes the currently divided work. If a worker is to leave based on an end 
command, he must not take a new job. When exiting, the worker writes to standard output:<br />
```
<name> goes home
```
and then terminates its thread.<br />
If a worker can work at several different locations at a given time, he chooses the location with the highest possible step so that jobs closest to completion are processed as a priority. 
If there are more of them, it selects the product closer to the beginning of the alphabet, i.e. product A before B.

Closing standard input (ie your program reads EOF from stdin) is a requirement to terminate the entire application. After this requirement, the worker leaves at the moment when no one can 
work on any product that his profession can work on (due to missing workers, jobs, or intermediate products). Until then, he waits and if a job appears, he will start working again. 
Termination of the application occurs after the departure of all workers.


# Factory varaibles

The manufacturing processes for products A–C are as follows:<br />
| --- | 1 | 2 | 3 | 4 | 5 | 6
| A | nuzky |	vrtacka |	ohybacka |	svarecka |	vrtacka |	lakovna
| B | vrtacka |	nuzky |	freza |	vrtacka |	lakovna |	sroubovak
| C | freza |	vrtacka |	sroubovak |	vrtacka |	freza |	lakovna

Operation times in milliseconds at the workstations are as follows:<br />
* nuzky: 100
* vrtacka: 200
* ohybacka: 150
* svarecka: 300
* lakovna: 400
* sroubovak: 250
* freza: 500

# Implementation 

The waiting of all threads must be efficient, the processor must not be exploited (busy waiting).
I don't create threads other than worker threads (it would be a problem for the evaluator).
If resources are available, the worker begins work immediately (as quickly as possible, without undue delay).
Executing commands from the input without undue delay, i.e. do not block the main thread unless absolutely necessary. The goal is that the main thread is not blocked and can respond quickly to sent commands.<br />

C/C++<br />
The application binary will be called factory and will be created in the same directory as the Makefile.
Compile the program with the -Wall -g -O2 flags and additionally with the flags in the EXTRA_CFLAGS variable. If this variable is not defined on the make command line, set its value to “-fsanitize=address -fno-omit-frame-pointer” 
(see e.g. the ?= operator). If you do translation and linking separately, use the flags in EXTRA_CFLAGS when linking as well.

# Example for testing 

Input:
```
add nuzky
add vrtacka
add ohybacka
add svarecka
add lakovna
add sroubovak
add freza

start Nora nuzky
start Vojta vrtacka
start Otakar ohybacka
start Sofie svarecka
start Lucie lakovna
start Stepan sroubovak
start Filip freza

make A
```
Output:
```
Nora nuzky 1 A
Vojta vrtacka 2 A
Otakar ohybacka 3 A
Sofie svarecka 4 A
Vojta vrtacka 5 A
Lucie lakovna 6 A
done A
```
Input:
```
$ ./factory <<EOF
start Nora nuzky
add nuzky
make A
EOF
```
Output:
```
Nora nuzky 1 A
Nora goes home
```
