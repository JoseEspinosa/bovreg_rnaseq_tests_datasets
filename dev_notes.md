# BovReg nf-core-rnaseq pipeline modifications notes

## Cmds

* Run the bovreg test in the cluster

```console
NXF_VER=21.04.1 nextflow run main.nf --save_reference -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity -bg -resume
```

* Run tagada pipeline on test profile in local

```console
NXF_VER=21.04.1 nextflow run main.nf --output directory -profile test,singularity
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
