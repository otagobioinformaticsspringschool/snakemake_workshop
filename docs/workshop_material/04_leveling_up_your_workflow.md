# 04 - Leveling up your workflow!

## Catching up

!!! file-code "From section 03, you should have the following Snakefile:"

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

## 4.1 Use a profile for HPC

In section 3.16, we have seen that a snakemake workflow can be run on an HPC cluster.
To reduce the boilerplate, we can use a [configuration profile](https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles) to configure default options.
In this case, we use it to set the `--cluster` and the `--jobs` options.

!!! terminal-2 "Make a `slurm` profile folder"

    ```bash
    # create the profile folder
    mkdir slurm
    touch slurm/config.yaml
    ```
!!! file-code "write the following to config.yaml"
    ```bash
    jobs: 20
    cluster: "sbatch --time 00:10:00 --mem 512MB --cpus-per-task 8 --account nesi02659"
    ```

!!! terminal-2 "Then run the snakemake workflow using the `slurm` profile"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --profile slurm --use-envmodules
    ```
    ```bash
    snakemake --profile slurm --use-envmodules
    ```

If you interrupt the execution of a snakemake workflow using <KBD>CTRL+C</KBD>, already submitted Slurm jobs won't be cancelled.
We tell snakemake how to cancel Slurm jobs using `scancel` via the `--cluster-cancel` option and adding `--parsable` to the `sbatch` command, to make it return the job ID.

!!! code-compare "Edit config.yaml"
    ```diff
    jobs: 20
    - cluster: "sbatch --time 00:10:00 --mem 512MB --cpus-per-task 8"
    + cluster: "sbatch --parsable --time 00:10:00 --mem 512MB --cpus-per-task 8"
    + cluster-cancel: scancel
    ```

You can specify different resources (memory, cpus, gpus, etc.) for each target in the workflow and refer to them in the `cluster` option using placeholders.
Default resources for all rules can also be set using the `default-resources` option.

Update the profile `slurm/config.yaml` file as follows (using a multiline option to improve readability)

!!! code-compare "Edit config.yml"

    ```diff
    jobs: 20
    - cluster: "sbatch --parsable --time 00:10:00 --mem 512MB --cpus-per-task 8"
    + cluster:
    +     sbatch
    +         --parsable
    +         --time {resources.time_min}
    +         --mem {resources.mem_mb}
    +         --cpus-per-task {resources.cpus}
    +         --account nesi02659
    + default-resources: [cpus=2, mem_mb=512, time_min=10]
    cluster-cancel: scancel
    ```

and add resources definitions in the workflow.
Here we give more CPU resources to `trim_galore` to make it run faster.

??? code-compare "Edit snakefile"

    ```diff
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
    +   resources:
    +       cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"
    ```
    
??? file-code "Current slurm profile:"

    ```
    jobs: 20
    cluster:
        sbatch
            --parsable
            --time {resources.time_min}
            --mem {resources.mem_mb}
            --cpus-per-task {resources.cpus}
            --account nesi02659
    default-resources: [cpus=2, mem_mb=512, time_min=10]
    cluster-cancel: scancel
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
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"
    ```
    
<br>

!!! terminal-2 "Run the workflow again"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```
    # run dryrun/run again
    snakemake --dryrun --profile slurm --use-envmodules
    ```
    ```bash
    snakemake --profile slurm --use-envmodules
    ```

    - If you monitor the progress of your jobs using `squeue`, you will notice that some jobs now request 2 or 8 CPUs.

    ??? success "output"

        ```
        JOBID         USER     ACCOUNT   NAME        CPUS MIN_MEM PARTITI START_TIME     TIME_LEFT STATE    NODELIST(REASON)
        26763281      lkemp    nesi02659 spawner-jupy   4      4G interac 2022-05-11T1     7:21:18 RUNNING  wbn003
        26763492      lkemp    nesi02659 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn144
        26763493      lkemp    nesi02659 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn212
        26763494      lkemp    nesi02659 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn145
        26763495      lkemp    nesi02659 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn146
        26763496      lkemp    nesi02659 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn217
        26763497      lkemp    nesi02659 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn229
        ```
    
<br>

Now looking at the content of our workflow folder, it is getting cluttered with Slurm log files:

!!! terminal "code"

    ```bash
    ls -lh
    ```

    ??? success "output"

        ```bash
        total 1.8M
        -rw-rw----+ 1 lkemp nesi02659 4.2K May 11 12:10 dag_1.png
        -rw-rw----+ 1 lkemp nesi02659 3.8K May 11 12:13 dag_2.png
        -rw-rw----+ 1 lkemp nesi02659  12K May 11 12:16 dag_3.png
        -rw-rw----+ 1 lkemp nesi02659  20K May 11 12:19 dag_4.png
        -rw-rw----+ 1 lkemp nesi02659  15K May 11 12:21 dag_5.png
        -rw-rw----+ 1 lkemp nesi02659  12K May 11 12:23 dag_6.png
        -rw-rw----+ 1 lkemp nesi02659  26K May 11 12:24 dag_7.png
        drwxrws---+ 5 lkemp nesi02659 4.0K May 11 12:25 logs
        -rw-rw----+ 1 lkemp nesi02659  11K May 11 12:24 rulegraph_1.png
        drwxrws---+ 3 lkemp nesi02659 4.0K May 11 12:34 slurm
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:27 slurm-26763403.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:27 slurm-26763404.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:27 slurm-26763405.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:27 slurm-26763406.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:27 slurm-26763407.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:27 slurm-26763408.out
        -rw-rw----+ 1 lkemp nesi02659  865 May 11 12:29 slurm-26763409.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:30 slurm-26763418.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:30 slurm-26763419.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:30 slurm-26763420.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:30 slurm-26763421.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:30 slurm-26763422.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:30 slurm-26763423.out
        -rw-rw----+ 1 lkemp nesi02659  865 May 11 12:32 slurm-26763431.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:33 slurm-26763435.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:34 slurm-26763436.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:33 slurm-26763437.out
        -rw-rw----+ 1 lkemp nesi02659  837 May 11 12:34 slurm-26763438.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:34 slurm-26763439.out
        -rw-rw----+ 1 lkemp nesi02659  809 May 11 12:34 slurm-26763440.out
        -rw-rw----+ 1 lkemp nesi02659  865 May 11 12:35 slurm-26763444.out
        -rw-rw----+ 1 lkemp nesi02659  857 May 11 12:36 slurm-26763447.out
        -rw-rw----+ 1 lkemp nesi02659  829 May 11 12:36 slurm-26763448.out
        -rw-rw----+ 1 lkemp nesi02659  857 May 11 12:36 slurm-26763449.out
        -rw-rw----+ 1 lkemp nesi02659  857 May 11 12:36 slurm-26763450.out
        -rw-rw----+ 1 lkemp nesi02659  829 May 11 12:36 slurm-26763451.out
        -rw-rw----+ 1 lkemp nesi02659  829 May 11 12:36 slurm-26763452.out
        -rw-rw----+ 1 lkemp nesi02659  885 May 11 12:37 slurm-26763454.out
        -rw-rw----+ 1 lkemp nesi02659 1.8K May 11 12:34 Snakefile
        ```
        
<br>

Let's clean this and create a dedicated folder `logs/slurm` for future log files:

!!! terminal "code"

    ```bash
    # remove slurm log files
    rm *.out
    ```
    ```bash
    # create a new folder for Slurm log files
    mkdir logs/slurm
    ```

!!! code-compare "then instruct Slurm to save its log files in it, in the profile `slurm/config.yaml` file"

    ```diff
    jobs: 20
    cluster:
        sbatch
            --parsable
            --time {resources.time_min}
            --mem {resources.mem_mb}
            --cpus-per-task {resources.cpus}
    +       --output logs/slurm/slurm-%j-{rule}.out
            --account nesi02659
    default-resources: [cpus=2, mem_mb=512, time_min=10]
    cluster-cancel: scancel
    ```

Note that `logs/slurm/slurm-%j-{rule}.out` contains a placeholder `{rule}`, which will be replaced by the name of the rule during the execution of the workflow.

Finally, to improve the communication between Snakemake and Slurm, we meed an additional script translating Slurm job status for Snakemake.
The `--cluster-status` option is used to tell Snakemake which script to use.

!!! terminal-2 "Create an executable `status.py` file"

    ```bash
    # create an empty file
    touch status.py
    ```
    ```bash
    # make it executable
    chmod +x status.py
    ```
    
    - and copy the following content in it
    
    ```python
    #!/usr/bin/env python
    import subprocess
    import sys
    
    jobid = sys.argv[1]
    
    output = str(subprocess.check_output("sacct -j %s --format State --noheader | head -1 | awk '{print $1}'" % jobid, shell=True).strip())
    
    running_status=["PENDING", "CONFIGURING", "COMPLETING", "RUNNING", "SUSPENDED"]
    if "COMPLETED" in output:
        print("success")
    elif any(r in output for r in running_status):
        print("running")
    else:
        print("failed")
    ```

??? code-compare "Then modify the profile `slurm/config.yaml` file"

    ```diff
    jobs: 20
    cluster:
        sbatch
            --parsable
            --time {resources.time_min}
            --mem {resources.mem_mb}
            --cpus-per-task {resources.cpus}
            --output logs/slurm/slurm-%j-{rule}.out
            --account nesi02659
    default-resources: [cpus=2, mem_mb=512, time_min=10]
    cluster-cancel: scancel
    + cluster-status: ./status.py
    ```

??? file-code "Current slurm profile:"

    ```
    jobs: 20
    cluster:
        sbatch
            --parsable
            --time {resources.time_min}
            --mem {resources.mem_mb}
            --cpus-per-task {resources.cpus}
            --output logs/slurm/slurm-%j-{rule}.out
            --account nesi02659
    default-resources: [cpus=2, mem_mb=512, time_min=10]
    cluster-cancel: scancel
    cluster-status: ./status.py
    ```

<br>

Once all of this is in place, we can:

!!! quote ""

    - submit Slurm jobs with the right resources per Snakemake rule,
    - cancel the workflow and Slurms jobs using CTRL-C,
    - keep all slurm jobs log files in a dedicated folder,
    - and make sure Snakemake reports Slurm jobs failures.

!!! question "Exercise"

    Run the snakemake workflow with Slurm jobs then use `scancel JOBID` to cancel some Slurm. See how Snakemake reacts with and without the `status.py` script.


## 4.2 Pull out parameters

??? code-compare "Edit snakefile"
    
    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
        -    expand("../results/sam/{sample}.sam", sample = SAMPLES)
        +    expand("../results/bam/{sample}.bam", sample = SAMPLES)

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
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    + rule sam_bam:
    +    input:
    +        SAM = "../results/sam/{sample}.sam"
    +    output:
    +        BAM = "../results/bam/{sample}.bam"
    +    params: "-S -b"
    +    log: "logs/samtools/{sample}.sam.log"
    +    threads: 2
    +    envmodules:
    +        "SAMtools/1.9-GCC-7.4.0"
    +    shell: "samtools view {params} {input} > {output}"
    ```
    
??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)

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
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        shell: "samtools view {params} {input} > {output}"
    ```

<br>

!!! terminal-2 "Run a dryrun to check it works"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```
    # run dryrun again
    snakemake --dryrun --profile slurm --use-envmodules
    ```

## 4.3 Pull out user configurable options

We can separate the user configurable options away from the workflow. This supports reproducibility by minimising the chance the user makes changes to the core workflow.

Create a configuration file in a new directory `config/`

File structure:

```bash
demo_workflow/
      |_______results/
      |_______workflow/
      |          |_______logs/
      |          |_______slurm/
      |          |_______Snakefile
      |          |_______status.py
      |_______config
                 |_______config.yaml
```
!!! terminal "code"

    ```bash
    # create config directory
    mkdir ../config
    ```
    ```bash
    # create configuration file
    touch ../config/config.yaml
    ```

Now we need to pull out the parameters the user would likely need to configure. Let's give the user the option to pass any parameters they like to fastqc. In our `../config/config.yaml` file, add the configuration options and add a couple flags to be passed to fastqc and multiqc:

```yaml
# set software parameters for...
PARAMS:
  # ... fastqc
  FASTQC: "--kmers 5"
  # ... multiqc
  MULTIQC: "--flat"
```

??? code-compare "Edit snakefile : In the Snakefile, tell Snakemake to grab the variables `PARAMS` from `../config/config.yaml`"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
    +   params:
    +       fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
    -       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
    +       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
    +   params:
    +       multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
    -       "multiqc {input} -o ../results/ &> {log}"
    +       "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        shell: "samtools view {params} {input} > {output}"
    ```
    
??? file-code "Current snakefile"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")

    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        shell: "samtools view {params} {input} > {output}"
    ```

<br>

!!! terminal-2 "Let's use our configuration file! Run workflow again:"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --profile slurm --use-envmodules
    ```
    ```bash
    snakemake --profile slurm --use-envmodules
    ```

    ??? failure "Didn't work? My error:"
    
        ```bash
        KeyError in line 19 of /scale_wlg_persistent/filesets/project/nesi02659/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile:
        'PARAMS'
          File "/scale_wlg_persistent/filesets/project/nesi02659/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile", line 19, in <module>
        ```

<br>

Snakemake can't find our 'Key' - we haven't told Snakemake where our config file is so it can't find our config variables. We can do this by passing the location of our config file to the `--configfile` flag

!!! terminal "code"

    ```bash
    # remove output of last run
    rm -r ../results/
    ```
    ```diff
    # run dryrun/run again
    - snakemake --dryrun --profile slurm --use-envmodules
    - snakemake --profile slurm --use-envmodules
    + snakemake --dryrun --profile slurm --use-envmodules --configfile ../config/config.yaml
    + snakemake --profile slurm --use-envmodules --configfile ../config/config.yaml
    ```

Alternatively, we can define our config file in our Snakefile in a situation where the configuration file is likely to always be named the same and be in the exact same location `../config/config.yaml` and you don't need the flexibility for the user to specify their own configuration files:

??? code-compare "Edit snakefile"

    ```diff
    # define our configuration file
    + configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        shell: "samtools view {params} {input} > {output}"
    ```
    
??? file-code "Current snakefile:"

    ```python
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        shell: "samtools view {params} {input} > {output}"
    ```

<br>

Then we don't need to specify where the configuration file is on the command line

!!! terminal "code"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```diff 
    # run dryrun/run again
    - snakemake --dryrun --profile slurm --use-envmodules --configfile ../config/config.yaml
    - snakemake --profile slurm --use-envmodules --configfile ../config/config.yaml
    + snakemake --dryrun --profile slurm --use-envmodules
    + snakemake --profile slurm --use-envmodules
    ```

## 4.4 Leave messages for the user

We can provide the user of our workflow more information on what is happening at each stage/rule of our workflow via the `message:` directive. We are able to call many variables such as:

- Input and output files `{input}` and `{output}`
- Specific input and output files such as `{input.R1}`
- Our `{params}`, `{log}` and `{threads}` directives

??? code-compare "Edit snakefile"

    ```diff
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
    +   message:
    +       "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    +   message:
    +       "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
    +   message:
    +       "Aligning reads to the reference for {wildcards.sample}"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
    +   message:
    +       "Converting sam: {input} to bam: {output}"
        shell: "samtools view {params} {input} > {output}"
    ```
    
??? file-code "Current snakefile:"

    ```python
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        message:
            "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        message:
            "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
            SAM = "../results/sam/{sample}.sam"
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        message:
            "Aligning reads to the reference for {wildcards.sample}"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        message:
            "Converting sam: {input} to bam: {output}"
        shell: "samtools view {params} {input} > {output}"
    ```
    
<br>

!!! terminal "code"

    ```diff
    # remove output of last run
    rm -r ../results/*
    ```

    ```bash
    # run dryrun/run again
    snakemake --dryrun --profile slurm --use-envmodules
    snakemake --profile slurm --use-envmodules
    ```
    
    - Now our messages are printed to the screen as our workflow runs

    ??? success "output"

        ```bash
        
        ```


<br>

## 4.5 Create temporary files

In our workflow, we are likely to be creating files that we don't want, but are used or produced by our workflow (intermediate files). We can mark such files as temporary so Snakemake will remove the file once it doesn't need to use it anymore.

For example, we might not want to keep our fastqc output files since our multiqc report merges all of our fastqc reports for each sample into one report. Let's have a look at the files currently produced by our workflow with:

!!! terminal "code"

    ```bash
    ls -lh ../results/fastqc/
    ```

    ??? success "output"

        ```bash
        
        ```

<br>

Let's mark all the trimmed fastq files as temporary in our Snakefile by wrapping it up in the `temp()` function

??? code-compare "Edit snakefile"
    
    ```diff
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        message:
            "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        message:
            "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
    -       SAM = "../results/sam/{sample}.sam"
    +       SAM = temp("../results/sam/{sample}.sam")
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        message:
            "Aligning reads to the reference for {wildcards.sample}"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        message:
            "Converting sam: {input} to bam: {output}"
        shell: "samtools view {params} {input} > {output}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand("../results/bam/{sample}.bam", sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq"
        output:
            html = ["../results/fastqc/{sample}_1.trim.sub_fastqc.html", "../results/fastqc/{sample}_2.trim.sub_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        envmodules:
            "FastQC/0.11.9"
        message:
            "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1.trim.sub_fastqc.zip", "../results/fastqc/{sample}_2.trim.sub_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        envmodules:
            "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
        message:
            "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule bwa_align:
        input:
            R1 = "../../data/trimmed_fastq_small/{sample}_1.trim.sub.fastq",
            R2 = "../../data/trimmed_fastq_small/{sample}_2.trim.sub.fastq",
            genome = "../../data/ref_genome/ecoli_rel606.fasta"
        output:
           SAM = temp("../results/sam/{sample}.sam")
        log: "logs/bwa/{sample}.bwa.log"
        threads: 2
        resources:
            cpus = 8
        envmodules:
            "BWA/0.7.17-gimkl-2017a"
        message:
            "Aligning reads to the reference for {wildcards.sample}"
        shell: "bwa mem -t {threads} {input.genome} {input.R1} {input.R2} > {output} 2> {log}"

    rule sam_bam:
        input:
            SAM = "../results/sam/{sample}.sam"
        output:
            BAM = "../results/bam/{sample}.bam"
        params: "-S -b"
        log: "logs/samtools/{sample}.sam.log"
        threads: 2
        envmodules:
            "SAMtools/1.9-GCC-7.4.0"
        message:
            "Converting sam: {input} to bam: {output}"
        shell: "samtools view {params} {input} > {output}"
    ```
    
    
<br>

!!! terminal "code"

    ```diff
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash    
    # run dryrun/run again
    snakemake --dryrun --profile slurm --use-envmodules
    snakemake --profile slurm --use-envmodules
    ```

!!! terminal-2 "Now when we have a look at the `../results/fastqc/` directory with:"

    ```bash
    ls -lh ../results/fastqc/
    ```

    - These html files have been removed once Snakemake no longer needs the files for another rule/operation, and we've saved some space on our computer (from 4.5 megabytes to 3 megabytes in this directory).

    ??? success "output"

        ```bash

        ```

<br>

!!! note ""

    This becomes particularly important when our data become big data, since we don't want to keep any massive intermediate output files that we don't need. Otherwise this can start to clog up the memory on our computer. It ensures our workflow is scalable when our data becomes big data.

## 4.6 Generating a snakemake report

With Snakemake, we can automatically generate detailed self-contained HTML reports after we run our workflow with the following command:

!!! terminal "code"

    ```bash
    snakemake --report ../results/snakemake_report.html
    ```

!!! note "Note"
    you won't be able to view a rendered version of this html while it is on the remote server, however after you transfer it to your local computer you should be able to view it in your web browser*

In our report:

- We get an interactive version of our directed acyclic graph (DAG).
- When you click on a node in the DAG, the input and output files are fully outlined, the exact software used and the exact shell command that was run.
- You are also provided with runtime information under the `Statistics` tab outlining how long each rule/sample ran for, and the date/time each file was created.

!!! html5 "My report"

    ![snakemake_report](./images/snakemake_report.gif)


<br>

These reports are highly configurable, have a look at an example of what can be done with a report [here](https://koesterlab.github.io/resources/report.html)

*See more information on creating Snakemake reports [in the Snakemake documentation](https://snakemake.readthedocs.io/en/stable/snakefiles/reporting.html)*

## 4.7 Linting your workflow

Snakemake has a built in linter to support you building best practice workflows, let's try it out:

!!! terminal "code"

    ```bash
    snakemake --lint
    ```

    ??? success "output"

        ```bash
        
        ```


<br>

We have a few things we could improve in our workflow!

Writing a best practice workflow is more important than having Marie Kondo level tidiness, it increases the chance your workflow will continue to be used and maintained by others (and ourselves), making the code we write useful (it's exciting seeing someone else using your code!). If your workflow was used in scientific research, it makes your workflow accessible for people to reproduce your research findings; it isn't going to be a nightmare for them to run and they are more likely to try and have success doing so.

Read more about the [best practices for Snakemake](https://snakemake.readthedocs.io/en/stable/snakefiles/best_practices.html)

# Takeaways

---
!!! quote ""

    - Pull out your parameters and put them in `params:` directive
    - Pulling the user configurable options away from the core workflow will support reproducibility by reducing the chance of changes to the core workflow
    - Leaving messages for the user of your workflow will help them understand what is happening at each stage and follow the workflows progress
    - Mark files you won't need once the workflow completes to reduce the memory usage - *particularly* when dealing with big data
    - Generate a snakemake report to get a summary of the workflow run - these are highly configurable
    - Lint your workflow and check it complies with best practices - this supports reproducibility and portability
    - There is so much more to explore, such as creating [modular workflows](https://snakemake.readthedocs.io/en/stable/snakefiles/modularization.html), automatically grabbing [remote files](https://snakemake.readthedocs.io/en/stable/snakefiles/remote_files.html) from places like Google Cloud Storage and Dropbox, [run various types of scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#external-scripts) such as [python scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#python), [R and RMarkdown scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#r-and-r-markdown) and [Jupyter Notebooks](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#jupyter-notebook-integration)
    
---

# Summary commands

!!! terminal ""

    - Use the parameter directive (`params`) to keep the parameters and flags of your programs separate from your shell command, for example:
    
    ```bash
    params:
        "--paired"
    ```
    
    - Run your snakemake workflow (using environment modules to load your software AND with a configuration file) with:
    
    ```bash
    snakemake --cores 2 --use-envmodules --configfile ../config/config.yaml
    ```
    
    - Alternatively, define your config file in the Snakefile:
    
    ```bash
    configfile: "../config/config.yaml"
    ```
    
    - Use the `message` directive to provide information to the user on what is happening real time, for example:
    
    ```bash
    message:
        "Undertaking quality control checks {input}"
    ```
    
    - Mark temporary files to remove (once they are no longer needed by the workflow) with `temp()`, for example:
    
    ```bash
    temp(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"])
    ```
    
    - Create a basic interactive Snakemake report after running your workflow with:
    
    ```bash
    snakemake --report ../results/snakemake_report.html
    ```

# Our final snakemake workflow!

See [leveled_up_demo_workflow](https://github.com/nesi/snakemake_workshop/tree/main/leveled_up_demo_workflow) for the final Snakemake workflow we've created up to this point

- - - 


<p align="center"><b><a class="btn" href="https://nesi.github.io/snakemake_workshop/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
