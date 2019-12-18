# Dual Threshold Optimzation (DTO)
Dual Threshold Optimization (DTO) is a method that sets the thresholds for TF binding and TF-perturbation response by considering both data sets together. DTO chooses, for each TF, the pair of (binding, response) thresholds that minimizes the probability that the overlap between the bound and responsive sets results from random gene selection

## Requirement
#### 1. Python libraries
Install the following Python libraries if not installed, using your favorite package management tool such `pip` or `conda`:
```
pip install scipy numpy pandas
```
#### 2. [Optional] Job queuing system
Job queuing system [SLURM](http://slurm.schedmd.com/documentation.html) is preferred to enable high-throughput computing. SLURM version tested: 
```
slurm-wlm 17.11.7
```  

## Input data
#### 1. Transcriptional responses to TF perturbation
A data matrix of the levels of transcriptional responses to TF perturbations, where columns represent the individual perturbed TFs and rows represent the genes. Each column is expected to be the output of differential expression analysis that compares the expression profiles of TF perturbation and pre-preturbation samples. The entry can either be represented as log fold-change or p-value.

#### 2. TF binding strengths
A data matrix of the binding strengths, where columns represent the individual assayed TFs and rows represent the genes. Each entry can be represented as occupancy level or statistical significance of the TF binding events occur on the gene's regulatory region (promoters and/or enhancers). For the gene that have multiple peaks at its regulatory region, you may use the sum or max of the binding strengths of those peaks.

#### Note
- Make sure that the column names of the two datasets have the same naming scheme to allow proper pairing of the samples. For example, use systematic name for all (such as ENSG00000256223) or use common name (ZNF10).
- Similarily make sure the row names are consistent for proper mateching of the gene sets.

## Code usage
#### 1. Run DTO on authentic datasets
```
python thresholdSearch.py -d <response_csv> --DE_decreasing <True/False> -b <binding_csv> --Bin_decreasing <True/False> --sbatch_loc <output_dir>/authentic_model/ --genes_universe <gene_universe_file> [--geneNames_file <gene_name_file>] [--run_local]
```
Required arguments:
- `-d <response_csv>` CSV file of transcriptional response levels of TF-perturbation. The response levels are based on differential expression or DE analysis on TF-perturbed expression profile vs. pre-perturbation profile. Those levels can be represented as absolute of log fold-changes (LFCs) or p-values.
- `-b <binding_csv>` CSV file of TF binding strengths. The binding strengths can be represented as occupancy levels of the peaks (e.g. heights of ChIP peaks) or statistical significances (e.g. p-values).
- `--DE_decreasing <True/False>` Use 'True' if the response levels should be raneked in descending order. Use 'False' if ranked in ascending order. For example: If using LFCs, set 'True' as higher absolute LFC represents stronger response. If using p-values, set 'False' as lower p-value represents stronger response.
- `--Bin_decreasing <True/False>` Use 'True' if the binding strengths should be ranked in descending order. Use 'False' if ranked in ascending order. For example: If using occupancy levels, set 'True' as higher occupancy level represents stronger TF binding. If using p-values, set 'False' as lower p-value represents stronger TF binding.
- `--sbatch_loc <output_dir>/authentic_model/` Directory for output data.
- `--genes_universe <gene_universe_file>` Gene universe list file, which contains all possible genes to be considered as the universe for calculating overlap statistics.

Optional arguments: 
- `--geneNames_file <gene_name_file>` CSV file of gene name conversion, if the systematic gene names in the data files are preferred to be converted into common gene names.
- `--run_local` If no SLURM is available, set this flag to run DTO in serial fashion on your local machine.

#### 2. Run DTO on randomized datasets
```
python thresholdSearch.py --random True -d <response_csv> --DE_decreasing <True/False> -b <binding_csv> --Bin_decreasing <True/False> --sbatch_loc <output_dir>/random_models/ --genes_universe <gene_universe_file> [--geneNames_file <gene_name_file>] [--run_local]
```
Required arguments:
- Same as above, except:
- `--random True` Use 'True' for running DTO on randomized input data. The DTO results on randomized data is used to create an empirical null distribution.
- `--sbatch_loc <output_dir>/random_models/` Make a new output directory for null model output.

#### 3. Summarize DTO results
```
python summarizeFinalResults.py -i <output_dir> [--run local]
```
Required arguments:
- `-i <output_dir>` Directory of DTO results including both authentic model and randomized models.

Optional arguments:
- `--run_local` If no SLURM is available, set this flag to run DTO in serial fashion on your local machine.

## Output data
