# 03 - Create a basic workflow

## 3.01 Aim

---

!!! info ""

    Let's create a basic workflow that will do some of the analysis steps for genetic data. We will have three samples with two files each - six files in total. These files will be processed through the below workflow, passing through three software.

---

<center>
![rulegraph_1](./images/rulegraph_1.png)
</center>

We have paired end sequencing data for three ecoli samples `SRR2584863`, `SRR2584866`, and `SRR2589044` to process in the `./data` directory. Let's have a look:

!!! terminal "code"

    ```bash
    ls -lh data/trimmed_fastq_small/
    ```

    ??? success "output"

        ```bash
        total 332M
        -rw-rw----+ 1 murray.cadzow nesi02659 58M Jul  6  2018 SRR2584863_1.trim.sub.fastq
        -rw-rw----+ 1 murray.cadzow nesi02659 53M Jul  6  2018 SRR2584863_2.trim.sub.fastq
        -rw-rw----+ 1 murray.cadzow nesi02659 55M Jul  6  2018 SRR2584866_1.trim.sub.fastq
        -rw-rw----+ 1 murray.cadzow nesi02659 57M Jul  6  2018 SRR2584866_2.trim.sub.fastq
        -rw-rw----+ 1 murray.cadzow nesi02659 58M Jul  6  2018 SRR2589044_1.trim.sub.fastq
        -rw-rw----+ 1 murray.cadzow nesi02659 53M Jul  6  2018 SRR2589044_2.trim.sub.fastq
        ```

<br>

## 3.02 Snakemake workflow file structure

Workflow file structure:

```bash
data/
    |_______ref_genome/
    |_______trimmed_fastq_small/
demo_workflow/
      |_______results/
      |_______workflow/
                 |_______Snakefile
```

We will create and run our workflow from the `workflow` directory send all of our file outputs/results to the `results` directory

_Read up on the best practice workflow structure [here](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility)_

Create this file structure and our main Snakefile with:

!!! terminal "code"

    ```bash
    mkdir -p demo_workflow/{results,workflow}
    ```
    ```bash
    touch demo_workflow/workflow/Snakefile
    ```

??? info "Add snakefile to git (optional recommended)"

    ```bash
    git add deme_worflow/workflow/Snakefile
    git commit -m 'start of snakefile'
    ```

Now you should have the very beginnings of your Snakemake workflow in a `demo_workflow` directory. Let's have a look:

!!! terminal "code"

    ```bash
    ls -lh demo_workflow/
    ```

    ??? success "output"


        ```bash
        total 1.0K
        drwxrws---+ 2 murray.cadzow nesi02659 4.0K May 11 12:07 results
        drwxrws---+ 2 murray.cadzow nesi02659 4.0K May 11 12:07 workflow
        ```

<br>
!!! terminal "code"

    ```bash
    ls -lh demo_workflow/workflow/
    ```

    ??? success "output"

        ```bash
        total 0
        -rw-rw----+ 1 murray.cadzow nesi02659 0 May 11 12:07 Snakefile
        ```

<br>

Within the `workflow` directory (where we will create and run our workflow), we have a `Snakefile` file that will be the backbone of our workflow.

## 3.03 Run the software on the command line

First lets run the first step in our workflow ([fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)) directly on the command line to get the syntax of the command right and check what outputs files we expect to get. Knowing what files the software will output is important for Snakemake since it is a lazy "pull" based system where software/rules will only run if you tell it to create the output file. We will talk more about this later!

!!! terminal "code"

    - First make sure to have [fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) available. On NeSI, load the corresponding module
    ```bash
    module load FastQC/0.12.1
    ```

    - See what parameters are available so we know how we want to run this software before we put it in a Snakemake workflow
    ```bash
    fastqc --help
    ```

    - Create a test directory to put the output files
    ```bash
    mkdir fastqc_test
    ```

    Run fastqc directly on the command line on one of the samples
    ```bash
    fastqc ./data/trimmed_fastq_small/SRR2584863_1.trim.sub_fastq ./data/trimmed_fastq_small/SRR2584863_2.trim.sub_fastqc -o ./fastqc_test -t 2
    ```

    ??? success "output"

        ```bash
        Started analysis of SRR2584863_1.trim.sub_fastq
        Approx 5% complete for SRR2584863_1.trim.sub_fastq
        Approx 10% complete for SRR2584863_1.trim.sub_fastq
        Approx 15% complete for SRR2584863_1.trim.sub_fastq
        Approx 20% complete for SRR2584863_1.trim.sub_fastq
        Approx 25% complete for SRR2584863_1.trim.sub_fastq
        Approx 30% complete for SRR2584863_1.trim.sub_fastq
        Approx 35% complete for SRR2584863_1.trim.sub_fastq
        Approx 40% complete for SRR2584863_1.trim.sub_fastq
        Approx 45% complete for SRR2584863_1.trim.sub_fastq
        Approx 50% complete for SRR2584863_1.trim.sub_fastq
        Approx 55% complete for SRR2584863_1.trim.sub_fastq
        Approx 60% complete for SRR2584863_1.trim.sub_fastq
        Started analysis of SRR2584863_2.trim.sub_fastq
        Approx 65% complete for SRR2584863_1.trim.sub_fastq
        Approx 5% complete for SRR2584863_2.trim.sub_fastq
        Approx 70% complete for SRR2584863_1.trim.sub_fastq
        Approx 10% complete for SRR2584863_2.trim.sub_fastq
        Approx 75% complete for SRR2584863_1.trim.sub_fastq
        Approx 15% complete for SRR2584863_2.trim.sub_fastq
        Approx 80% complete for SRR2584863_1.trim.sub_fastq
        Approx 20% complete for SRR2584863_2.trim.sub_fastq
        Approx 25% complete for SRR2584863_2.trim.sub_fastq
        Approx 85% complete for SRR2584863_1.trim.sub_fastq
        Approx 90% complete for SRR2584863_1.trim.sub_fastq
        Approx 30% complete for SRR2584863_2.trim.sub_fastq
        Approx 35% complete for SRR2584863_2.trim.sub_fastq
        Approx 95% complete for SRR2584863_1.trim.sub_fastq
        Approx 40% complete for SRR2584863_2.trim.sub_fastq
        Analysis complete for SRR2584863_1.trim.sub_fastq
        Approx 45% complete for SRR2584863_2.trim.sub_fastq
        Approx 50% complete for SRR2584863_2.trim.sub_fastq
        Approx 55% complete for SRR2584863_2.trim.sub_fastq
        Approx 60% complete for SRR2584863_2.trim.sub_fastq
        Approx 65% complete for SRR2584863_2.trim.sub_fastq
        Approx 70% complete for SRR2584863_2.trim.sub_fastq
        Approx 75% complete for SRR2584863_2.trim.sub_fastq
        Approx 80% complete for SRR2584863_2.trim.sub_fastq
        Approx 85% complete for SRR2584863_2.trim.sub_fastq
        Approx 90% complete for SRR2584863_2.trim.sub_fastq
        Approx 95% complete for SRR2584863_2.trim.sub_fastq
        Analysis complete for SRR2584863_2.trim.sub_fastq
        ```

    <br>

    - What are the output files of fastqc? Find out with:
    ```bash
    ls -lh ./fastqc_test
    ```

    ??? success "output"

        ```bash
        total 2.5M
        -rw-rw----+ 1 murray.cadzow nesi02659 718K May 11 12:08 SRR2584863_1.trim.sub_fastqc.html
        -rw-rw----+ 1 murray.cadzow nesi02659 475K May 11 12:08 SRR2584863_1.trim.sub_fastqc.zip
        -rw-rw----+ 1 murray.cadzow nesi02659 726K May 11 12:08 SRR2584863_2.trim.sub_fastqc.html
        -rw-rw----+ 1 murray.cadzow nesi02659 479K May 11 12:08 SRR2584863_2.trim.sub_fastqc.zip
        ```

<br>

## 3.04 Create the first rule in your workflow

!!! file-code "Let's wrap this up in a Snakemake workflow! Start with the basic structure of a Snakefile:"

    ```python
    # target OUTPUT files for the whole workflow
    rule all:
        input:

    # workflow
    rule my_rule:
        input:
            ""
        output:
            ""
        threads:
        shell:
            ""
    ```

!!! scale-balance "Now add our fastqc rule, let's:"

    - Name the rule
    - Fill in the the input fastq files from the `data` directory (*path relative to the Snakefile*)
    - Fill in the output files (now you can see it's useful to know what files fastqc outputs!)
    - Set the number of threads
    - Write the fastqc shell command in the `shell:` section and pass the input/output variables to the shell command
    - Set the final output files for the whole workflow in `rule all:`

---

The use of the word `input` in `rule all` can be confusing, but in this context, it is referring to the final _output_ files of the whole workflow

---

??? code-compare "Edit snakefile"

    ```diff
    # target OUTPUT files for the whole workflow
    rule all:
        input:
    +       "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
    +       "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
    +       "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
    +       "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"

    # workflow
    - rule my_rule:
    + rule fastqc:
        input:
    +       R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
    +       R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
        output:
    +       html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
    +       zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
    +   threads: 2
        shell:
    +        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    ```

??? file-code "Current snakefile:"

    ```python
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
        threads: 2
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    ```

<br>

When you have multiple input and output files:

- You can "name" you inputs/outputs, they can be called separately in the shell command
- Remember to use commas between multiple inputs/outputs, it's a common source of error!

Let's test the workflow! First we need to be in the `workflow` directory, where the Snakefile is

!!! terminal "code"

    ```bash
    cd demo_workflow/workflow/
    ```

## 3.05 Dryrun

Then let's carry out a dryrun of the workflow, where no actual analysis is undertaken (fastqc is _not_ run) but the overall Snakemake structure is run/validated. This is a good way to check for errors in your Snakemake workflow before actually running your workflow.

!!! terminal "code"

    ```bash
    snakemake --dryrun
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2


        [Mon Nov 13 03:09:34 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1_fastqc.html, ../results/fastqc/SRR2584863_2_fastqc.html, ../results/fastqc/SRR2584863_1_fastqc.zip, ../results/fastqc/SRR2584863_2_fastqc.zip
            jobid: 1
            reason: Missing output files: ../results/fastqc/SRR2584863_1_fastqc.html, ../results/fastqc/SRR2584863_1_fastqc.zip, ../results/fastqc/SRR2584863_2_fastqc.zip, ../results/fastqc/SRR2584863_2_fastqc.html
            resources: tmpdir=/dev/shm/jobs/40901556


        [Mon Nov 13 03:09:34 2023]
        localrule all:
            input: ../results/fastqc/SRR2584863_1_fastqc.html, ../results/fastqc/SRR2584863_2_fastqc.html, ../results/fastqc/SRR2584863_1_fastqc.zip, ../results/fastqc/SRR2584863_2_fastqc.zip
            jobid: 0
            reason: Input files updated by another job: ../results/fastqc/SRR2584863_1_fastqc.zip, ../results/fastqc/SRR2584863_2_fastqc.zip, ../results/fastqc/SRR2584863_1_fastqc.html, ../results/fastqc/SRR2584863_2_fastqc.html
            resources: tmpdir=/dev/shm/jobs/40901556

        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2

        Reasons:
            (check individual jobs above for details)
            input files updated by another job:
                all
            missing output files:
                fastqc

        This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
        ```

<br>

The last table in the output confirms that the workflow will run one sample (`count 1`) through fastqc (`job fastqc`)

## 3.06 Create a DAG

We can also visualise our workflow by creating a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG). We can tell snakemake to create a DAG with the `--dag` flag, then pipe this output to the [dot software](https://graphviz.org/) and write the output to the file, `dag_1.png`

!!! terminal "code"

    ```bash
    snakemake --dag | dot -Tpng > dag_1.png
    ```

    ??? success "DAG:"

        <center>![DAG_1](./images/dag_1.png)</center>

<br>

Our diagram has a node for each job which are connected by edges representing dependencies

_Note. this diagram can be output to several other image formats such as svg or pdf_

## 3.07 Fullrun

Let's do a full run of our workflow (by removing the `--dryrun` flag). We will also now need to specify the maximum number of cores to use at one time with the `--cores` flag before snakemake will run

!!! terminal "code"

    ```bash
    snakemake --cores 2
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Using shell: /usr/bin/bash
        Provided cores: 2
        Rules claiming more threads will be scaled down.
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2

        Select jobs to execute...

        [Mon Nov 13 03:14:57 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 1
            reason: Missing output files: ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html
            threads: 2
            resources: tmpdir=/dev/shm/jobs/40901556

        Started analysis of SRR2584863_1.trim.sub.fastq
        Approx 5% complete for SRR2584863_1.trim.sub.fastq
        Started analysis of SRR2584863_2.trim.sub.fastq
        Approx 10% complete for SRR2584863_1.trim.sub.fastq
        Approx 5% complete for SRR2584863_2.trim.sub.fastq
        Approx 15% complete for SRR2584863_1.trim.sub.fastq
        Approx 10% complete for SRR2584863_2.trim.sub.fastq
        Approx 20% complete for SRR2584863_1.trim.sub.fastq
        Approx 15% complete for SRR2584863_2.trim.sub.fastq
        Approx 25% complete for SRR2584863_1.trim.sub.fastq
        Approx 20% complete for SRR2584863_2.trim.sub.fastq
        Approx 30% complete for SRR2584863_1.trim.sub.fastq
        Approx 25% complete for SRR2584863_2.trim.sub.fastq
        Approx 35% complete for SRR2584863_1.trim.sub.fastq
        Approx 30% complete for SRR2584863_2.trim.sub.fastq
        Approx 40% complete for SRR2584863_1.trim.sub.fastq
        Approx 35% complete for SRR2584863_2.trim.sub.fastq
        Approx 45% complete for SRR2584863_1.trim.sub.fastq
        Approx 40% complete for SRR2584863_2.trim.sub.fastq
        Approx 50% complete for SRR2584863_1.trim.sub.fastq
        Approx 45% complete for SRR2584863_2.trim.sub.fastq
        Approx 55% complete for SRR2584863_1.trim.sub.fastq
        Approx 50% complete for SRR2584863_2.trim.sub.fastq
        Approx 55% complete for SRR2584863_2.trim.sub.fastq
        Approx 60% complete for SRR2584863_1.trim.sub.fastq
        Approx 60% complete for SRR2584863_2.trim.sub.fastq
        Approx 65% complete for SRR2584863_1.trim.sub.fastq
        Approx 65% complete for SRR2584863_2.trim.sub.fastq
        Approx 70% complete for SRR2584863_1.trim.sub.fastq
        Approx 70% complete for SRR2584863_2.trim.sub.fastq
        Approx 75% complete for SRR2584863_1.trim.sub.fastq
        Approx 75% complete for SRR2584863_2.trim.sub.fastq
        Approx 80% complete for SRR2584863_1.trim.sub.fastq
        Approx 80% complete for SRR2584863_2.trim.sub.fastq
        Approx 85% complete for SRR2584863_1.trim.sub.fastq
        Approx 85% complete for SRR2584863_2.trim.sub.fastq
        Approx 90% complete for SRR2584863_1.trim.sub.fastq
        Approx 90% complete for SRR2584863_2.trim.sub.fastq
        Approx 95% complete for SRR2584863_1.trim.sub.fastq
        Approx 95% complete for SRR2584863_2.trim.sub.fastq
        Approx 100% complete for SRR2584863_2.trim.sub.fastq
        Approx 100% complete for SRR2584863_1.trim.sub.fastq

        [Mon Nov 13 03:15:05 2023]
        Finished job 1.
        1 of 2 steps (50%) done
        Select jobs to execute...

        [Mon Nov 13 03:15:05 2023]
        localrule all:
            input: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 0
            reason: Input files updated by another job: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            resources: tmpdir=/dev/shm/jobs/40901556

        [Mon Nov 13 03:15:05 2023]
        Finished job 0.
        2 of 2 steps (100%) done
        Complete log: .snakemake/log/2023-11-13T031457.382959.snakemake.log

        ```

<br>

It worked! Now in our results directory we have our output files from fastqc. Let's have a look:

!!! terminal "code"
`bash
    ls -lh ../results/fastqc/
    `

    ??? success "output"
        ```bash
        total 1.5M
        -rw-rw----+ 1 murray.cadzow nesi02659 239K Nov 13 03:52 SRR2584863_1.trim.sub_fastqc.html
        -rw-rw----+ 1 murray.cadzow nesi02659 284K Nov 13 03:52 SRR2584863_1.trim.sub_fastqc.zip
        -rw-rw----+ 1 murray.cadzow nesi02659 243K Nov 13 03:52 SRR2584863_2.trim.sub_fastqc.html
        -rw-rw----+ 1 murray.cadzow nesi02659 293K Nov 13 03:52 SRR2584863_2.trim.sub_fastqc.zip
        ```

{% include exercise.html title="e3dot10" content=e3dot10%}
<br>

## 3.08 Lazy evaluation

What happens if we try a dryrun or full run now?

!!! terminal "code"

    ```bash
    snakemake --dryrun --cores 2
    ```

    !!! success "output"

        ```bash
        Building DAG of jobs...
        Nothing to be done (all requested files are present and up to date).
        ```

<br>

!!! terminal "code"

    ```bash
    snakemake --cores 2
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Nothing to be done (all requested files are present and up to date).
        Complete log: .snakemake/log/2023-11-13T035419.940732.snakemake.log
        ```

<br>

Nothing happens, all the target files in `rule all` have already been created so Snakemake does nothing

Also, what happens if we create another directed acyclic graph (DAG) after the workflow has been run?

!!! terminal "code"

    ```bash
    snakemake --dag | dot -Tpng > dag_2.png
    ```

    ??? image "DAG"

        <center>![DAG_2](./images/dag_2.png)</center>

<br>

Notice our workflow 'job nodes' are now dashed lines, this indicates that their output is up to date and therefore the rule doesn't need to be run. We already have our target files!

This can be quite informative if your workflow errors out at a rule. You can visually check which rules successfully ran and which didn't.

## 3.09 Run using environment modules

fastqc worked because we loaded it in our current shell session. Let's specify the environment module for fastqc so the user of the workflow doesn't need to load it manually.

??? code-compare "Edit snakefile "Update our rule to use it using the `envmodules:` directive"

    ```diff
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
        threads: 2
        envmodules:
            "FastQC/0.12.1"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    ```

??? file-code "Current snakefile:"

    ```python
    # target OUTPUT files for the whole workflow
        rule all:
        input:
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
        threads: 2
        envmodules:
            "FastQC/0.12.1"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    ```

<br>

Run again, now telling Snakemake to use [environment modules](https://nesi.github.io/hpc-intro/14-modules/index.html) to automatically load our software by using the `--use-envmodules` flag

!!! terminal "code"

    ```diff
    # first remove output of last run
    rm -r ../results/*

    # Run dryrun again
    - snakemake --dryrun --cores 2
    + snakemake --dryrun --cores 2 --use-envmodules
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2


        [Mon Nov 13 03:58:29 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 1
            reason: Missing output files: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html
            threads: 2
            resources: tmpdir=/dev/shm/jobs/40901556


        [Mon Nov 13 03:58:29 2023]
        localrule all:
            input: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 0
            reason: Input files updated by another job: ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html
            resources: tmpdir=/dev/shm/jobs/40901556

        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2

        Reasons:
            (check individual jobs above for details)
            input files updated by another job:
                all
            missing output files:
                fastqc

        This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
        ```

<br>

!!! terminal-2 "Let's do a full run"

    ```diff
    - snakemake --cores 2
    + snakemake --cores 2 --use-envmodules
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Using shell: /usr/bin/bash
        Provided cores: 2
        Rules claiming more threads will be scaled down.
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2

        Select jobs to execute...

        [Mon Nov 13 03:59:13 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 1
            reason: Missing output files: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html
            threads: 2
            resources: tmpdir=/dev/shm/jobs/40901556

        Activating environment modules: FastQC/0.12.1

        The following modules were not unloaded:
        (Use "module --force purge" to unload all):

        1) XALT/minimal   2) slurm   3) NeSI
        2) Started analysis of SRR2584863_1.trim.sub.fastq
        Approx 5% complete for SRR2584863_1.trim.sub.fastq
        Started analysis of SRR2584863_2.trim.sub.fastq
        Approx 10% complete for SRR2584863_1.trim.sub.fastq
        Approx 5% complete for SRR2584863_2.trim.sub.fastq
        Approx 15% complete for SRR2584863_1.trim.sub.fastq
        Approx 10% complete for SRR2584863_2.trim.sub.fastq
        Approx 20% complete for SRR2584863_1.trim.sub.fastq
        Approx 15% complete for SRR2584863_2.trim.sub.fastq
        Approx 25% complete for SRR2584863_1.trim.sub.fastq
        Approx 20% complete for SRR2584863_2.trim.sub.fastq
        Approx 30% complete for SRR2584863_1.trim.sub.fastq
        Approx 25% complete for SRR2584863_2.trim.sub.fastq
        Approx 35% complete for SRR2584863_1.trim.sub.fastq
        Approx 30% complete for SRR2584863_2.trim.sub.fastq
        Approx 40% complete for SRR2584863_1.trim.sub.fastq
        Approx 35% complete for SRR2584863_2.trim.sub.fastq
        Approx 45% complete for SRR2584863_1.trim.sub.fastq
        Approx 40% complete for SRR2584863_2.trim.sub.fastq
        Approx 50% complete for SRR2584863_1.trim.sub.fastq
        Approx 45% complete for SRR2584863_2.trim.sub.fastq
        Approx 55% complete for SRR2584863_1.trim.sub.fastq
        Approx 50% complete for SRR2584863_2.trim.sub.fastq
        Approx 55% complete for SRR2584863_2.trim.sub.fastq
        Approx 60% complete for SRR2584863_1.trim.sub.fastq
        Approx 60% complete for SRR2584863_2.trim.sub.fastq
        Approx 65% complete for SRR2584863_1.trim.sub.fastq
        Approx 65% complete for SRR2584863_2.trim.sub.fastq
        Approx 70% complete for SRR2584863_1.trim.sub.fastq
        Approx 70% complete for SRR2584863_2.trim.sub.fastq
        Approx 75% complete for SRR2584863_1.trim.sub.fastq
        Approx 75% complete for SRR2584863_2.trim.sub.fastq
        Approx 80% complete for SRR2584863_1.trim.sub.fastq
        Approx 80% complete for SRR2584863_2.trim.sub.fastq
        Approx 85% complete for SRR2584863_1.trim.sub.fastq
        Approx 85% complete for SRR2584863_2.trim.sub.fastq
        Approx 90% complete for SRR2584863_1.trim.sub.fastq
        Approx 90% complete for SRR2584863_2.trim.sub.fastq
        Approx 95% complete for SRR2584863_1.trim.sub.fastq
        Approx 95% complete for SRR2584863_2.trim.sub.fastq
        Approx 100% complete for SRR2584863_2.trim.sub.fastq
        Approx 100% complete for SRR2584863_1.trim.sub.fastq
        [Mon Nov 13 03:59:21 2023]
        Finished job 1.
        1 of 2 steps (50%) done
        Select jobs to execute...

        [Mon Nov 13 03:59:21 2023]
        localrule all:
            input: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 0
            reason: Input files updated by another job: ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html
            resources: tmpdir=/dev/shm/jobs/40901556

        [Mon Nov 13 03:59:21 2023]
        Finished job 0.
        2 of 2 steps (100%) done
        Complete log: .snakemake/log/2023-11-13T035913.003859.snakemake.log
        ```

<br>

Notice it now says that "Activating environment modules: FastQC/0.12.1". Now the software our workflow uses will be automatically loaded!

## 3.10 Capture our logs

So far our logs (for fastqc) have been simply printed to our screen. As you can imagine, if you had a large automated workflow (that you might not be sitting at the computer watching run) you'll want to capture all that information. Therefore, any information the software spits out (including error messages!) will be kept and can be looked at once you return to your machine from your coffee break.

!!! info "We can get the logs for each rule to be written to a log file via the `log:` directive:"

    - It's a good idea to organise the logs by:
      - Putting the logs in a directory labelled after the rule/software that was run
      - Labelling the log files with the sample name the software was run on

    - Also make sure you tell the software (fastqc) to write the standard output and standard error to this log file we defined in the `log:` directive in the shell script (eg. `&> {log}`)

??? code-compare "Edit snakefile"

    ```diff
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
    +    log:
    +        "logs/fastqc/SRR2584863.log"
        threads: 2
        envmodules:
            "FastQC/0.12.1"
        shell:
    -        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    +        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
    ```

??? file-code "Current snakefile"

    ```python
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
            "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
            "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/SRR2584863.log"
        threads: 2
        envmodules:
            "FastQC/0.12.1"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
    ```

<br>

---

A tangent about [standard streams](https://en.wikipedia.org/wiki/Standard_streams)

- These are standard streams in which information is returned by a computer process - in our case the logs that we see returned to us on our screen when we run fastqc
- There are two main streams:
  - standard output (the log messages)
  - standard error (the error messages)

Different ways to write log files:

| Syntax | standard output in terminal | standard error in terminal | standard output in file | standard error in file |
| ------ | --------------------------- | -------------------------- | ----------------------- | ---------------------- |
| `>`    | NO                          | YES                        | YES                     | NO                     |
| `2>`   | YES                         | NO                         | NO                      | YES                    |
| `&>`   | NO                          | NO                         | YES                     | YES                    |

(Table adapted from [here](https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file))

---

!!! terminal-2 "Run again"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --cores 2 --use-envmodules
    snakemake --cores 2 --use-envmodules
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Using shell: /usr/bin/bash
        Provided cores: 2
        Rules claiming more threads will be scaled down.
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        1
        total         2

        Select jobs to execute...

        [Mon Nov 13 04:13:31 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2584863.log
            jobid: 1
            reason: Missing output files: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip; Code has changed since last execution
            threads: 2
            resources: tmpdir=/dev/shm/jobs/40901556

        Activating environment modules: FastQC/0.12.1

        The following modules were not unloaded:
        (Use "module --force purge" to unload all):

        1) XALT/minimal   2) slurm   3) NeSI
        [Mon Nov 13 04:13:41 2023]
        Finished job 1.
        1 of 2 steps (50%) done
        Select jobs to execute...

        [Mon Nov 13 04:13:41 2023]
        localrule all:
            input: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            jobid: 0
            reason: Input files updated by another job: ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip
            resources: tmpdir=/dev/shm/jobs/40901556

        [Mon Nov 13 04:13:41 2023]
        Finished job 0.
        2 of 2 steps (100%) done
        Complete log: .snakemake/log/2023-11-13T041331.085160.snakemake.log
        ```

<br>

!!! terminal-2 "We now have a log file, lets have a look at the first 10 lines of our log with:"

    ```bash
    head ./logs/fastqc/SRR2584863.log
    ```

    ??? success "output"

        ```bash
        Started analysis of SRR2584863_1.trim.sub.fastq
        Approx 5% complete for SRR2584863_1.trim.sub.fastq
        Approx 10% complete for SRR2584863_1.trim.sub.fastq
        Approx 5% complete for SRR2584863_2.trim.sub.fastq
        Approx 15% complete for SRR2584863_1.trim.sub.fastq
        Approx 10% complete for SRR2584863_2.trim.sub.fastq
        Approx 20% complete for SRR2584863_1.trim.sub.fastq
        ```

<br>

<p align="center"><b>We have logs. Tidy logs.</b><br></p>

<center>![logs](https://miro.medium.com/max/2560/1*ohWUB5snJRaMe-vJ8HaoiA.png){width="400"}</center>

!!! question "Exercise:"

    Try creating an error in the shell command (for example remove the `-o` flag) and use the three different syntaxes for writing to your log file. What is and isn't printed to your screen and to your log file?

## 3.11 Scale up to analyse all of our samples

We are currently only analysing one of our three samples

Let's scale up to run all of our samples by using [wildcards](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#wildcards), this way we can grab all the samples/files in the `data` directory and analyse them

- Set a global wildcard that defines the samples to be analysed
- Generalise where this rule uses an individual sample (`SRR2584863`) to use this wildcard `{sample}`
- Use the [expand function](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#the-expand-function) (`expand()`) function to tell snakemake that `{sample}` is what we defined in our global wildcard `SAMPLES,`
- Snakemake can figure out what `{sample}` is in our rule since it's defined in the targets in `rule all:`

??? code-compare "Edit snakefile"

    ```diff
    # define samples from data directory using wildcards
    + SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
    -       "../results/fastqc/SRR2584863_1.trim.sub_fastqc.html",
    -       "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html",
    -       "../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip",
    -       "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"
    +       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
    +       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
    +       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
    +       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES)

    # workflow
    rule fastqc:
        input:
    -       R1 = "../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq",
    -       R2 = "../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq"
    +       R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
    +       R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
    -       html = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.html", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.html"],
    -       zip = ["../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip", "../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip"]
    +       html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
    +       zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
    -       "logs/fastqc/SRR25848631.log"
    +       "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES)

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
    ```

<br>

!!! terminal-2 "Visualise workflow"

    ```bash
    snakemake --dag | dot -Tpng > dag_3.png
    ```

    - Now we have three samples running though our workflow, one of which has already been run in our last run (SRR25848631) indicated by the dashed lines

    ??? image "DAG"

        <center>![DAG_3](./images/dag_3.png)</center>

<br>

!!! terminal-2 "Run workflow again"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun again
    snakemake --dryrun --cores 2 --use-envmodules
    ```

    - See how it now runs over all three of our samples in the output of the dryrun

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        3
        total         4


        [Mon Nov 20 22:58:45 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.      sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2584863.log
            jobid: 3
            reason: Missing output files: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/      SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            wildcards: sample=SRR2584863
            threads: 2
            resources: tmpdir=/dev/shm/jobs/41187219


        [Mon Nov 20 22:58:45 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2589044_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2589044_2.trim.sub.fastq
            output: ../results/fastqc/SRR2589044_1.trim.sub_fastqc.html, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.html, ../results/fastqc/SRR2589044_1.trim.      sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2589044.log
            jobid: 1
            reason: Missing output files: ../results/fastqc/SRR2589044_2.trim.sub_fastqc.html, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip, ../results/fastqc/       SRR2589044_1.trim.sub_fastqc.html, ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip
            wildcards: sample=SRR2589044
            threads: 2
            resources: tmpdir=/dev/shm/jobs/41187219


        [Mon Nov 20 22:58:45 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584866_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584866_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584866_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_1.trim.      sub_fastqc.zip, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2584866.log
            jobid: 2
            reason: Missing output files: ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip, ../results/fastqc/        SRR2584866_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.html
            wildcards: sample=SRR2584866
            threads: 2
            resources: tmpdir=/dev/shm/jobs/41187219


        [Mon Nov 20 22:58:45 2023]
        localrule all:
            input: ../results/fastqc/SRR2589044_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.       sub_fastqc.html, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.  trim.sub_fastqc.html, ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.        trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.        trim.sub_fastqc.zip
            jobid: 0
            reason: Input files updated by another job: ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_1.trim.sub_fastqc.html, ../        results/fastqc/SRR2584866_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../       results/fastqc/SRR2584866_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.  zip, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.        html, ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip
            resources: tmpdir=/dev/shm/jobs/41187219

        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        3
        total         4

        Reasons:
            (check individual jobs above for details)
            input files updated by another job:
                all
            missing output files:
                fastqc

        This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
        ```

<br>

!!! terminal "code"

    ```bash
    # full run again
    snakemake --cores 2 --use-envmodules
    ```

    - All three samples were run through our workflow! And we have a log file for each sample for the fastqc rule

    ```bash
    ls -lh ./logs/fastqc
    ```

    ??? success "output"
        ```bash
        ls -l ./logs/fastqc/
        total 2
        -rw-rw----+ 1 murray.cadzow nesi02659 2276 Nov 21 22:06 SRR2584863.log
        -rw-rw----+ 1 murray.cadzow nesi02659 2276 Nov 21 22:05 SRR2584866.log
        -rw-rw----+ 1 murray.cadzow nesi02659 2276 Nov 21 22:05 SRR2589044.log
        ```

<br>

## 3.12 Add more rules

- Connect the outputs of fastqc to the inputs of multiqc
- Add a new final target for `rule all:`

??? code-compare "Edit snakefile"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES)
    +       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES),
    +       "../results/multiqc_report.html"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    + rule multiqc:
    +   input:
    +       expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
    +   output:
    +       "../results/multiqc_report.html"
    +   log:
    +       "logs/multiqc/multiqc.log"
    +   envmodules:
    +       "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    +   shell:
    +       "multiqc {input} -o ../results/ &> {log}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES),
            "../results/multiqc_report.html"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
           "logs/multiqc/multiqc.log"
        envmodules:
           "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    ```

<br>

!!! terminal-2 "Run workflow again"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --cores 2 --use-envmodules
    snakemake --cores 2 --use-envmodules
    ```

    - Visualise workflow
    ```bash
    snakemake --dag | dot -Tpng > dag_4.png
    ```

    - Now we have two rules in our workflow (fastqc and multiqc), we can also see that multiqc isn't run for each sample (since it merges the output of fastqc for all samples)

    ??? image "DAG:"

        ![DAG_4](./images/dag_4.png)

<br>

## 3.13 More about Snakemake's lazy behaviour

What happens if we only have the final target file (`../results/multiqc_report.html`) in `rule all:`

??? code-compare "Edit snakefile"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
    -       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
            "../results/multiqc_report.html"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
           "logs/multiqc/multiqc.log"
        envmodules:
           "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
           "logs/multiqc/multiqc.log"
        envmodules:
           "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    ```

<br>

!!! terminal-2 "Run workflow again"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun again
    snakemake --dryrun --cores 2 --use-envmodules
    ```

    - It still works because it is the last file in the workflow sequence, Snakemake will do all the steps necessary to get to this target file (therefore it runs both fastqc and multiqc)

    - Visualise workflow
      ```bash
      snakemake --dag | dot -Tpng > dag_5.png
      ```

    - Although the workflow ran the same, the DAG actually changed slightly, now there is only one file target and only the output of multiqc goes to `rule all`

    ??? image "DAG"

        ![DAG_5](./images/dag_5.png)

<br>

!!! warning "Beware: Snakemake will also NOT run rules that it doesn't need to run in order to get the target files defined in rule: all"

For example if only our fastqc outputs are defined as the target in `rule: all`

??? code-compare "Edit snakefile"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
        +   expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
        +   expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
        +   expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
        +   expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES)
        -   "../results/multiqc_report.html"

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
           "logs/multiqc/multiqc.log"
        envmodules:
           "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
            expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
            expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES)


    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
           "logs/multiqc/multiqc.log"
        envmodules:
           "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    ```

<br>

!!! terminal-2 "Run again"

    ```bash
    # run dryrun again
    snakemake --dryrun --cores 2 --use-envmodules
    ```

    !!! success "My partial output:"

        ```bash
        Job stats:
        job       count
        ------  -------
        all           1
        fastqc        3
        total         4
        ```

<br>

!!! terminal-2 "Our multiqc rule won't be run/evaluated : Visualise workflow"

    ```bash
    snakemake --dag | dot -Tpng > dag_6.png
    ```

    - Now we are back to only running fastqc in our workflow, despite having our second rule (multiqc) in our workflow

    ??? image "DAG:"

        ![DAG_6](./images/dag_6.png)

<br>

<p align="center"><b>Snakemake is lazy.</b><br></p>

![Snakemake is lazy.](https://64.media.tumblr.com/0492923adeb79cb841e29968135305d5/tumblr_nzdagaL6EH1uavdlbo7_1280.png)

## 3.14 Add even more rules

Let's add the rest of the rules. We want to get to:

<center>![rulegraph_1](./images/rulegraph_1.png)</center>

We currently have fastqc and multiqc, so we still need to add trim_galore

??? code-compare "Edit snakefile"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
    -       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.html", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.html", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_1.trim.sub_fastqc.zip", sample = SAMPLES),
    -       expand("../results/fastqc/{sample}_2.trim.sub_fastqc.zip", sample = SAMPLES),
    +       "../results/multiqc_report.html",
    +       expand("../results/sam/{sample}.sam", sample = SAMPLES)

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"

    +  rule bwa_align:
    +    input:
    +        R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
    +        R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
    +        genome = "../../data/ref_genome/ecoli_rel606.fasta"
    +    output:
    +        SAM = "../results/sam/{sample}.sam"
    +    log: "logs/bwa/{sample}.bwa.log"
    +    threads: 2
    +    envmodules:
    +        "BWA/0.7.17-gimkl-2017a"
    +    shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/sam/{sample}.sam", sample = SAMPLES)

    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"

    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ &> {log}"

    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"
    ```

<br>

!!! terminal-2 "Visualise workflow"

    ```bash
    snakemake --dag | dot -Tpng > dag_7.png
    ```

    Fantastic, we are starting to build a workflow!

    ??? image "DAG:"

        ![DAG_7](./images/dag_7.png)

<br>

However, when analysing many samples, our DAG can become messy and complicated. Instead, we can create a rulegraph that will let us visualise our workflow without showing every single sample that will run through it

!!! terminal "code"

    ```bash
    snakemake --rulegraph | dot -Tpng > rulegraph_1.png
    ```

    !!! image "My rulegraph:"

        ![rulegraph_1](./images/rulegraph_1.png)

<br>

!!! terminal-2 "An aside: another option that will show all your input and output files at each step:"

    ```bash
    snakemake --filegraph | dot -Tpng > filegraph.png
    ```

    ??? image "My filegraph:"

        ![filegraph](./images/filegraph.png)

<br>


Run the rest of the workflow

!!! terminal "code"

    ```bash
    # run dryrun/run again
    snakemake --dryrun --cores 2 --use-envmodules
    snakemake --cores 2 --use-envmodules
    ```

!!! question "Notice it will run only one rule/sample/file at a time...why is that?"

## 3.15 Throw it more cores

Run again allowing Snakemake to use more cores overall `--cores 4` rather than `--cores 2`

!!! terminal "code"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --cores 4 --use-envmodules
    snakemake --cores 4 --use-envmodules
    ```

Notice the whole workflow ran much faster and several samples/files/rules were running at one time. This is because we set each rule to run with 2 threads. Initially we specified that the _maximum_ number of cores to be used by the workflow was 2 with the `--cores 2` flag, meaning only one rule and sample can be run at one time. When we increased the _maximum_ number of cores to be used by the workflow to 4 with `--cores 4`, up to 2 samples could be run through at one time.

## 3.16 Throw it even more cores

With a high performance cluster such as [NeSi](https://www.nesi.org.nz/), you can start to REALLY scale up, particularly when you have many samples to analyse or files to process. This is because the number of cores available in a HPC is HUGE compared to a laptop or even an high end server.

<p align="center"><b>Boom! Scalability here we come!</b><br></p>

<center>![image](./nesi_images/mahuika_maui_real.png){width="800"}</center>

To run the workflow on the cluster, we need to ensure that each step is run as a dedicated job in the queuing system of the HPC. On NeSI, the queuing system is managed by [Slurm](https://slurm.schedmd.com/documentation.html).

Use the `--cluster` option to specify the job submission command, using `sbatch` on NeSI.
This command defines resources used for each job (maximum time, memory, number of cores...).
In addition, you need to specify a maximum number of concurrent jobs using `--jobs`.

!!! terminal "code"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run again on the cluster
    snakemake --cluster "sbatch --time 00:10:00 --mem 512MB --cpus-per-task 8 --account nesi02659" --jobs 10 --use-envmodules
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Using shell: /usr/bin/bash
        Provided cluster nodes: 10
        Job stats:
        job          count
        ---------  -------
        all              1
        bwa_align        3
        fastqc           3
        multiqc          1
        total            8

        Select jobs to execute...

        [Sun Nov 26 02:45:58 2023]
        rule bwa_align:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq, ../../data/ref_genome/ecoli_rel606.fasta
            output: ../results/sam/SRR2584863.sam
            log: logs/bwa/SRR2584863.bwa.log
            jobid: 7
            reason: Missing output files: ../results/sam/SRR2584863.sam
            wildcards: sample=SRR2584863
            threads: 2
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 7 with external jobid 'Submitted batch job 41492185'.

        [Sun Nov 26 02:45:58 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2589044_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2589044_2.trim.sub.fastq
            output: ../results/fastqc/SRR2589044_1.trim.sub_fastqc.html, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.html, ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2589044.log
            jobid: 2
            reason: Missing output files: ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip
            wildcards: sample=SRR2589044
            threads: 2
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 2 with external jobid 'Submitted batch job 41492186'.

        [Sun Nov 26 02:45:58 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584866_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584866_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584866_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2584866.log
            jobid: 3
            reason: Missing output files: ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip
            wildcards: sample=SRR2584866
            threads: 2
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 3 with external jobid 'Submitted batch job 41492187'.

        [Sun Nov 26 02:45:58 2023]
        rule bwa_align:
            input: ../../data/trimmed_fastq_small/SRR2589044_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2589044_2.trim.sub.fastq, ../../data/ref_genome/ecoli_rel606.fasta
            output: ../results/sam/SRR2589044.sam
            log: logs/bwa/SRR2589044.bwa.log
            jobid: 5
            reason: Missing output files: ../results/sam/SRR2589044.sam
            wildcards: sample=SRR2589044
            threads: 2
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 5 with external jobid 'Submitted batch job 41492188'.

        [Sun Nov 26 02:45:58 2023]
        rule bwa_align:
            input: ../../data/trimmed_fastq_small/SRR2584866_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584866_2.trim.sub.fastq, ../../data/ref_genome/ecoli_rel606.fasta
            output: ../results/sam/SRR2584866.sam
            log: logs/bwa/SRR2584866.bwa.log
            jobid: 6
            reason: Missing output files: ../results/sam/SRR2584866.sam
            wildcards: sample=SRR2584866
            threads: 2
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 6 with external jobid 'Submitted batch job 41492189'.

        [Sun Nov 26 02:45:58 2023]
        rule fastqc:
            input: ../../data/trimmed_fastq_small/SRR2584863_1.trim.sub.fastq, ../../data/trimmed_fastq_small/SRR2584863_2.trim.sub.fastq
            output: ../results/fastqc/SRR2584863_1.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.html, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            log: logs/fastqc/SRR2584863.log
            jobid: 4
            reason: Missing output files: ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip
            wildcards: sample=SRR2584863
            threads: 2
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 4 with external jobid 'Submitted batch job 41492190'.
        [Sun Nov 26 02:46:18 2023]
        Finished job 7.
        1 of 8 steps (12%) done
        [Sun Nov 26 02:46:18 2023]
        Finished job 2.
        2 of 8 steps (25%) done
        [Sun Nov 26 02:46:18 2023]
        Finished job 3.
        3 of 8 steps (38%) done
        [Sun Nov 26 02:46:18 2023]
        Finished job 5.
        4 of 8 steps (50%) done
        [Sun Nov 26 02:46:18 2023]
        Finished job 6.
        5 of 8 steps (62%) done
        [Sun Nov 26 02:46:18 2023]
        Finished job 4.
        6 of 8 steps (75%) done
        Select jobs to execute...

        [Sun Nov 26 02:46:18 2023]
        rule multiqc:
            input: ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip
            output: ../results/multiqc_report.html
            log: logs/multiqc/multiqc.log
            jobid: 1
            reason: Missing output files: ../results/multiqc_report.html; Input files updated by another job: ../results/fastqc/SRR2584863_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_2.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584863_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2584866_1.trim.sub_fastqc.zip, ../results/fastqc/SRR2589044_2.trim.sub_fastqc.zip
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=<TBD>

        Submitted job 1 with external jobid 'Submitted batch job 41492195'.
        [Sun Nov 26 02:46:48 2023]
        Finished job 1.
        7 of 8 steps (88%) done
        Select jobs to execute...

        [Sun Nov 26 02:46:48 2023]
        localrule all:
            input: ../results/multiqc_report.html, ../results/sam/SRR2589044.sam, ../results/sam/SRR2584866.sam, ../results/sam/SRR2584863.sam
            jobid: 0
            reason: Input files updated by another job: ../results/sam/SRR2589044.sam, ../results/multiqc_report.html, ../results/sam/SRR2584863.sam, ../results/sam/SRR2584866.sam
            resources: mem_mb=1000, mem_mib=954, disk_mb=1000, disk_mib=954, tmpdir=/dev/shm/jobs/41486074

        [Sun Nov 26 02:46:48 2023]
        Finished job 0.
        8 of 8 steps (100%) done
        Complete log: .snakemake/log/2023-11-26T024558.566062.snakemake.log
        ```

<br>

If you open another terminal on the HPC, you can use the `squeue` command to list of your jobs and their state (pending, running, etc.):

!!! terminal "code"

    ```bash
    squeue --me
    ```

    ??? success "output"


        ```bash
        JOBID         USER     ACCOUNT   NAME        CPUS MIN_MEM PARTITI START_TIME     TIME_LEFT STATE    NODELIST(REASON)    
        41486074      murray.c nesi02659 spawner-jupy   2      4G interac 2023-11-25T2     3:25:34 RUNNING  wbn001              
        41492185      murray.c nesi02659 snakejob.bwa   8    512M large   2023-11-26T0        9:56 RUNNING  wbn010              
        41492186      murray.c nesi02659 snakejob.fas   8    512M large   2023-11-26T0        9:56 RUNNING  wbn010              
        41492187      murray.c nesi02659 snakejob.fas   8    512M large   2023-11-26T0        9:56 RUNNING  wbn063              
        41492188      murray.c nesi02659 snakejob.bwa   8    512M large   2023-11-26T0        9:56 RUNNING  wbn063              
        41492189      murray.c nesi02659 snakejob.bwa   8    512M large   2023-11-26T0        9:56 RUNNING  wbn063              
        41492190      murray.c nesi02659 snakejob.fas   8    512M large   2023-11-26T0        9:56 RUNNING  wbn063     
        ```

<br>

An additional trick is to use the `watch` command to repeatedly call any command in the terminal, giving you a lightweight monitoring tool ;-).
Here we will use it to see your jobs gets queued and executed in real time:

!!! terminal "code"

    ```bash
    watch squeue --me
    ```

You can exit the view create by `watch` by pressing CTRL+C.

# Takeaways

---

!!! quote ""

    - Once familiar with environment modules, the software are very straightforward to integrate in your snakemake workflow
    - Run your commands directly on the command line before wrapping it up in a Snakemake rule
    - First do a dryrun to check the Snakemake structure is set up correctly
    - Work iteratively (get each rule working before moving onto the next)
    - File paths are relative to the Snakefile
    - Run your workflow from where your Snakefile is
    - Visualise your workflow by creating a DAG (directed acyclic graph), a rulegraph or filegraph
    - Use environment modules to load software in your workflow - this improves reproducibility
    - Snakemake is lazy...
      - It will only do something if it hasn't already done it
      - It will pick up where it left off, rather than run the whole workflow again
      - It *won't* do any steps that aren't necessary to get to the target files defined in `rule: all`
    - `input:` `output:` `log:` and `threads:` directives need to be called in the `shell` directive
    - Capture your log files
    - Organise your log files by naming them after the rule that was run and sample that was analysed
    - You don't need to specify all the target files in `rule all:`, the final file in a given chain of tasks will suffice
    - We can massively speed up our analyses by running our samples in parallel

---

# Summary commands

!!! terminal "code"

    - Create a directed acyclic graph (DAG) with:

    ```bash
    snakemake --dag | dot -Tpng > dag.png
    ```

    - Create a rulegraph with:

    ```bash
    snakemake --rulegraph | dot -Tpng > rulegraph.png
    ```

    - Create a filegraph with:

    ```bash
    snakemake --filegraph | dot -Tpng > filegraph.png
    ```

    - Run a dryrun of your snakemake workflow with:

    ```bash
    snakemake --dryrun
    ```

    - Run your snakemake workflow with:

    ```bash
    snakemake --cores 2
    ```

    - Run a dryrun of your snakemake workflow (using environment modules to load your software) with:

    ```bash
    snakemake --dryrun --cores 2 --use-envmodules
    ```

    - Run your snakemake workflow (using environment modules to load your software) with:

    ```bash
    snakemake --cores 2 --use-envmodules
    ```

    - Run your snakemake workflow using multiple jobs on NeSI:

    ```bash
    snakemake --cluster "sbatch --time 00:10:00 --mem=512MB --cpus-per-task 8" --jobs 10 --use-envmodules
    ```

    - Create a global wildcard to get process all your samples in a directory with:

    ```bash
    SAMPLES, = glob_wildcards("../relative/path/to/samples/{sample}_1.fastq.gz")
    ```

    - Combine this with the expand function to tell Snakemake to look at your global wildcard to figure out what you refer to as `{sample}` in your workflow

    ```bash
    expand("../results/{sample}_1.fastq.gz", sample = SAMPLES)
    ```

    - Increase the number of samples that can be analysed at one time in your workflow by increasing the maximum number of cores to be used at one time with the `--cores` command

    ```bash
    snakemake --cores 4 --use-envmodules
    ```

# Our final snakemake workflow!

See [basic_demo_workflow](https://github.com/otagobioinformaticsspringschool/snakemake_workshop/blob/main/basic_demo_workflow/workflow/) for the final Snakemake workflow we've created up to this point

---

<p align="center"><b><a class="btn" href="https://otagobioinformaticsspringschool.github.io/snakemake_workshop/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
````
