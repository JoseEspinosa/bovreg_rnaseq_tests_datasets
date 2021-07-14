# Development notes

## Description of the TAGADA workflow

### Stringtie

The workflow implements 3 `stringtie` steps:

1. assemble:

    ```console
    stringtie $map $direction -G $annotation -o "$prefix".gff
    ```

2. combine

    ```console
    stringtie --merge $assemblies -G $annotation -o novel_no_biotype.gff
    ```

3. quantify

    ```console
    stringtie $map \\
        $direction \\
        -e \\
        -B \\
        -G $annotation \\
        -A "$prefix"."$type"_genes_TPM.tsv \\
        -o "$prefix"."$type".gtf
    ```

    Note: `-e` only estimate the abundance of given reference transcripts (requires `-G`)

In summary what they do is to assemble the transcripts without using the `-e` flag, i.e. Stringtie estimates the abundance of de novo transcripts (not in the `gtf` file provided with the `-G` option).

The equivalent thing in the nf-core pipeline is to use the `--stringtie_ignore_gtf` parameter, this way the `-e` option is not used, **by default it is used**.

### FEELnc

The info is mainly summarized from [here](https://github.com/tderrien/FEELnc)

1. `FEELnc_filter.pl` removes transcripts with exons of the reference annotation, especially protein coding exons. These transcripts are filtered since the most probable is that they belong to new mRNA isoforms

    ```console
    FEELnc_filter.pl --mRNAfile $reference_annotation \\
        --infile $novel_annotation \\
        --biotype transcript_biotype=protein_coding \\
        > candidate_transcripts.gtf
    ```

    Note:

    ```console
    -b,--biotype                  Only consider transcript(s) from the reference annotation having this(these) biotype(s) (e.g : -b transcript_biotype=protein_coding,pseudogene) [default undef i.e all transcripts]
    ```

2. `FEELnc_codpot.pl`

The main step of the pipeline (FEELnc_codpot) aims at computing the coding potential score (CPS),ranging between 0 and 1, foreach of the candidate transcripts in the `candidate_lncRNA.gtf` file.

3. `FEELnc_classifier.pl`

The third FEELnc module (`FEELnc_classifier`) formalizes the annotation of lncRNAs based on neighboring genes in order to predict lncRNA functions and RNA partners.
Examples: 

* intergenic antisense upstream’ which correspond to divergent lincRNAs (i.e. transcribed in head to head orientation with the RNA partner).

* lincRNAs located less than 5kb from their mRNA partner, belong to the ‘sense intergenic upstream’ class and may correspond to dubious lncRNAs that are actually 5΄UTR extensions of the neighboring protein-coding RNAs
## Things TODO

* Check if `stringtie` needs to be reimplemented for `stringtie quantify` **#Done**

    The same module can be used, note that the option `stringtie_ignore_gtf` should be used

* Parametrize everything in `modules.config` **#Done**

* Test with real cow data. **#workInProgress**

* Check what does all the code of TAGADA for rearranging files

* Implement `skip_feelnc` and skip it for tests **#Done**

* Add cpus parameter to command line of stringtie

* Create a complete subworkflow for:
    * stringtie
    * feelnc

* Change `stringtie_merge` by the version on nf-core **#Done**

* This one was tag for using it as a template, but I don't remember why: `$params.gtf_group_features` in `salmon_tx2gene.nf`
## In case of having time

* Multiqc?

* Check the gff and gtf naming

## Test configuration

The minimum reference genome requirements are a FASTA and GTF file, all other files required to run the pipeline can be generated from these files. However, it is more storage and compute friendly if you are able to re-use reference genome files as efficiently as possible. It is recommended to use the --save_reference

