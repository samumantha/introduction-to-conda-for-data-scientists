---
title: "Sharing Environments"
teaching: 30
exercises: 15
questions:
- "Why should I share my Conda environment with others?"
- "How do I share my Conda environment with others?"
- "How do I create a custom kernel for my Conda environments inside JupyterLab?"
objectives:
- "Create an environment from a YAML file that can be read by Windows, Mac OS, or Linux."
- "Create an environment based on exact package versions."
- "Create a custom kernel for a Conda environment for use inside JupyterLab and Jupyter notebooks."
keypoints:
- "Sharing Conda environments with other researchers facilitates the reprodicibility of your research."
- "Create an`environment.yml` file that describes your project's software environment."
- "Creating custom kernels enables you to connect your Conda environments to an existing JupterLab install."
---

## Working with environment files

When working on a collaborative research project it is often the case that your operating system 
might differ from the operating systems used by your collaborators. Similarly, the operating 
system used on a remote cluster to which you have access will likely differ from the operating 
system that you use on your local machine. In these cases it is useful to create an operating 
system agnostic environment file which you can share with collaborators or use to re-create an 
environment on a remote cluster. 

### Creating an environment file

In order to make sure that your environment is truly shareable, you need to make sure that 
that the contents of your environment are described in such a way that the resulting 
environment file can be used to re-create your environment on Linux, Mac OS, and Windows. Conda 
uses YAML ("YAML Ain't Markup Language") for writing its environment files. YAML is a 
human-readable data-serialization language that is commonly used for configuration files and that 
uses Python-style indentation to indicate nesting.

Creating your project's Conda environment from a single environment file is a Conda "best practice". 
Not only do you have a file to share with collaborators but you also have a file that can be placed 
under version control which further enhancing the reproducibility of your research project and 
workflow.

> ## Default `environment.yml` file
> 
> Note that by convention Conda environment files are called `environment.yml`. As such if you use 
> the `conda env create` sub-command without passing the `--file` option, then `conda` will expect to 
> find a file called `environment.yml` in the current working directory and will throw an error if a 
> file with that name can not be found.
{: .callout}

Let's take a look at a few example `environment.yml` files  to give you an idea of how to write 
your own environment files.

~~~
name: machine-learning-env

dependencies:
  - ipython
  - matplotlib
  - pandas
  - pip
  - python
  - scikit-learn
~~~
{: .language-yaml}

This `environment.yml` file would create an environment called `machine-learning-env` with the 
most current and mutually compatible versions of the listed packages (including all required 
dependencies). The newly created environment would be installed inside the `~/miniconda3/envs/` 
directory. Alternatively, if we intended this environment file to be used to create an environment 
inside a sub-directory call `./env` of the project directory, we can set the `name` key 
to `null` as follows, since the environment will be named by path.

~~~
name: null

dependencies:
  - ipython
  - matplotlib
  - pandas
  - pip
  - python
  - scikit-learn
~~~
{: .language-yaml}

Finally, since explicit versions numbers for all packages should be preferred a better environment 
file would be the following.

~~~
name: null

dependencies:
  - ipython=7.13
  - matplotlib=3.1
  - pandas=1.0
  - pip=20.0
  - python=3.6
  - scikit-learn=0.22
~~~
{: .language-yaml}

Note that we are only specifying the major and minor version numbers and not the patch or build 
numbers. Defining the version number by fixing only the major and minor version numbers while 
allowing the patch version number to vary allows us to use our environment file to update our 
environment to get any bug fixes whilst still maintaining significant consistency of our 
Conda environment across updates.

> ## *Always* version control your `environment.yml` files!
>
> While you should *never* version control the contents of your `env/` environment sub-directory, 
> you should *always* version control your `environment.yml` files. Version controlling your 
> `environment.yml` files together with your project's source code means that you always know 
> which versions of which packages were used to generate your results at any particular point in 
> time.
{: .callout}

Let's suppose that you want to use the `environment.yml` file defined above to create a Conda 
environment in a sub-directory of some project directory. Here is how you would accomplish this 
task.

~~~
$ cd project-dir
$ conda env create --prefix ./env --file environment.yml
$ conda activate ./env
~~~
{: .language-bash}

Note that the above sequence of commands assumes that the `environment.yml` file is stored within 
your `project-dir` directory. 

> ## Beware the `conda env export` command
>  
> Many other Conda tutorials (including the official documentation) encourage the use of the 
> `conda env export` command to export an existing environment. For example, to export the packages 
> installed into the previously created `machine-learning-env` you would run the following command.
> 
> ~~~
> $ conda env export --name machine-learning-env --no-builds
> ~~~
> {: .language-bash}
> 
> When you run this command, you will see the resulting YAML formatted representation of your Conda 
> environment streamed to the terminal. Recall that we only listed five packages when we 
> originally created `machine-learning-env` yet from the output of the `conda env export` command 
> we see that these five packages result in an environment with roughly 80 dependencies!
> 
> In practice this command does not *consistently* produce environments that are reproducible 
> across Mac OS, Windows, and Linux. The issue is that even after removing the build numbers (by 
> passing the `--no-builds` option), an environment file exported from an environment created on, 
> say Mac OS, will often still contain Mac OS specific packages that will not exist for Windows 
> or Linux.
{: .callout}

> ## Create a new environment from a YAML file.
> 
> Create a new project directory and then create a new `environment.yml` file inside your project 
> directory with the following contents.
>
> ~~~
> name: scikit-env
> 
> dependencies:
>   - ipython=7.13
>   - matplotlib=3.1
>   - pandas=1.0
>   - pip=20.0
>   - python=3.6
>   - scikit-learn=0.22
> ~~~
> {: .language-yaml}
>
> Now use this file to create a new Conda environment. Where is this new environment created? 
> Using the same `environment.yml` file create a Conda environment as a sub-directory called 
> `env/` inside a newly created project directory. Compare the contents of the two environments.
> 
> > ## Solution
> > 
> > To create a new environment from a YAML file use the `conda env create` sub-command as follows.
> > 
> > ~~~
> > $ mkdir project-dir
> > $ cd project-dir
> > $ nano environment.yml
> > $ conda env create --file environment.yml
> > ~~~
> > {: .language-bash}
> >
> > The above sequence of commands will create a new Conda environment inside the 
> > `~/miniconda3/envs` directory. In order to create the Conda environment inside a sub-directory 
> > of the project directory you need to pass the `--prefix` to the `conda env create` command as 
> > follows.
> > 
> > ~~~
> > $ conda env create --file environment.yml --prefix ./env
> > ~~~
> > {: .language-bash}
> > 
> > You can now run the `conda env list` command and see that these two environments have been 
> > created in different locations but contain the same packages.
> {: .solution}
{: .challenge}

### Updating an environment

You are unlikely to know ahead of time which packages (and version numbers!) you will need to use 
for your research project. For example it may be the case that...  

*   ...one of your core dependencies just released a new version (dependency version number update).
*   ...you need an additional package for data analysis (add a new dependency).
*   ...you have found a better visualization package and no longer need to old visualization package 
    (add new dependency and remove old dependency).

If any of these occurs during the course of your research project, all you need to do is update 
the contents of your `environment.yml` file accordingly and then run the following command.

~~~
$ conda env update --prefix ./env --file environment.yml  --prune
~~~
{: .language-bash}

Note that the `--prune` option cause Conda to remove any dependencies that are no longer required 
from the environment.

> ## Rebuilding a Conda environment from scratch
> 
> When working with `environment.yml` files it is often just as easy to rebuild the Conda 
> environment from scratch whenever you need to add or remove dependencies. To rebuild a Conda 
> environment from scratch you simply pass the `--force` option to the `conda env create` command 
> which will remove any existing environment directory before rebuilding it using the provided 
> environment file. 
> ~~~
> $ conda env create --prefix ./env --file environment.yml --force
> ~~~
> {: .language-bash}
{: .callout}

> ## Add Dask to the environment to scale up your analytics
> 
> Add to the `scikit-env` environment file and update the environment. [Dask](https://dask.org/) 
> provides advanced parallelism for data science workflows enabling performance at scale for the 
> core Python data science tools such as Numpy Pandas, and Scikit-Learn.  
>
> > ## Solution
> > 
> > The `environment.yml` file should now look as follows.
> > 
> > ~~~
> > name: scikit-env
> >
> > dependencies:
> >   - dask=2.16
> >   - dask-ml=1.4
> >   - ipython=7.13
> >   - matplotlib=3.1
> >   - pandas=1.0
> >   - pip=20.0
> >   - python=3.6
> >   - scikit-learn=0.22
> > ~~~
> > 
> > The following command will rebuild the environment from scratch with the new Dask dependencies.
> > 
> > ~~~
> > $ conda env create --prefix ./env --file environment.yml --force 
> > ~~~
> > {: .language-bash}
> > 
> > The following command will update the envirionment in-place with the new Dask dependencies.
> > 
> > ~~~
> > $ conda env update --prefix ./env --file environment.yml  --prune
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}


> ## Optional exercise: 
> 
> If you are working on a project of your own, now is the time to create a conda environment for your project! 
> If you do not (yet) have a project of your own you can use the (Python) project description below.
> 
> Project: The project aims at creating interactive visualizations based on data from a csv file in Python. 
> In order to read the csv file and do some preprocessing, pandas (https://pandas.pydata.org/) is used. 
> For the interactive visualization bokeh (https://docs.bokeh.org/en/latest/index.html) is chosen. 
> 
> Create a conda environment for the project. You can choose where you want to store the environment.
> Make your environment shareable and reproducible.
> What would you do, if a colleague provides you with old scripts for the preprocessing that depend on numpy version 1.9?
>
>
> > ## Solution
> > 
> > 1. Create conda environment
> > Here you got a few options:
> > 1.1 Create an environment file with latest versions (that are available from Anaconda)
> > ~~~
> > name: interactive_viz_env
> >
> > dependencies:
> >   - python=3.6
> >   - pandas=1.2.0
> >   - bokeh=2.2.3
> > ~~~
> > and then:
> > 
> > ~~~
> > $ conda env create --file environment.yml
> > ~~~
> > {: .language-bash}
> >
> > Note that this way, all versions are chosen for you (latest), also the Python version.
> > 
> > 1.2 Create an empty environment and install packages into it
> > 
> > ~~~
> > $ conda create --name interactive_viz_env python=3.6
> > $ conda activate interactive_viz_env
> > ~~~
> > {: .language-bash}
> >
> > This creates an environment with Python 3.6 (you can also choose another) version and then activates it.
> >
> > ~~~
> > $ conda install pandas=1.2.0
> > $ conda install bokeh=2.2.3
> > ~~~
> > {: .language-bash}
> > 
> > Both 1.1 and 1.2 will create a conda environment with the requested packages in a way that they will work together.
> > Check with `conda list` (from within the environment, ie after `conda activate`) which versions of packages have been installed.
> > Now you can write code that uses these versions. When you are done, you may want to share that code with a colleague.
> > In order to use your code or reproduce your results, your colleague will also need an environment.yml file to reproduce your software environment.
> > The environment file from 1.1 would not be sufficient, as new versions may have come out since you created it and then the latest version 
> > your colleague gets, is different from what you used.
> > So, in this case it will be necessary to create an updated environment.yml with version numbers, which can for example be done like this 
> > (from within the environment: 
> > ~~~
> > $ conda env export --from-history > environment.yml
> > ~~~
> > {: .language-bash}
> > 
> > ....
> >
> > Note that the latest pandas version depends on minimum numpy version 1.16, the latest bokeh on minimum numpy version 1.11.
> > For the first part, this is not an issue, as conda will find the newest numpy version that is compatible with both.
> {: .solution}
{: .challenge}


## Making Jupyter aware of your Conda environments

Both JupyterLab and Jupyter Notebooks automatically ensure that the standard IPython kernel is 
always available by default. However, if you want to use a kernel based on a particular Conda 
environment from inside Jupyter (and Juptyer is *not* installed inside your environment) then will 
need to create a 
[kernel spec](https://jupyter-client.readthedocs.io/en/latest/kernels.html#kernelspecs) file for 
your Conda environments manually.

Before you can create a custom kernel for you Conda environment you need to make sure that the 
[`ipykernel`](https://pypi.org/project/ipykernel/) package is installed in your Conda environment 
as you will need to use this package to create the kernel spec file. Here is the updated `xgboost-env`
`environment.yml` file that includes the `ipykernel` package.

~~~
name: xgboost-env

dependencies:
  - ipykernel=5.3
  - ipython=7.13
  - matplotlib=3.1
  - pandas=1.0
  - pip=20.0
  - python=3.6
  - scikit-learn=0.22
  - xgboost=1.0
~~~
{: .language-yaml}

Next, rebuild the Conda environment using the following command.

~~~
$ conda env create --prefix ./env --file environment.yml --force
~~~

Once the Conda environment has been re-built you can activate the environment and then create the 
custom kernel for the activated environment.

~~~
$ conda activate ./env
$ python -m ipykernel install --user --name xgboost-env --display-name "XGBoost"
~~~

The last command installs a kernel spec file for the current environment. Kernel spec files are 
JSON files which can be viewed and changed with a normal text editor. The `--name` value is used 
by Jupyter internally; `--display-name` is what you see in the JupyterLab launcher menu as well as 
the Jupyter Notebook dropdown kernel menu. This command will overwrite any existing kernel with 
the same name. 

> ## Create a kernel for a Conda environment
>
> Create a custom kernel for the `machine-learning-env` environment created in a previous challenge.
> 
> > ## Solution
> > 
> > In order to activate an existing environment by name you use the `conda activate` command as 
> > follows.
> > 
> > ~~~
> > $ conda activate machine-learning-env
> > $ python -m ipykernel install --user --name machine-learning-env
> > ~~~
> > {: .language-bash}
> > 
> > Note that by leaving the `--display-name` unspecified, the display name will match the value 
> > provided to `--name` .
> >
> {: .solution}
{: .challenge}

{% include links.md %}

