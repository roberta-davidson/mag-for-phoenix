# mag-for-phoenix
How to run nf-core/mag on University of Adelaide's "Phoenix" HPC 

nf-core/mag [website](https://nf-co.re/mag/2.5.4) & [GitHub](https://github.com/nf-core/mag/) \
I used version 2.5.4 of the pipeline while writing this wiki, some bugs may be resolved, others may be new - good luck!

## Dependencies
mag requires Nextflow DSL2 (different to DSL1 that eager uses). \
SO: you need Nextflow version > 22.03.0-edge \
on Phoneix we have the module Nextflow/23.03.0 so do: `module load Nextflow/23.03.0` before running the pipeline.
