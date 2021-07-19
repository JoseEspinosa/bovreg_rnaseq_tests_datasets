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

2.1 Categories

{INPUT}.lncRNA.gtf || {INPUT}.lncRNA.fa: a .GTF/.FA file of the transcripts below the CPS (i.e the final set of lncRNAs).
 - {INPUT}.mRNA.gtf || {INPUT}.mRNA.fa: a .GTF/.FA file of the transcripts above the coding potential cutoff (i.e the final set of mRNAs).

FEELnc allows the user to increase the performance metrics to obtain high-confidence predictions of lncRNAs/mRNAs, although this option leads to the creation of an intermediate category of ambiguous coding/noncoding transcripts (**TUCp**).

3. `FEELnc_classifier.pl`

The third FEELnc module (`FEELnc_classifier`) formalizes the annotation of lncRNAs based on neighboring genes in order to predict lncRNA functions and RNA partners.
Examples: 

* intergenic antisense upstream’ which correspond to divergent lincRNAs (i.e. transcribed in head to head orientation with the RNA partner).

* lincRNAs located less than 5kb from their mRNA partner, belong to the ‘sense intergenic upstream’ class and may correspond to dubious lncRNAs that are actually 5΄UTR extensions of the neighboring protein-coding RNAs.

### prepDE.py (from Stringtie [docs](https://ccb.jhu.edu/software/stringtie/index.shtml?t=manual))

prepDE.py derives hypothetical read counts for each transcript from the coverage values estimated by StringTie for each transcript, by using this simple formula: reads_per_transcript = coverage * transcript_len / read_len

There are two ways to provide input to the prepDE.py script:

1. one option is to provide a path to a directory containing all sample sub-directories, with the same structure as the ballgown directory in the StringTie protocol paper in preparation for Ballgown. By default (no -i option), the script is going to look in the current directory for all sub-directories having .gtf files in them, as in this example:

    ```console
    ./sample1/sample1.gtf
    ./sample2/sample2.gtf
    ./sample3/sample3.gtf
    ```

Alternatively, one can provide a text file listing sample IDs and their respective paths (sample_lst.txt)