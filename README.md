# mag-for-phoenix

How to run nf-core/mag on University of Adelaide's "Phoenix" HPC

nf-core/mag [website](https://nf-co.re/mag/2.5.4) & [GitHub](https://github.com/nf-core/mag/) \
I used version 2.5.4 of the pipeline while writing this wiki, some bugs may be resolved, others may be new - good luck!

## Dependencies

mag requires Nextflow DSL2 (different to DSL1 that eager uses). \
So you need Nextflow version > 22.03.0-edge \
on Phoneix we have the module Nextflow/23.03.0 so do: `module load Nextflow/23.03.0` before running the pipeline.

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
