# mag-for-phoenix

How to run nf-core/mag on University of Adelaide's "Phoenix" HPC

nf-core/mag [website](https://nf-co.re/mag/2.5.4) & [GitHub](https://github.com/nf-core/mag/) \
I used version 2.5.4 of the pipeline while writing this wiki, some bugs may be resolved, others may be new - good luck!

## Dependencies

mag requires Nextflow DSL2 (different to DSL1 that eager uses).

So you need Nextflow version > 22.03.0-edge

On Phoneix we have the module Nextflow/23.03.0 so do: `module load Nextflow/23.03.0` before running the pipeline. \
(For a reason that I forgot, I installed Nextflow 23.10.0 myself and use that)

We use Singularity to handle packages used by the pipeline so run `module load Singularity/3.10.5` as well.

## Institutional Configuration

Other information that the pipeline needs to run on Phoenix is contained within the institutional configuration file. Called `phoenix.config` in this repo, and here I have also included additional modifications I found useful. Copy it and pass to your submission script with `-c phoenix.config`.

## Running the pipeline

There is lots of good info on how to build a mag script on the nf-core/mag [website](https://nf-co.re/mag/2.5.4)

Here is the one I'm currently using as an example:

```nextflow
nextflow run nf-core mag -r 2.5.4 \
 -c ./phoenix.config \
 -profile singularity \
 --outdir /hpcfs/users/aXXXXXX/micro_func/results \
 --input samplesheet_paired.csv \
 --reads_minlength 15 \
 --megahit_options="--presets meta-large" \
 --skip_spades \
 --skip_spadeshybrid \
 --binning_map_mode own \
 --min_contig_size 1500 \
 --bowtie2_mode="--very-sensitive" \
 --binqc_tool checkm \
 --checkm_db /hpcfs/users/aXXXXXX/micro_func/DB/CheckM/ \
 --skip_concoct \
 --refine_bins_dastool \
 --run_gunc \
 --gunc_db /hpcfs/users/aXXXXXX/micro_func/DB/gunc/gunc_db_progenomes2.1.dmnd \
 --gtdb /hpcfs/users/aXXXXXX/micro_func/DB/gtdbtk/gtdbtk_r214_data.tar.gz \
 --ancient_dna \
 --pydamage_accuracy 0.5
```

## Running nf-core/mag in a screen on Phoenix

- The nextflow pipeline will manage submission of individual slurm jobs to Phoenix.
- This allows you to run the pipeline command from the terminal directly and then monitor progress as the jobs run.
- BUT - if you get cut off from your login the pipeline will crash.
- SOLUTION: run from a "screen"
- A screen looks like a terminal but you can attach and detach from it wthout it breaking.

1. run a screen named mag:

```bash
screen -S mag
```

2. Load the required modules:

```bash
module use /apps/skl/modules/all/
module load Singularity/3.10.5
module load Nextflow/23.03.0
```

3. Copy your nextflow run command onto the command line and press enter
4. You should be able to see the progression of the pipeline in real time.
5. To detatch from the screen press `Ctrl`+`a` and then `d`
6. You can check the queue as normal to see which jobs are running:

```bash
squeue -u aXXXXXXX`
```

7. to Re-attatch to screen:

```bash
screen -r mag
```

8. If the pipeline fails, investigate/solve the error and then resume from where you left of by running the same command but add `-resume` to it

## Issues and my solutions

Various issues I encountered while running this pipeline on Phoenix. I've done my best to describe the solutions below, if you find a better one please let me know!

### Single end data doesn't work

I don't remember why but SE data does not work, nor does collapsed paired end data pretending to be single end - use paired end reads!

### Database download fails

**Problem:** mag uses databases in a lot of steps. The default is for the pipeline to doenload these for you and then run the program. \
This doens't work on phoenix because there is no internet access on the compute nodes.

**Solution:** Download the databases manually from the login node (where there is internet) and give the paths in the command (like my example above) \
FYI: I have heard they are planning to change this behaviour in future versions of the pipeline.

I have databases for CheckM, gunc, gtdbtk and busco saved and hard-coded the paths to the command.

### ERROR: "...mag stickied on revision X.XX..."

I don't know why this happens but sometimes it can't find the version of the pipeline you are asking for even if it exists and you've used it previously. (I had this issue with nf-core/eager once too).

My solution was to clone my own copy of the mag repository locally and use that:

1. Go to the [mag GitHub](https://github.com/nf-core/mag) to get the up to date repo link (click the green "Code" button and copy the link)

2. Run:

```bash
git clone https://github.com/nf-core/mag.git 
```

3. Now you should have a directory `mag/`

4. Change the first line of your pipeline run script replace `nf-core/mag -r X.XX` with `<path>/mag/`:

```bash
nextflow run <path>/mag/ -c
```

### Prokka issue - fails on `cat` command

Looks like this:

```nextflow
Error executing process > 'NFCORE_MAG:MAG:PROKKA (MEGAHIT-MetaBAT2-group-10.14)'

Caused by:
Process NFCORE_MAG:MAG:PROKKA (MEGAHIT-MetaBAT2-group-10.14) terminated with an error exit status (2)

Command executed:

prokka
--metagenome
--cpus 2
--prefix MEGAHIT-MetaBAT2-group-10.14


MEGAHIT-MetaBAT2-group-10.14.fa

cat <<-END_VERSIONS > versions.yml
"NFCORE_MAG:MAG:PROKKA":
prokka: 
(prokka --version 2>&1) | sed 's/^.*prokka //')
END_VERSIONS

Command exit status:
2

Command output:
(empty)

Command error:
.
.
.
[21:24:52] Running: cat MEGAHIT-MetaBAT2-group-10.14/MEGAHIT-MetaBAT2-group-10.14.IS.tmp.42.faa | parallel --gnu --plain -j 2 --block 14374 --recstart '>' --pipe blastp -query - -db /usr/local/db/kingdom/Bacteria/IS -evalue 1e-30 -qcov_hsp_perc 90 -num_threads 1 -num_descriptions 1 -num_alignments 1 -seg no > MEGAHIT-MetaBAT2-group-10.14/MEGAHIT-MetaBAT2-group-10.14.IS.tmp.42.blast 2> /dev/null
[21:24:53] Could not run command: cat MEGAHIT-MetaBAT2-group-10.14/MEGAHIT-MetaBAT2-group-10.14.IS.tmp.42.faa | parallel --gnu --plain -j 2 --block 14374 --recstart '>' --pipe blastp -query - -db /usr/local/db/kingdom/Bacteria/IS -evalue 1e-30 -qcov_hsp_perc 90 -num_threads 1 -num_descriptions 1 -num_alignments 1 -seg no > MEGAHIT-MetaBAT2-group-10.14/MEGAHIT-MetaBAT2-group-10.14.IS.tmp.42.blast 2> /dev/null
```

This is an unresolved issue reported [here](https://github.com/nf-core/mag/issues/601) \
For some reason I found that this is resolved by running prokka locally on the login node, rather than submitting to the compute nodes. This should be avoided when using a HPC but in this case it is the only way to run the step of the pipeline. \
I added this to the `phoenix.config` file to execute locally and one at a time so the same temp files are not overwritten:

```bash
process {
  withName: PROKKA {
    container = '/hpcfs/groups/acad_users/containers/prokka_1.14.6--pl5321hdfd78af_5.sif'
    executor = 'local'
    maxForks = 1
    }
}
```

### GTDBTK_CLASSIFYWF step fails, PplacerException

This is an error in the pplacer step of GTDB-Tk, also seen [here](https://github.com/Ecogenomics/GTDBTk/issues/352). The step is loading a whole phylogenetic tree so requires a lot of RAM. I added this to the config file to account for it.

```bash
process {
  withName: GTDBTK_CLASSIFYWF {
    memory = '200G'
  }
}
```

### process failing because no long contigs generated

I don't remember the name of the process but typically if the sample does not have enough reads to generate any contigs longer than a certain length, it will error and stop the pipeline. \
To ignore errors on this specific step add this to the `phoenix.config` file to ignore errors generated by that specifc step:

```bash
process {
  withName: <process_name> {
    errorStrategy 'ignore'
  }
}
```
