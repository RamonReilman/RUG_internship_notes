# Title: Greengenes2 unifies microbial data in a single reference tree

- Author: Daniel McDonald, Yueyu Jiang, Metin Balaban, Kalen Cantrell, Qiyun Zhu, Antonio Gonzalez, James T. Morton, Giorgia Nicolaou, Donovan H. Parks, Søren M. Karst, Mads Albertsen, Philip Hugenholtz, Todd DeSantis, Se Jin Song, Andrew Bartko, Aki S. Havulinna, Pekka Jousilahti, Susan Cheng, Michael Inouye, Teemu Niiranen, Mohit Jain, Veikko Salomaa, Leo Lahti, Siavash Mirarab, Rob Knight.
- Year: 2024
- Type: Article describing the [Greengenes2](/02_Tools/Greengenes2.md) reference database
- Reference: (McDonald et al., 2024)
- Date Reviewed: 13-10-2026

## Research question
> What question is this article trying to ask?

16S rRNA studies and shotgun metagenomics studies usually give different results for the same samples, usually blamed on the PCR amplification bias. The article asks whether a single unified reference source, with genomic and 16S rRNA data, can make these methods work together.
## Main argument
> Main claim or statement / thesis

- A single reference tree can be built that unifies genome and 16S rRNA records
- When 16S rRNA and shotgun metagenomic data from the same samples are analyzed using the same tree, they agree much more with eachother.
## Methods
> What methods are used to answer the question

Greengenes2 was built with an iterative approach, starting from the whole-genome catalog of 15953 bacterial and archeal genomes sampled evenly from NCBI. A phylogenomic tree was constructed from this catalog by summarizing the evoluationary trajectory of 380 global marker genes, using uDance. 16S rRNA sequences were then inserted into this, this unified both data types into 1 tree.
## Key finding
> What are the key findings of this article?
- 16S rRNA and shotgun metagenomics data generated from the same samples agree far more closely. When analyzed using the same unified tree, compared to when each sample is analyzed with its own seperate database.
## Strengths
> What are the strengths of this article?

## Limitations
> What are the limitations of this article?

## Important quotes
> Place to put important quotes found in the article (Include page number)

## Relevance to my project
> How does this article relate to my own project?

Greengenes2 is one of the reference databases that both Savont and wf-16s can use, which is the entire reason it was chosen for this project's comparison. Its taxonomy format (`d__Bacteria;p__...;s__...`) and species naming conventions are used in my custom TAXid database, simulated data, and CAMI truth profiles.
## Related literature
> Other papers/books related / mentioned by this article (link to their own article review)

## Open questions
> What remains unanswered?