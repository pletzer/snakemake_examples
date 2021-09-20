# snakemake_examples

This repository provides examples of how Snakemake can be used to generate a workflow. A workflow takes a number of input files and produces output files.

## How Snakemake works

Snakemake requires a `Snakefile`, which embodies all the rules to generate the output files from the input files. Note that the rules can cascade, that is the output files may depend on some intermediate files, which may depend on some other, temporary files and so on. A rule is a essentially a task that takes some input files and produces output files. 

Snakemake is lazy in that tasks will only be executed if necessary. For instance, if some but not all output files exist then only the tasks reuquired to generate the missing output files will be executed.

In all the examples below, the directory structure is
```
<example>/
├── data
│   ├── mydata_0.txt
│   ├── ...
├── results
└── workflow
    ├── Snakefile
```
The `Snakefile` is stored under workflow. The inout files are under `data/` and the output files will be stored under `results/`. The `snakemake` command will be executed under `workflow/`, at the same level as `Snakefile`.


## Cheking out the code

```
git clone https://github.com/pletzer/snakemake_examples
cd snakemake_examples
export SNAKEMAKE_EXAMPLES=$(pwd)
```

## Installing Snakemake

On mahuika, it suffices to 
```
ml snakemake
```

To install Snakemake on a personal computer, we recommend installing [Miniconda3](https://docs.conda.io/projects/conda/en/latest/user-guide/install/download.html). 
Install scripts exist for Windows, Mac OS X and Linux. Once you have Miniconda3 installed, type:
```
conda create -n snakeenv python=3.8
conda activate snakeenv
cd $SNAKEMAKE_EXAMPLES
pip install -r requirements.txt
```

### A simple example

In this simple example, there is no input file, just a rule `produceA` that creates a file `../results/A.txt` and a rule `produceBFromA` that copies the (empty) file `../results/A.txt` to `../results/B.txt`. 

![Rules](https://github.com/pletzer/snakemake_examples/blob/main/images/simple/worflow.png)


The `Snakefile` file looks like:
```
rule all:
	input:
		"../results/B.txt"

rule produceA:
	output:
		"../results/A.txt"
	shell:
		"touch {output}"

rule produceBFromA:
	input:
		"../results/A.txt"
	output:
		"../results/B.txt"
	shell:
		"cp {input} {output}"
```
Note the final rule `all`, which checks the result file `../results/B.txt` and does not produce anything. This final task is akin a manager checking that the work is complete. 

To run the example, 
```
cd $SNAKEMAKE_EXAMPLES/simple/workflow
rm -rf ../results/*
```

Let's see what the workflow looks like, without actually running it:
```
snakemake -j1 --dryrun
```

Option `-j1` tells Snakemake to run on one core. Workflows can be compute intensive and it may be beneficial to run on more than one core in such cases.

A nice way to visualise the workflow:
```
snakemake --dag | dot -Tpng > workflow.png
```
The above requires the program `dot` to be installed. Dag stands for "directed acylcic graph". In other words the workflow cannot have circular dependencies 
where input depends on the output and output depends in the input.

Now that we're happy with the workflow, let's run it:
```
snakemake -j1
```

This will produce files `../results/A.txt` and `../results/B.txt`. Note that removing `rm ../results/B.txt` and rerunning `snakemake -j1` will recreate `B.txt` but not `A.txt`

### Example with a more complex dependency

The next example
![Rules](https://github.com/pletzer/snakemake_examples/blob/main/images/triad/workflow.png)
involves a final result `C.txt` that depends on two files, `A.txt` and `B.txt`, with `B.txt` depending on `A.txt`.

Start with 
```
cd $SNAKEMAKE_EXAMPLES/triad/workflow
rm -rf ../results/*
```
then run
```
snakemake -j1
```

Try to remove file `../results/A.txt` and then rerun `snakemake -j1`. Note that this will not recreate `A.txt` as the final product `C.txt` is still present. 

Try to remove `../results/{A,C}.txt` and then re-run `snakemake -j1`. This will generate `A.txt` and `C.txt`, but not `B.txt` since this file was already present.

### Example showing how to aggregate many files into one

Statistical operations often involve combining data from many files into a single number. In this example, we start with 100 files, each containing a number which we want to sum up. Although the addition step can be performed in bash, we'll implement it in Python. Snakemake understands Python, we'll program the rules in Python.

Start with 
```
cd $SNAKEMAKE_EXAMPLES/array/workflow
```
The 100 input data files are under `../data`:
```
mydata_0.txt
mydata_1.txt
...
mydata_99.txt
```

File `Snakefile`:
```Python
# import the Python glob module
from glob import glob

rule all:
	input:
		# the single output file that will contain the sum
		"../results/sum.txt"

rule add:
	input:
		# the input files each contain a number
		glob("../data/mydata_*.txt")
	output:
		"../results/sum.txt"
	run:
		# add the number in the input files using Python code
		res = 0 # stores the sum
		for filename in input: # input is a list of file names
			with open(filename) as f: # open each file
				n = int(f.read()) # read the content of the file
				res += n # increment the sum
		with open(output[0], 'w') as out: # write out the result
			out.write(f'{res}\n')
```

To run the workflow, type:
```
rm -rf ../results/*s
snakemake -j1
```
Check that the sum, `cat ../results/sum.txt` is `4950`. 


### Example showing how to submit the tasks to a SLURM sheduler

When tasks take a long time to execute, it is time to run the tasks on a supercomputer. In this example, we'll rerun the tasks under SLURM, the scheduler on NeSI's mahuika platform. 

```
cd $SNAKEMAKE_EXAMPLES/array/workflow
rm -rf ../results/*
snakemake --cluster "sbatch --time=00:01:00 --ntasks=1 --cpus-per-task=1" --jobs=1
```
where `sbatch` is the command to submit a job under SLURM with the other options allocating resources (see [Slurm: reference sheet](https://support.nesi.org.nz/hc/en-gb/articles/360000691716-Slurm-Reference-Sheet) for more info).

### Example of a parallel workflow

Start with
```
cd $SNAKEMAKE_EXAMPLES/parallel/workflow
```

In this example we count the number of times the word "the" appears in variour text files. At the end we generate a table listing the number of "the" words in each file:
![Rules](https://github.com/pletzer/snakemake_examples/blob/main/images/parallel/workflow.png)

Note that rule `countWord` can be executed independently by different jobs. With Snakemake, we have a mechanism for executing jobs concurrently, which reduces the turnaround time. The number of simultaneously running jobs is set by the `--jobs` option. Even though we have three files, here we have decide to allow only two jobs to run concurrently below:
```
snakemake --cluster "sbatch --time=00:01:00 --ntasks=1 --cpus-per-task=1" --jobs=2
```




