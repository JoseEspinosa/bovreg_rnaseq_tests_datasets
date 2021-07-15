# BovReg nf-core-rnaseq pipeline modifications notes

## Cmds

* Run the bovreg test in the cluster

```console
NXF_VER=21.04.1 nextflow run main.nf --save_reference -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity -bg -resume
```

* Run the bovreg test in the cluster with hisat2

```console
NXF_VER=21.04.1 nextflow run main.nf --save_reference --aligner hisat2 --max_memory 24GB -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity -bg -resume
```

* Run the bovreg test in local

```console
NXF_VER=21.04.1 nextflow run main.nf --skip_feelnc -profile docker,test -resume
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

## Issue with awk in the biotype assignment

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

[ ] hacer lo de `-e`, skip stringtie second steps

[ ] comprobar lo de la shebang de awk

[ ] subworkflows
    [x] feelnc
    [ ] stringtie additional steps

[ ] add `skip-feelnc` in `nextflow.schema`

[x] hisat2 does it works?

* Not mandatory

[ ] add more files from `codpot` and `classifier`logs and stuff

[x] Move `format_stringtie_gtf`from `modules.config` to something such as `deseq2_qc_salmon_options.publish_dir` in `rnaseq.nf` NOT NEEDED there are examples such as:

```console
'qualimap_rnaseq' {
    publish_dir     = "${params.aligner}/qualimap"
}
```

* [36/2e55c1] Cached process > NFCORE_RNASEQ:RNASEQ:FORMAT_STRINGTIE_GTF (stringtie.merged.gtf)

```console
cd /users/cn/jespinosa/scratch/36/2e55c1e0f98aade85f4e9b3e410592
grep ENSBTAT00000008737 ARS-UCD1.2_Btau5.0.1Y_lifted_from_Ensemblv102.gtf
grep ENSBTAT00000008737 stringtie.merged.biotypes.gtf 
grep ENSBTAT00000008737 new.genes.gff
```

cat stringtie.merged.biotypes.gtf | grep StringTie | grep protein_coding | grep transcript:ENSBTAT00000069841
cat new.genes.gff | grep transcript:ENSBTAT00000069841

Estas lineas las necesito?

```console
reference_annotation_to_quantify.combine(Channel.of('reference')).concat(
  novel_annotation_to_quantify.combine(Channel.of('novel'))
).combine(maps_to_quantify).set {
  maps_to_quantify
}
```

Creo que solo es crear un channel con las values reference y novel

Si y luego lo que hace es correr stringtie con la referencia y con el nuevo gtf
## Modules stuff

[ ] Stringtie merge does not output version