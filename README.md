# Eukaryotyping
This repository contains the algorithms first described by Barratt, Park and colleagues in the manuscript that can be accessed [here](https://doi.org/10.1017/S0031182019000581).

"Eukaryotyping" refers to the analysis performed using the ensemble of similarity-based classification algorithms developed by Dr Mateusz Plucinski and Dr Joel Barratt. This analysis approach was initially designed to tackle complex *Cyclospora cayetanensis* genotyping datasets, though we are currently applying these algorithms to other genotyping challenges. This method constitutes a type of unsupervised [machine learning](https://en.wikipedia.org/wiki/Machine_learning), and was developed to adress three issues: (1) the common occurrence of missing data in genotyping datasets (e.g. such as a situation where 2 or 3 out of 6 MLST markers fail to amplify for a subset of your specimens), (2) the use of MLST methods in the context of sexually reproducing populations, where even closely related individuals may not possess the same genotype (and may be heterozygous) due to chromosomal crossover and random reassortment of chromosomes as occurs during meiosis and, (3) the issue of analyzing specimens which may be extremely complex, potentially representing mixed populations of individuals. Ever try to perform a phylogeny or generate a cluster dendrogram in a situation where for one MLST marker you detect one haplotype, at another you detect three, at another you detect four and another you detect two - in the same specimen? This is essentially what we deal with when we attempt to genotype *Cyclospora cayetanensis* directly from human stool. It gets extremely complicated.

Using the "Eukaryotyping" method you do not need to exclude specimens with a partial genotype (within reason). For example, if you have a large MLST dataset where some specimens have only half of their MLST markers successfully sequenced, this method will usually cope just fine provided the markers that were successfully sequenced are sufficiently diverse. If you have a complex dataset where some specimens are heterozygous or homozygous at any marker, this method will also cope just fine. Even if your dataset represents a population of mixed genotypes (i.e., such as a stool specimen containing a set of haplotypes representing a sexually reproducing population of *Cyclospora*), this method should cope just fine. The end result is a distance matrix where the values (all between 0 and 1) represent how closely related every possible pair of specimens is. A value of "0" indicates that the specimens are identical, and a value of "1" indicates the specimens are not related.

## Using these scripts
The R scripts provided here constitute the "Eukaryotyping" R code that was initially prepared for analysis of our complex *Cyclospora cayetanensis* genotyping data. These files were written by Dr Mateusz Plucinski who is a co-author of the [manuscript](https://doi.org/10.1017/S0031182019000581) that first describes the algorithms.

#### Importing data
Five files are provided here (six if you include the README file). One of these files (import_data_V2.r) must be modified by the user depending on how the user wishes the haplotypes to be defined, and depending on the name of the file/data sheet containing the haplotype data. An example of how the haplotype data sheet must be formatted is provided in this file: "Example_haplotype_data_sheet.txt". This format must be adhered to or the scripts will not run correctly. The names of the markers used can be changed by the end user, but the format must not change.

#### Naming markers
The names of the markers (i.e., the "locinames" variable) must be modified in the "import_data_V2.r" script (around line 6) according to a system defined by the user. For example, if the user has several large genes/loci that they wish to split into smaller parts, there is the option to do so by defining them as GENE_1_PART_A, GENE_1_PART_B, GENE_1_PART_C etc.., and the base name (i.e., the locinames_base variable - around line 9) of this marker is then defined as "GENE_1" (take a look at "import_data_V2.r" and you will get the idea). If the end user does not wish to divide a given marker (e.g. "MARKER_Z") into sub-markers, then the base name is simply "MARKER_Z" - the same as the "locinames" variable. Ultimately, it is the end user that develops a system for defining haplotypes that best suits their purpose.

#### Caveats for dividing markers into smaller parts
There are some caveats that must be considered when dividing up markers in the way described above, and this would depend on the entropy of the marker and the number of markers being examined. Ultimately, if the marker is incredibly hypervariable (e.g., lets say, there are 50 haplotypes in your population of 60 specimens), very few links will be identified at this locus by the algorithms. In this case, the end user may wish to include additional markers possessing fewer haplotypes (i.e., markers of a lower entropy), or split this hypervariable locus into smaller pieces such that the number of haplotypes at each piece is drastically reduced. Alternatively, the end user may wish to exclude such hypervariable markers from this analysis.

#### Defining the ploidy of each marker
The user also needs to define the ploidy of each marker in the "import_data_V2.r" script - around line 11. For example, if your markers are mitochondrial, then you would indicate "1" for the ploidy as the mitochondrial genome is inherited via an extranuclear mechanism. If your organism is a sexually reproducing eukaryote with 2N chromosomes, any nuclear/chromosomal loci must have their ploidy set to "2". The ploidy must be defined for all markers previously included in the "locinames" variable (and in the same order) in the "import_data_V2.r" script (not in the the same order as the base names - if you do this for only the base names, the script will fail to run). The scripts provided here require ***a combination*** of nuclear and mitochondrial markers in order to run correctly.

#### Filtering to exclude specimens with insufficient data
While the "Eukaryotyping" ensemble does a good job at compensating for specimens with partial data (e.g., where not all markers in a given MLST panel were succesfully sequenced for a subset of specimens), the end user may wish to set a cutoff for the minimum amount of data required before a specimen is excluded from the analysis completely. The user can set very specific inclusion criteria by providing a set of filtering parameters for their genotyping dataset. This information is contained in the "cleandata" variable within the "import_data_V2.r" script - around line 54. This variable will define the inclusion criteria for your specimens. For instance, if 1 out of 8 typing markers was successfully sequenced for a given specimen, you may wish to exclude that specimen as this is not enough information to make an accurate cluster assignment. You may wish to only include specimens that sequenced successfully for at least markers 1, 2 and 8, plus at least one other marker, or only include specimens that sequenced for at least markers 1 to 4. These combinations are up to the user to decide upon. When modified correctly, the script will automatically exclude specimens that fail to meet the criteria defined. Note that these loci are defined numerically, and these numbers correspond to the precise order the markers are listed in, within the "locinames_base" variable of the "import_data_V2.r" (around line 9).

### Determining an epsilon value for Plucinski's [Naive Bayes](https://en.wikipedia.org/wiki/Naive_Bayes_classifier) classifier 
Next, the user might consider modifying the "euk_bayesian_fulldataset_V2.r" script to provide a user-defined epsilon value. This value reflects the rate of missing data, or how often markers or haplotypes that ***should*** be present in a specimen are not detected due to amplification and/or sequencing failures. We determined this value to be 0.3072 based on a series of deep amplicon sequencing (Illumina) experiments, performed on a large *Cyclospora* dataset, considering 8 genotyping markers. The user is open to run the algorithm using this epsilon value (this will possibly/probably provide a sensible answer), or the user may wish to run experiments to determine this value for their own dataset. For the sake of accuracy, we recommend that users determine an epsilon value that is optimised for their dataset.

### Starting the analysis
Once the "import_data_V2.r" and  "euk_bayesian_fulldataset_V2.r" are modified by the user to reflect their data, calculation of two matrices by each algorithm is commenced by running the script "run.r". This script first initiates the importation of data using the "import_data_V2.r" script. Next, calculation of the Bayesian matrix is initiated by "run.r" which runs the script "euk_bayesian_fulldataset_V2.r". This is the Bayesian algorithm developed by Dr Mateusz Plucinski. Once the Bayesian matrix is calculated, the "run.r" script then initiates calculation of the heuristic matrix by running the script:  "euk_heuristic_fulldataset.r". This script is the heuristic algorithm developed by Dr Joel Barratt, which probably constitutes a kind of [mixture model](https://en.wikipedia.org/wiki/Mixture_model) that performs an [unsupervised learning](https://en.wikipedia.org/wiki/Unsupervised_learning) task. The script called "euk_heuristic_fulldataset.r" does not need to be modified by the end user at any point (unless the user wishes to change the number of cores used to run the script in parallel - see below). It may seem obvious, but please ensure that all scripts and the necessary haplotype data sheet are in the same directory or the scripts will not function correctly.

### Output
These scripts will generate three csv files: the Bayesian matrix, the heuristic matrix, and the ensemble matrix. In the first description of this algorithm (see Supplementary material 1 - located [here](https://www.cambridge.org/core/journals/parasitology/article/genotyping-genetically-heterogeneous-cyclospora-cayetanensis-infections-to-complement-epidemiological-case-linkage/0C51FBFFB172DF50357C1D171E9B8657#fndtn-supplementary-materials)), the output values of the Bayesian matrix and the heuristic matrix were simply averaged to generate the ensemble matrix. However, we later found that the predictive value of this method was improved using an alternative normalization approach. This improved approach involves mapping the distribution of distances generated by the Bayesian algorithm to the empiric distribution of distances generated by the heuristic algorithm. The mean of these mapped pairs is then used to generate the final ensemble matrix of pairwise distances. The scripts provided here perform the latter (i.e., improved) normalization procedure.

### Visualisation
Finally, the ensemble matrix is clustered and visualized as a dendrogram (or some other way). We prefer to use Wards clustering method and Manhattan distances, as described in the [manuscript](https://doi.org/10.1017/S0031182019000581). We have also enjoyed using [MicrobeTrace](https://microbetrace.herokuapp.com), which is a great tool for visualization of paired distances/matrices, developed by [Tony Boyles](https://github.com/aaboyles).

### Parallelization
Parts of two of these scripts have been parallelized to reduce the time of computation: "euk_bayesian_fulldataset_V2.r" and "euk_heuristic_fulldataset.r". The number of cores used for each of these scripts is currently set to 12 and this setting should be adjusted depending on resources avaible on the users system. The setting can be adjusted for the "euk_bayesian_fulldataset_V2.r" script at around line 123. For the "euk_heuristic_fulldataset.r" script, this setting can be adjusted at around line 96.

### Notes
Theoretically, the more specimens from your population included in the analysis, and the more markers analyzed, the more accurate the clustering should be. This is a pop-gen tool and works best when many members of your test population are represented in the dataset. We are currently performing experiments to determine the optimal sample sizes and other factors that impact the acuracy of clustering.

***IMPORTANT: These scripts will not run correctly in R console or R studio. Run them in a terminal window only.***

### License agreement
The manuscript describing this method and the scripts provided here have been distributed under the terms of the Creative Commons Attribution-NonCommercial-ShareAlike [licence](http://creativecommons.org/licenses/by-nc-sa/4.0/), which permits non-commercial re-use, distribution, and reproduction in any medium, provided the same Creative Commons licence is included and the original work is properly cited.

### If you make use of these files and find our work helpful, please don't forget to cite us:

```Barratt, JLN, S Park, FS Nascimento, J Hofstetter, M Plucinski, S Casillas, RS Bradbury, MJ Arrowood, Y Qvarnstrom, E Talundzic (2019) Genotyping genetically heterogeneous Cyclospora cayetanensis infections to complement epidemiological case linkage. Parasitology:1–9 doi:10.1017/S0031182019000581```
