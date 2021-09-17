# snakemake_examples
Showing how to use snakemake to generate a workflow

## Installing dependencies

After installing Miniconda or Anaconda:
```
conda create -n snakeenv python=3.8
conda activate snakeenv
pip install -r requirements.txt
```

## Running the examples

Checkout out the code:
```
git clone https://github.com/pletzer/snakemake_examples
cd snakemake_examples
export SNAKEMAKE_EXAMPLES=$(pwd)
```

### Simple example

Start with 
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


