# Title: Assessing taxonomic metagenome profilers with OPAL

- Author: Fernando Meyer, Andreas Bremges, Peter Belmann, Stefan Janssen, Alice C. McHardy, David Koslicki.
- Year: 2019
- Type: Article describing the [OPAL](/02_Tools/OPAL.md) software package
- Reference: (Meyer et al., 2019)
- Date Reviewed: 13-10-2026

## Research question
> What question is this article trying to ask?

The number of metagenomic profiling methods has grown rapidly, but there was no standardized way to systematically compare them against each other using relevant performance criteria. This article asks how such a standardized comparison can be created.
## Main argument
> Main claim or statement / thesis

- OPAL (Open-community Profiling Assessment tooL) is a software package that implements commonly used performance metrics, including those from the first CAMI challenge, together with creating visualizations, this allows for a standard comparison of taxonomic profiles.
## Methods
> What methods are used to answer the question

OPAL accepts one or more taxonomic profiles as input, and benchmarks these at different taxonomic ranks against the gold standard profile, and outputs the results as flat files, tables, plots, and an interactive HTML page. Users can customize the relative importance of metrics, this allows them to move priorities based on the users wants and needs.
## Key finding
> What are the key findings of this article?
- Most profilers that correctly predicted relative abundances did not necessarily also correctly detect the presence or absence of taxa, and vice versa, this shows that no single metric is sufficient to describe the performance of a profiler.
- Profilers mostly overestimated counts, and performance for most profilers dropped below family level.
## Strengths
> What are the strengths of this article?

## Limitations
> What are the limitations of this article?

## Important quotes
> Place to put important quotes found in the article (Include page number)

## Relevance to my project
> How does this article relate to my own project?

OPAL is the tool used directly in this project to compare Savont and wf-16s against the ground truth. Its CAMI-format and TAXid requirements are the reason a custom greengenes2 TAXid database had to be built, since no valid TAXids for greengenes2 could be found.
## Related literature
> Other papers/books related / mentioned by this article (link to their own article review)
- Sczyrba, A. et al. (2017). Critical Assessment of Metagenome Interpretation—A benchmark of metagenomics software.
## Open questions
> What remains unanswered?