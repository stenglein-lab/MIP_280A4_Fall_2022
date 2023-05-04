This directory contains files necessary to create a conda environment that includes the tools necessary to complete the exercises in this course.


To create a new conda environment, change to the directory containing the bio_tools.yaml file and run:
```
conda env create --file bio_tools.yaml
```

The name of this environment will be bio_tools (specified in the yaml file).


To **activate** this conda environment (necessary to run tools in the environment), run: 
```
conda activate bio_tools 
```


To **update** this conda environment with additional conda packages, add them to the .yaml file, change to the directory containing the updated yaml file, and run:
```
conda env update --file bio_tools.yaml
```

If you want to activate this environment by default when you login.  Add the following line to the `.bashrc` file in your user's home directory:

```
conda activate bio_tools
```

