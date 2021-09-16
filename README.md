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

### Simple example

Start with 
```
cd simple/workflow
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

