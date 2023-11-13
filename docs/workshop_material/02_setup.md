# 02 - Setup

## Running on NeSI vs on your computer

During this workshop we will be running the material on the NeSI platform, using the Jupyter interface, however it is
also possible to run this material locally on your own machine.

One of the differences between running on NeSI or your own machine is that on NeSI we preinstall popular software and make it available to our users, whereas on your own machine you need to install the software yourself (e.g. using a package manager such as conda).

We have provided a guide for setting up your own machine using conda [here](./supplementary/99_appendix_setup_on_your_machine.md)) (note we will not be able to provide assistance if you decide to take this approach during the workshop).

??? jupyter "Connect to Jupyter on NeSI"

    1. Connect to [https://jupyter.nesi.org.nz](https://jupyter.nesi.org.nz)
    2. <p>Enter NeSI username, HPC password and 6 digit second factor token (as set on <a href="https://my.nesi.org.nz/account/hpc-account">MyNeSI</a>)<br>![image](./nesi_images/jupyter_login_labels_updated.png)</p>
    3. <p>Choose server options as below
    <br>make sure to choose the correct project code `nesi02659`, number of CPUs **4**, memory **4GB** prior to pressing ![image](./nesi_images/start_button.png){width="60"} button.

    <br>![image](./nesi_images/jupyter_server2022.png){width="700"}

    4. <p>Start a terminal session from the JupyterLab launcher<br>![image](./nesi_images/jupyterLauncher.png){width="500"}

## Create a working directory

When you connect to NeSI JupyterLab you always start in a new hidden directory. To make sure you can find your work next time, you should change to another location. Here we will switch to our project directory, since home directories can run out of space quickly. If you are using your own project use that instead of "nesi02659".

!!! terminal "code"

    ```
    cd ~/obss_2023/intro_snakemake
    ```

!!! info ""

    **You can also navigate to the above directory in the JupyterLab file browser, which can be useful for editing files and viewing images and html documents.**

## Load the Snakemake module

We use "environment modules" on NeSI to manage installed software. This allows you to pick and choose which software is available in your environment.
More details about environment modules can be found on the NeSI support [page](https://support.nesi.org.nz/hc/en-gb/articles/360000360576-Finding-Software).

!!! terminal-2 "The JupyterLab terminal comes with some modules preloaded and it can often be nicer to start with a clean environment:"

    ```bash
    module purge
    ```

    We can search for available Snakemake modules using the `module spider` command:

    ```bash
    module spider snakemake
    ```

    which shows we have many versions of snakemake installed. Now load a specific version of snakemake into your environment:

    ```bash
    module load snakemake/7.32.3-gimkl-2022a-Python-3.11.3
    ```

    Test that the snakemake command is now available by running the following command:

    ```bash
    snakemake --version
    ```

    It should print out the version of snakemake, i.e. "7.6.2".

    You can also run `module list` to see the list of modules that are currently loaded.

## Clone this repo

Clone this repo with the following:

!!! terminal "code"

    ```bash
    git clone https://github.com/otagobioinformatics/snakemake_workshop.git
    ```
    ```bash
    cd snakemake_workshop
    ```
    !!! info ""

        See the [Git Guides](https://github.com/git-guides/git-clone) for information on cloning a repo

---

<p align="center"><b><a class="btn" href="https://otagobioinformaticsspringschool.github.io/snakemake_workshop/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
