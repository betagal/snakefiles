# kallisto.snakefile

## View the job graph

```bash
snakemake \
  --snakefile kallisto.snakefile \
  --configfile config.yml \
  --forceall \
  --dag \
  | dot -Tpng > dag.png
```

![Snakemake directed acyclic graph (DAG).][dag]

[dag]: https://github.com/slowkow/snakefiles/blob/master/kallisto/dag.png

## Run the Snakefile

You can run snakemake like this. Notice the `--cluster 'bsub ...` option, used
to launch jobs on an [LSF] cluster.

[LSF]: https://en.wikipedia.org/wiki/Platform_LSF

```bash
snakemake \
  --snakefile kallisto.snakefile \
  --configfile config.yml \
  --jobs 999 \
  --cluster 'bsub -q normal -R "rusage[mem=4000]"'
```

```
Provided cluster nodes: 999
Job counts:
        count   jobs
        1       all
        1       collate_kallisto
        1       kallisto_index
        2       kallisto_quant
        5
rule kallisto_index:
        input: ../data/ensembl/Homo_sapiens.GRCh38.cdna.all.fa
        output: ../data/ensembl/kallisto/Homo_sapiens.GRCh38.cdna.all
        log: ../data/ensembl/kallisto/kallisto.index.log
        benchmark: ../data/ensembl/kallisto/kallisto.index.benchmark.tsv
1 of 5 steps (20%) done
rule kallisto_quant:
        input: ../data/ensembl/kallisto/Homo_sapiens.GRCh38.cdna.all, ../data/fastq/Sample2.R2.fastq.gz, ../data/fastq/Sample2.R1.fastq.gz
        output: ./Sample2/abundance.tsv
rule kallisto_quant:
        input: ../data/ensembl/kallisto/Homo_sapiens.GRCh38.cdna.all, ../data/fastq/Sample1.R2.fastq.gz, ../data/fastq/Sample1.R1.fastq.gz
        output: ./Sample1/abundance.tsv
2 of 5 steps (40%) done
3 of 5 steps (60%) done
rule collate_kallisto:
        input: ./Sample2/abundance.tsv, ./Sample1/abundance.tsv
        output: abundance.tsv.gz
4 of 5 steps (80%) done
localrule all:
        input: abundance.tsv.gz
5 of 5 steps (100%) done
```

## Output

The collated output looks like this:

```bash
zcat abundance.tsv.gz | head | column -t
```

```
sample   target_id        length  eff_length  est_counts  tpm
Sample2  ENST00000415118  8       9           0           0
Sample2  ENST00000448914  13      14          0           0
Sample2  ENST00000631435  12      13          0           0
Sample2  ENST00000434970  9       10          0           0
Sample2  ENST00000632684  12      13          0           0
Sample2  ENST00000431440  16      17          0           0
Sample2  ENST00000454691  18      19          0           0
Sample2  ENST00000451044  17      18          0           0
Sample2  ENST00000390581  23      24          0           0

```

```bash
zcat abundance.tsv.gz | tail
```

```
Sample1 ENST00000625481 1490    1334.85 0       0
Sample1 ENST00000631636 474     321.098 0       0
Sample1 ENST00000633023 398     250.721 0       0
Sample1 ENST00000632698 452     300.12  0       0
Sample1 ENST00000631664 178     69.0117 0       0
Sample1 ENST00000627692 927     771.852 0       0
Sample1 ENST00000631343 1490    1334.85 0       0
Sample1 ENST00000633652 508     355.098 0       0
Sample1 ENST00000634119 1226    1070.85 0       0
Sample1 ENST00000631874 147     49.842  0       0
```
