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

Checking whether there are syntax errors in Snakemake
```
snakemake -j1 --dryrun
```

Producing a dependency graph:
```
snakemake --dag | dot -Tpng > simple.png
```

Running the workflow:
```
snakemake -j1
```

This will produce files `../results/A.txt` and `../results/B.txt`. Note that removing `rm ../results/B.txt` and rerunning `snakemake -j1` will recreate `B.txt` but not `A.txt`

### Example with file C depending on A and B

Start with 
```
cd $SNAKEMAKE_EXAMPLES/triad/workflow
rm -rf ../results/*
```
then run
```
snakemake -j1
```

Check that you can remove file `../results/A.txt` and rerun `snakemake -j1`. However, this will not recreate `A.txt` as the final product `C.txt` is still present. On the other hand, `rm ../results/{A,C}.txt` and then running `snakemake -j1` will recreate `A.txt` and `C.txt`, but not `B.txt` since this file was already present.

### Example showing how to aggregate many files into one

Many statistical operations involve combining data from many files into a single number. 

Start with 
```
cd $SNAKEMAKE_EXAMPLES/array/workflow
rm -rf ../results/*
```
then run
```
snakemake -j1
```


