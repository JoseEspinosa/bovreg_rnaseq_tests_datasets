# BovReg nf-core-rnaseq pipeline modifications notes

## Cmds

* Run the bovreg test in the cluster

```console
NXF_VER=21.04.1 nextflow run main.nf --save_reference -c ./conf/test_bovreg.config -c /users/cn/jespinosa/git/lab_nxf_configs/conf/crg_cbcrg.config -profile singularity -bg -resume
```

--limitGenomeGenerateRAM 8709498922