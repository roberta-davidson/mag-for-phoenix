# mag-for-phoenix

How to run nf-core/mag on University of Adelaide's "Phoenix" HPC

nf-core/mag [website](https://nf-co.re/mag/2.5.4) & [GitHub](https://github.com/nf-core/mag/) \
I used version 2.5.4 of the pipeline while writing this wiki, some bugs may be resolved, others may be new - good luck!

## Dependencies

mag requires Nextflow DSL2 (different to DSL1 that eager uses). 

So you need Nextflow version > 22.03.0-edge 

On Phoneix we have the module Nextflow/23.03.0 so do: `module load Nextflow/23.03.0` before running the pipeline.

We use Singularity to handle packages used by the pipeline so run `module load Singularity/3.10.5` as well.

## Institutional Configuration

Other information that the pipeline needs to run on Phoenix is contained within the institutional configuration file. Called `phoenix.config` in this repo, copy it and pass to your submission script with `-c phoenix.config`.

## Running the pipeline

There is lots of good info on how to build a mag script on the [website](https://nf-co.re/mag/2.5.4)

Here is the one I'm currently using as an example:

```nextflow
nextflow run nf-core mag -r 2.5.4 \
 -c ./phoenix.config \
 -profile singularity \
 --outdir /hpcfs/users/a1717363/micro_func/results \
 --input samplesheet_paired.csv \
 --reads_minlength 15 \
 --megahit_options="--presets meta-large" \
 --skip_spades \
 --skip_spadeshybrid \
 --binning_map_mode own \
 --min_contig_size 1500 \
 --bowtie2_mode="--very-sensitive" \
 --binqc_tool checkm \
 --checkm_db /hpcfs/users/a1717363/micro_func/DB/CheckM/ \
 --skip_concoct \
 --refine_bins_dastool \
 --run_gunc \
 --gunc_db /hpcfs/users/a1717363/micro_func/DB/gunc/gunc_db_progenomes2.1.dmnd \
 --gtdb /hpcfs/users/a1717363/micro_func/DB/gtdbtk/gtdbtk_r214_data.tar.gz \
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
