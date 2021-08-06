# BovReg nf-core-rnaseq pipeline modifications notes

## Cmds

### Cluster

* Run the bovreg test in the cluster

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --save_reference -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity -bg -resume
  ```

* Run the bovreg test in the cluster with hisat2

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --save_reference --aligner hisat2 --max_memory 24GB -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity -bg -resume
  ```

### Local

* Run the bovreg test in local

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --skip_feelnc --stringtie_ignore_gtf --multiqc_custom_config assets/multiqc_config_bovreg.yaml -profile docker,test -resume
  ```

* Local with aws data

  ```console
  NXF_VER=21.04.1 nextflow run main.nf -c ./conf/test_bovreg.config --skip_feelnc --stringtie_ignore_gtf -profile docker
  ```

* Run FAANG/TAGADA pipeline on test profile in local

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --output directory -c test/test.config -profile singularity -resume
  ```

## Test configuration

The minimum reference genome requirements are a FASTA and GTF file, all other files required to run the pipeline can be generated from these files. However, it is more storage and compute friendly if you are able to re-use reference genome files as efficiently as possible. It is recommended to use the --save_reference

## Obtain reference

* Reference used by Gabriel:

  ```console
  http://ftp.ensembl.org/pub/release-102/gff3/bos_taurus/
  ```

* Index of ftp

  ```console
  https://www.ensembl.org/info/data/ftp/index.html
  ```

  Am I merging the three gffs from the bam at the same time  (correct) or one by one

## awk vs. gawk

The container needs `gawk` to make the awk scripts work.

## TODOs

### Mandatory

* Issue with awk in the biotype assignment

  Probably the problem is that the `transcript_id` is `transcript:ENSBTAT00000007786` in the file `ARS-UCD1.2_Btau5.0.1Y_lifted_from_Ensemblv102.gtf` the `transcript:` probably is the problem to match the ids.

  transcript_id "transcript:ENSBTAT00000007786"; gene_id "ENSBTAG00000006648"; biotype "protein_coding"; transcriptID "ENSBTAT00000007786"; version "5"; extra_copy_number "0"; logic_name "ensembl"; coverage "1.0"; sequence_ID "1.0"; copy_num_ID "gene:ENSBTAG00000006648_0";

  transcript_id "transcript:ENSBTAT00000007786"; gene_id "ENSBTAG00000006648"; protein_id "ENSBTAP00000007786"; extra_copy_number "0";

* [x] awk shebang problem, check it with the script

  I finally implemented directly in the shell section the awk code since otherwise I needed a shebang for awk that is different for conda and containers environments.
  If I just put the snippet there in `FORMAT_STRINGTIE_GTF` then it works.

* [x] Create a complete subworkflow for:

  * [x] feelnc
  * [x] stringtie additional steps

* [x] Implement `skip_feelnc` and skip it for tests **#Done**

* [x] add `skip-feelnc` in `nextflow.schema`

* [x] hisat2 does it works?

* [x] If -e is specified avoid merge and quantify

* [x] Parametrize everything in `modules.config` **#Done**

* [x] Change `stringtie_merge` by the version on nf-core

* [x] Hisat 2 run aligner test:

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --save_reference -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity,bovreg --aligner hisat2 -bg -resume
  ```

* [x] Check if `stringtie` needs to be reimplemented for `stringtie quantify` **#Done**

    The same module can be used, note that the option `stringtie_ignore_gtf` should be used

* [x] Check what does all the code of TAGADA for rearranging files

* [x] Test with real cow data. **#workInProgress**

* [x] Check how to create the python script to format the gtf files

  [36/2e55c1] Cached process > NFCORE_RNASEQ:RNASEQ:FORMAT_STRINGTIE_GTF (stringtie.merged.gtf)

  ```console
  cd /users/cn/jespinosa/scratch/36/2e55c1e0f98aade85f4e9b3e410592
  grep ENSBTAT00000008737 ARS-UCD1.2_Btau5.0.1Y_lifted_from_Ensemblv102.gtf
  grep ENSBTAT00000008737 stringtie.merged.biotypes.gtf 
  grep ENSBTAT00000008737 new.genes.gff
  ```

  cat stringtie.merged.biotypes.gtf | grep StringTie | grep protein_coding | grep transcript:ENSBTAT00000069841
  cat new.genes.gff | grep transcript:ENSBTAT00000069841

* [x] Linting

* [ ] CI implement

* [x] Add cpus parameter to command line of stringtie

* [ ] Check for missing TAGADA steps.

* [x] Update stringtie version

* [x] Update the way stringtie prepDE outputs version

* [ ] Add versions collect

* [ ] Add stringtie to multiqc report

* [ ] Add new parameters to json file 

### Not mandatory

* [ ] Add new processes to the `multiqc` report

* [ ] add more files from `codpot` and `classifier`logs and stuff

* [x] Move `format_stringtie_gtf`from `modules.config` to something such as `deseq2_qc_salmon_options.publish_dir` in `rnaseq.nf` NOT NEEDED there are examples such as:

  ```console
  'qualimap_rnaseq' {
      publish_dir     = "${params.aligner}/qualimap"
  }
  ```

* [x] Do we need to also quantify against the original reference

  ```console
  reference_annotation_to_quantify.combine(Channel.of('reference')).concat(
    novel_annotation_to_quantify.combine(Channel.of('novel'))
  ).combine(maps_to_quantify).set {
    maps_to_quantify
  }
  ```

  It only creates a channel and tags it with novel of reference `value` depending on its origin

  ```console
  [/da/58a5f2ca542f5ef9b056ae98b3bbc6/genes.gtf, reference, WT, 98, --rf, /0c/8f535309165ee30a3d09cfd0a33d50/WT.bam]
  [/85/c04aa26ed32ec167771fb3216eb171/novel.gff, novel, WT, 98, --rf, /0c/8f535309165ee30a3d09cfd0a33d50/WT.bam]
  [/da/58a5f2ca542f5ef9b056ae98b3bbc6/genes.gtf, reference, RAP1_IAA_30M, 99, --rf, /b9/eeb3b4816401f5801fa3f0d6a28a9c/RAP1_IAA_30M.bam]
  [/85/c04aa26ed32ec167771fb3216eb171/novel.gff, novel, RAP1_IAA_30M, 99, --rf, /b9/eeb3b4816401f5801fa3f0d6a28a9c/RAP1_IAA_30M.bam]
  [/da/58a5f2ca542f5ef9b056ae98b3bbc6/genes.gtf, reference, RAP1_UNINDUCED, 98, --rf, /cd/30e1ee71643283534093000c8c2a42/RAP1_UNINDUCED.bam]
  [/85/c04aa26ed32ec167771fb3216eb171/novel.gff, novel, RAP1_UNINDUCED, 98, --rf, /cd/30e1ee71643283534093000c8c2a42/RAP1_UNINDUCED.bam]
  ```

  Si y luego lo que hace es correr stringtie con la referencia y con el nuevo gtf.

* [ ] Update logs, docs and similar stuff. #WorkInProgress

* [ ] Mention the procedence of the tagada scripts.

* [ ] Check the gff and gtf naming

* [ ] * This one was tag for using it as a template, but I don't remember why: `$params.gtf_group_features` in `salmon_tx2gene.nf`

  The `--gtf_group_features_type` parameter will automatically be set to `gene_type` as opposed to `gene_biotype`, respectively.

* [ ] Think whether the `-l 100` option of the `stringtie_prepde` module (`prepDE.py` execution) to provide the read length should be provided as a parameter.

* [x] Add the new parameters to the json file.

### Modules-related

* [x] Stringtie merge does not output the version

## Decisions during development that might be revisited

* I am outputting all the `feelnc`versions since they are separate modules (`feelnc_codplot`, `feelnc_classifier`, `feelnc_filter`)

## Things that might be tackle on future versions

* The arguments of the `feelnc/codplot` module are directly provided in the script section, they could be passed as arguments using the `modules.config` configuration file.

## Multiqc feelnc

* The problem is that to run multiqc the tagada pipeline uses a custom multiqc plugin, see [here](https://github.com/FAANG/analysis-TAGADA/blob/c3c8c6a334d67c1b520614935b730134cb95eeaa/Dockerfile#L28-L30)

We may have to develop/copy the plugin and make it work on the pipeline, the problem is that then the image will be a custom one. 