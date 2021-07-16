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
  NXF_VER=21.04.1 nextflow run main.nf --skip_feelnc --stringtie_ignore_gtf -profile docker,test -resume
  ```

* Local with aws data

  ```console
  NXF_VER=21.04.1 nextflow run main.nf -c ./conf/test_bovreg.config --skip_feelnc --stringtie_ignore_gtf -profile docker
  ```

* Run tagada pipeline on test profile in local

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --output directory -c test/test.config -profile singularity -resume
  ```

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

## Issues

* Issue with awk in the biotype assignment

  Probably the problem is that the `transcript_id` is `transcript:ENSBTAT00000007786` in the file `ARS-UCD1.2_Btau5.0.1Y_lifted_from_Ensemblv102.gtf` the `transcript:` probably is the problem to match the ids.

  transcript_id "transcript:ENSBTAT00000007786"; gene_id "ENSBTAG00000006648"; biotype "protein_coding"; transcriptID "ENSBTAT00000007786"; version "5"; extra_copy_number "0"; logic_name "ensembl"; coverage "1.0"; sequence_ID "1.0"; copy_num_ID "gene:ENSBTAG00000006648_0";

  transcript_id "transcript:ENSBTAT00000007786"; gene_id "ENSBTAG00000006648"; protein_id "ENSBTAP00000007786"; extra_copy_number "0";

* awk shebang problem, check it with the script

  I finally implemented directly in the shell section the awk code since otherwise I needed a shebang for awk that is different for conda and containers environments.
  If I just put the snippet there in `FORMAT_STRINGTIE_GTF` then it works.

Linting
And CI
bucket s3
Pass you the files

* If -e is specified avoid merge and quantify

* Hisat 2 run aligner test:

  ```console
  NXF_VER=21.04.1 nextflow run main.nf --save_reference -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity,bovreg --aligner hisat2 -bg -resume
  ```

## Decisions during development that might be revisited

* I am outputting all the `feelnc`versions since they are separate modules (`feelnc_codplot`, `feelnc_classifier`, `feelnc_filter`)

## 14/07/2021 TODOs

* Mandatory

  [x] hacer lo de `-e`, skip stringtie second steps

  [ ] comprobar lo de la shebang de awk

  [x] subworkflows
      [x] feelnc
      [x] stringtie additional steps

  [x] add `skip-feelnc` in `nextflow.schema`

  [x] hisat2 does it works?

* Not mandatory

[ ] add more files from `codpot` and `classifier`logs and stuff

[x] Move `format_stringtie_gtf`from `modules.config` to something such as `deseq2_qc_salmon_options.publish_dir` in `rnaseq.nf` NOT NEEDED there are examples such as:

```console
'qualimap_rnaseq' {
    publish_dir     = "${params.aligner}/qualimap"
}
```

* Check how to create the python script to format the gtf files

  [36/2e55c1] Cached process > NFCORE_RNASEQ:RNASEQ:FORMAT_STRINGTIE_GTF (stringtie.merged.gtf)

  ```console
  cd /users/cn/jespinosa/scratch/36/2e55c1e0f98aade85f4e9b3e410592
  grep ENSBTAT00000008737 ARS-UCD1.2_Btau5.0.1Y_lifted_from_Ensemblv102.gtf
  grep ENSBTAT00000008737 stringtie.merged.biotypes.gtf 
  grep ENSBTAT00000008737 new.genes.gff
  ```

  cat stringtie.merged.biotypes.gtf | grep StringTie | grep protein_coding | grep transcript:ENSBTAT00000069841
  cat new.genes.gff | grep transcript:ENSBTAT00000069841

* Do we need to also quantify against the original reference

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

* Update logs, docs and similar stuff

* Mention the procedence of the tagada scripts

  Si y luego lo que hace es correr stringtie con la referencia y con el nuevo gtf
## Modules stuff

  [ ] Stringtie merge does not output the version
