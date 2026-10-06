# Comparing Savont to EPI2ME wf-16s workflow

Ramon Reilman

This file will contain an overview of what I've done to compare these 2 workflows, and the results of that. Since the code takes up a lot of space, i moved all of these code blocks to the end of the report

## Methods / tools / code

I compared these 2 tools using 3 different types of data.

1. Potato data from Thomas
2. A Gutmock dataset from zymo
3. Simulated data using nanosim

### Potato data
I ran the wf-16s on the potato data (From Thomas Vogel), using less processes and a small queue size due to it crashing otherwise. Running it with a lower max_len value caused the results to be empty, so this was a necessary step. It was run using the Silva database, the same one as Savont. (Code Block 1)

These same values (like max length) were kept for Savont, to keep it equal. (Code Block 2)

### GutMock

For the gutmock i had to first turn the bam files into fasta files: (Code Block 3)

And i had to remove the adapters that were on the data: (Code Block 4)

After this I was able to use both wf-16s and Savont on the data. (Code Block 5)

### Simulated data

While both Savont and wf-16s ran and on the GutMock and potato data, there was no accurate way to compare these 2 tools for the following reasons:

1. We don't know what the exact microbiome is of real sequenced data (at least not in this case). This causes real data, like the potato dataset, to be inherently useless for comparing the 2 tools. So I read about the mock datasets (like gutmock), this however came with different issues.
2. Most, if not all, mock datasets that I found (like the gutmock dataset) have their ground truth values using NCBI tax ID's or NCBI naming conventions. Savont, at this moment, does not support an NCBI reference database. Since both tools need the same reference database, I could not use the NCBI for wf-16s, and another for Savont. This meant I had to generate a fake mock dataset using NanoSim, using a database that both

3. OPAL (the tools used to compare Savont and wf-16s) requires the input data to have two important things:
	- All input files need to have the CAMI Profiling output (version 0.9.3) https://github.com/bioboxes/rfc/blob/60263f34c57bc4137deeceec4c68a7f9f810f6a5/data-format/profiling.mkd
	- These input files need to contain TAXids. The issue with this is that I could not find any valid TAXids for the greengenes2 db (the only reference db that Savont and wf-16s share). So i had to generate my own TAXid database for greengenes2
	
4. The Savont greengenes2 db and wf-16s are not the same, the wf-16s version is a plus version that contains more species than the Savont version. To fix this I updated my own tool, that is able to generate a custom db for wf-16s, that is based on the Savont db. Using my own TAXids that were defined in the third point.

### Generating Simulated data

One of the first steps i had to do was split the Savont database. (Code Block 6)

I had made the choice to make my simulated dataset comparable to my gutmock dataset, planning to use the following species:

Bacteroides_H_857956 fragilis
Faecalibacterium prausnitzii
Veillonella_A rogosae
Prevotella corporis
Clostridioides_A difficile
Roseburia_A_166204 hominis
Fusobacterium_C animalis
Bifidobacterium_388775 adolescentis
Escherichia albertii
Limosilactobacillus fermentum
Escherichia sonnei
Escherichia ruysiae
Escherichia boydii
Akkermansia muciniphila_D_776786
Escherichia dysenteriae
Bifidobacterium_388775 faecale
Salmonella bongori

The TAXids for these species were already in my reference db at this point, so it was the easiest way to build new data. (Code Block 7)

This moves all of the files that contain sequences belonging to species in the species list to a different folder, that will contain the custom set of species.

The headers in these files are duplicated, this is something that NanoSim does not want, so i have to change these. The command below does the following:

Loops through all files, replaces their name with an ID (from my own taxid db) (Code Block 8)

This caused headers in the training sets to look like this: "\>58_c021" with 58 being the species taxID and the c021, the sequence id.

After training the data on the gutmock replicate 1, it revealed to me that the sequences my model had been trained with were too long, so i made the input file for this contain shorter sequences: (Code Block 9)

Running NanoSim without this step caused it to create new abundances with very high deviations from my expected abundances, in some cases even 2 or 3 orders of magnitude. Investigating this issue led met to NanoSim pull request #232 (A fix for abundance deviation in metagenome mode).

When a drawn read length exceeds the chosen reference sequence, NanoSim will fall back to a randomly selected sequence from the entire reference pool, if this happens enough times, the eventual true abundances will deviate from the input abundances. I ended up fixing this issue by limiting the length of sequences in the rep1 data, as seen above.

I could not change the median, sd, or max length of NanoSim later downstream. This was because anytime I used these flags, NanoSim would hang, seemingly indefinitely. This matches another known issue on the repo (#185). Only the species with lower abundance showed a slight deviation at the end, but nothing that is not negligible.

This range was chosen due to the input species files having sequences in this length range.

This could now be used to train a model (Code Block 10)

And that model could now be used to generate data (Code Block 11)

(Code Block 12)

### Running Savont and wf-16s

One of the last issues now was that both wf-16s and Savont don't use the same greengenes2 version, a fix for this was built into my python library that is being built for this project: https://github.com/RamonReilman/SavontAnalysis 
(Code Block 13)

This generates the following files:

- Gg2_renamed.fa, a fasta file with sequences, now with headers that have a ggX (so the first sequence is gg0). With their lineage: "\>gg0 d__Bacteria;p__Bacillota_A_368345;c__Clostridia_258483;o__Lachnospirales;f__Lachnospiraceae;g__Eubacterium_G_180878;s__ventriosum" for the first sequence in the file.
- Ref2taxid.tsv, maps these ggX id's to the fitting TAXids from my custom database
- Names.dmp, contains all names (and TAXids) for my entire db
- Nodes.dmp, contains all TAXids, taxonomic ranks, and parental TAXids for entire db

These files could now be used to run the wf-16s workflow (Code Block 14)

(Code Block 15)

(Code Block 16)

These files can now be transformed into the CAMI taxonomic standard with my library: (Code Block 17)

(Code Block 18)

(Code Block 19)

And can then be used as in input for OPAL: (Code Block 20)

## Results

### Potato-data

#### WF-16s

![help](../Resources/Compared/Potato/NextFlow/wf-16s.png)
Figure 1, the relative abundances of genera in the different potato samples generated by the wf-16s workflow.

#### Savont
![](Resources/Compared/Potato/Savont/abundance_plot.png)


Figure 2, relative abundance of genera in the potato data, generated by Savont.

### GutMock data

#### WF-16s
![](Resources/Compared/GutMock_SUP/NextFlow/wf-gutmock.png)
Figure 3, relative abundance of genera in replicate 2 of the gutmock dataset, generated by the wf-16s workflow
#### Savont
![](Resources/Compared/GutMock_SUP/Savont/abundance_plot.png)
Figure 4, relative abundance of genera in replicate 2 of the gutmock dataset, generated by the wf-16s workflow
### Generated Data

Table 1, Important metrics for every taxonomic rank (if available), between Savont and wf-16s.

| Metric | Rank | Savont | wf-16s |
|---|---|---|---|
| Detection (F1-score) | Phylum | 1.000 | 1.000 |
| Detection (F1-score) | Class | 1.000 | 1.000 |
| Detection (F1-score) | Order | 1.000 | 1.000 |
| Detection (F1-score) | Family | 1.000 | 1.000 |
| Detection (F1-score) | Genus | 0.957 | 1.000 |
| Detection (F1-score) | Species | 0.786 | 0.850 |
| Purity | Genus | 1.000 | 1.000 |
| Purity | Species | 1.000 | 0.739 |
| Completeness | Genus | 0.917 | 1.000 |
| Completeness | Species | 0.647 | 1.000 |
| Abundance accuracy (Bray-Curtis) | Phylum | 0.008 | 0.788 |
| Abundance accuracy (Bray-Curtis) | Class | 0.008 | 0.788 |
| Abundance accuracy (Bray-Curtis) | Order | 0.018 | 0.788 |
| Abundance accuracy (Bray-Curtis) | Family | 0.027 | 0.788 |
| Abundance accuracy (Bray-Curtis) | Genus | 0.035 | 0.788 |
| Abundance accuracy (Bray-Curtis) | Species | 0.050 | 0.791 |
| Abundance accuracy (L1) | Phylum | 0.015 | 0.882 |
| Abundance accuracy (L1) | Class | 0.015 | 0.882 |
| Abundance accuracy (L1) | Order | 0.035 | 0.882 |
| Abundance accuracy (L1) | Family | 0.052 | 0.882 |
| Abundance accuracy (L1) | Genus | 0.069 | 0.882 |
| Abundance accuracy (L1) | Species | 0.096 | 0.884 |
| Sum of abundance | Phylum | 0.989 | 0.118 |
| Sum of abundance | Species | 0.924 | 0.118 |
| Other | Weighted UniFrac error | 0.0024 | 0.0800 |
| Other | Unweighted UniFrac error | 0.204 | 0.538 |
| Other | Weighted UniFrac (CAMI) | 0.310 | 0.799 |
| Other | Unweighted UniFrac (CAMI) | 34.0 | 64.00 |

2 important metrics to discuss are purity and completeness.

Savont at the species level does not find any false positives (purity of 1), whereas wf-16s has a lower purity, and thus finds false positives. The completeness on the other hand is a bit lower for Savont, it does not find all species in the ground truth. The completeness for WF-16s is 1, meaning it has found all species that the ground truth expected. I am curious to see if we can possibly lower the amount of missing species in Savont.

The sum of abundance for wf-16s is extremely low (about 11%). I tried to lower the --min_ref_coverage from 90 to 70, which did increase the sum of abundance, but lowered the purity:

Purity

| Rank | Savont | wf-16s |
|---|---|---|
| Genus | 1.0 | 1.0 |
| Species | 1.0 | 0.68 |

Sum of abundance
- Previously: 11.8%
- Now: 52%

I could not seem to increase or decrease the species completeness for Savont two. After looking into what caused this lower completeness i found the following in the asv mapping file:

final_asv_69_depth_429 100.00% gEscherichia; -> Greengenes_unannotated
final_asv_69_depth_429 100.00% gEscherichia;s__albertii; -> Escherichia albertii

Savont's species-level misses are explained by unresolved ties between correct matching named-species references and incompletely-annotated genus-only records present in the Greengenes2 database itself. The 2 lines above (from asv_mappings.tsv) (ASV 69, ASV 71) shows the same ASV achieving 100% identity against both a named species and an unannotated genus-only record at the same time, with inconsistent resolution of the tie. I was unsure if this was intended behavior, so i talked to the developer who said the following:

"Here's how savont actually deals with tie breaks right now, which is a bit poorly documented and may be useful to know:

1. Each ASV gets mapped to the reference database. All unambiguously best hits are pooled together for each ASV.
2. We look at the best hits across all ASVs, and then prefer taxonomy that are seen commonly across all ASVs.

This is done for the following reason: suppose a genome has two 16S copies, with two corresponding ASVs. ASV 1 may map perfectly to two different species, whereas ASV 2 unambiguously maps to the correct species. In this case, we'll assign both ASVs to the correct species.

This has a corollary that if many references are called "g_....; unannotated", it'll prefer the unannotated, since it's seen a lot." - bluenote-1577.

## Conclusions / discussion

---
---

# Appendix: Commands

**Code Block 1**
```bash
nextflow run epi2me-labs/wf-16s \
--fastq /home/hackllab/for-ramon/potato-fungi-18S-20260817/fastq_pass/ \
--out_dir /home/ramon/output/runs_out/potato/next \
--classifier minimap2 \
-with-apptainer \
-without-docker \
-process.maxForks 1 \
 -queue-size 1 \
--database_set SILVA_138_1 \
--max_len 4000 \
--threads 2
```

**Code Block 2**
```bash
savont asv --pooled-samples \
/home/hackllab/for-ramon/potato-fungi-18S-20260817/fastq_pass/barcode01/FBH02060_pass_barcode01_10104705_00000000_0.fastq \
/home/hackllab/for-ramon/potato-fungi-18S-20260817/fastq_pass/barcode02/FBH02060_pass_barcode02_10104705_00000000_0.fastq \
/home/hackllab/for-ramon/potato-fungi-18S-20260817/fastq_pass/barcode03/FBH02060_pass_barcode03_10104705_00000000_0.fastq \
/home/hackllab/for-ramon/potato-fungi-18S-20260817/fastq_pass/barcode04/FBH02060_pass_barcode04_10104705_00000000_0.fastq \
/home/hackllab/for-ramon/potato-fungi-18S-20260817/fastq_pass/barcode05/FBH02060_pass_barcode05_10104705_00000000_0.fastq \
-o /home/ramon/output/runs_out/potato/savont_asv/ \
-M 4000 \
-t 5

savont classify -i /home/ramon/output/runs_out/potato/savont_asv/ -o /home/ramon/output/runs_out/potato/savont_tax/ -d /home/ramon/savont_db/silva-138.2 -t 4
```

**Code Block 3**
```bash
dir=/home/hackllab/for-ramon/GutMock_sup/
mkdir -p "$dir"
for file in /home/hackllab/for-ramon/zymo_16s_2025.09/GutMock/MAB114/sup/*; do
file_name=$(basename "$file")
bedtools bamtofastq -i "$file"/*.bam -fq /dev/stdout > "$dir"/"$file_name".fastq
done
```

**Code Block 4**
```bash
outpt=/home/hackllab/for-ramon/gutmock_out_sup/trimmed
mkdir -p "$outpt"
for file in /home/hackllab/for-ramon/GutMock_sup/*; do
file_name=$(basename "$file")
cutadapt -g 'AGRGTTYGATYMTGGCTCAG' -a 'SGGYTACCTTGTTACGACTT' -O 15 -e 0.1 --report=full -o "$outpt"/"$file_name" "$file"
done
```

**Code Block 5**
```bash
for rep in rep1 rep2 rep3 rep4; do
nextflow run epi2me-labs/wf-16s \
--fastq "/home/hackllab/for-ramon/gutmock_out_sup/trimmed/${rep}.fastq" \
--out_dir /home/ramon/output/runs_out/gutmock_nf_sup/${rep}/ \
--sample "$rep" \
--classifier minimap2 \
-with-apptainer \
-without-docker \
--database_set SILVA_138_1 \
--threads 4
done

savont asv --pooled-samples \
/home/hackllab/for-ramon/gutmock_out_sup/trimmed/rep1.fastq \
/home/hackllab/for-ramon/gutmock_out_sup/trimmed/rep2.fastq \
/home/hackllab/for-ramon/gutmock_out_sup/trimmed/rep3.fastq \
/home/hackllab/for-ramon/gutmock_out_sup/trimmed/rep4.fastq \
-o /home/ramon/output/runs_out/gutmock_nf_sup/savont_asv/ \
-t 5

savont classify -i /home/ramon/output/runs_out/gutmock_nf_sup/savont_asv/ -o /home/ramon/output/runs_out/gutmock_nf_sup/savont_taxo/ -d /home/ramon/savont_db/silva-138.2 -t 4
```

**Code Block 6**
```bash
seqkit split -i /home/ramon/savont_db/greengenes2-2024.09/gg2_2024_09_toSpecies_trainset.fa
```

**Code Block 7**
```bash
SPECIES_LIST="/home/ramon/species_list.txt"
cd /home/ramon/savontdb/greengenes2-2024.09
while IFS=" " read genus species; do
file_wanted=(find ./ -iname "*genus" -iname "species*")
cp $file_wanted /home/ramon/custom_set/"genus""species".fasta
done < "SPECIES_LIST"
```

**Code Block 8**
```bash
mkdir -p /home/ramon/custom_set/clean
cd /home/ramon/custom_set
: > genome_list.tsv
: > id_lineage.tsv

while read -r id path; do
path="{path/#\~/HOME}"
seqkit seq -m 1300 -M 1700 "$path"
| seqkit sample -n 30 -s 42
| seqkit replace -p '.+' -r 'c{nr}' --nr-width 3 \
> clean/id.fasta
printf '%s\t%s\n' "id" "/home/ramon/custom_set/clean/id.fasta" >> genome_list.tsv
printf '%s\t%s\n' "id" "(seqkit seq -n "path" | head -1)" >> id_lineage.tsv
done < /home/ramon/savont_db/greengenes2-2024.09/genome_list.txt

for f in clean/*.fasta; do
id=(basename "f" .fasta)
seqkit replace -p '^' -r "{id}_" "f"
done > train_ref.fasta
```

**Code Block 9**
```bash
seqkit seq -m 1300 -M 1500 /home/hackllab/for-ramon/GutMock_sup/rep1.fastq > /home/hackllab/for-ramon/GutMock_sup/rep1_short.fastq
```

**Code Block 10**
```bash
python3 /home/ramon/tools/NanoSim/src/read_analysis.py genome -i /home/hackllab/for-ramon/GutMock_sup/rep1_short.fastq -rg /home/ramon/custom_set/train_ref.fasta -o ~/models/dept_model_v2 --fastq -t 4
```

**Code Block 11**
```bash
python3 /home/ramon/tools/NanoSim/src/simulator.py metagenome -gl /home/ramon/custom_set/genome_list.tsv -a /home/ramon/savont_db/greengenes2-2024.09/abundance.tsv -c /home/ramon/models/dept_model_v2 -o /home/ramon/test_data/sim_final_v2 --fastq --seed 42 -t 10 2> final_v2_warnings.log
```

**Code Block 12**
```bash
python3 /home/ramon/tools/NanoSim/src/read_analysis.py quantify -e meta -i /home/ramon/test_data/sim_final_v2_sample0_aligned_reads.fastq -gl /home/ramon/custom_set/genome_list.tsv -o /home/ramon/run/fin_v2 -t 4
```

**Code Block 13**
```bash
savont-analyse customdb --input_fasta /home/redman/jaar4/greengenes2-2024.09/gg2_2024_09_toSpecies_trainset.fa
```

**Code Block 14**
```bash
savont asv ~/test_data/sim_final_v2_sample0_aligned_reads.fastq -o ~/output/runs_out_greengenes2/savont_asv -t 5 --quality-value-cutoff 90
```

**Code Block 15**
```bash
savont classify -i ~/output/runs_out_greengenes2/savont_asv/ -o ~/output/runs_out_greengenes2/savont_tax -d ~/savont_db/greengenes2-2024.09/ -t 5
```

**Code Block 16**
```bash
nextflow run epi2me-labs/wf-16s --fastq "/home/ramon/test_data/sim_final_v2_sample0_aligned_reads.fastq" --out_dir /home/ramon/output/runs_out_greengenes2/sim_v2_data/rep0/ --classifier minimap2 -process.maxForks 1 -queue-size 1 -with-apptainer -without-docker --reference /home/ramon/savont_db/greengenes2-2024.09/new_db/gg2_renamed.fa --ref2taxid /home/ramon/savont_db/greengenes2-2024.09/new_db/ref2taxid.tsv --taxonomy /home/ramon/savont_db/greengenes2-2024.09/new_db/taxdump --threads 4 --taxonomic_rank S
```

**Code Block 17**
```bash
savont-analyse profile2CAMI --input_dir /home/redman/jaar4/runs_out_greengenes2/savont_tax --tool savont --sampleID simdata_rep0
```

**Code Block 18**
```bash
savont-analyse profile2CAMI --input_dir /home/redman/jaar4/runs_out_greengenes2/sim_v2_data/rep0/ --tool wf-16s --sampleID simdata_rep0
```

**Code Block 19**
```bash
savont-analyse profile2CAMI --input_dir /home/redman/jaar4/runs_out_greengenes2/ --tool nanosim --sampleID simdata_rep0
```

**Code Block 20**
```bash
opal.py -g ground_truth.profile abundance.profile wf-16s.profile -o /home/redman/jaar4/runs_out_greengenes2/opal
```
