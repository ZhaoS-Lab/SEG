# SEG
The program SEG analyzes the log2-ratios from whole genome sequencing or microarray (CGH or SNP array) data.  SEG first identifies change-points among the log2-ratio series along a chromosome and then identifies segments that are significantly amplified or deleted. SEG was developed by Dr. Mucheng Zhang, Dr. Deli Liu, Rr. Jie Tang and others in Dr. Shaying Zhao's lab.  Please contact szhao@uga.edu for any issues regarding SEG
 
SEG USAGE
SEG analysis input data in two steps. Step 1 in only needed for first-time run.
				
STEP 1: Calculate standard deviations and the mean log2-ratio of aCGH data, chromsome by chromosome, as well as across the 
	whole genome. 
    
    USAGE: ./stat input_step1 output_step1 [#_of_chromosome]
	Option: number of chromosomes (default = 24)

    Input_step1 Format:
        Title: name of each column
        data lines: chromosome_name probe_position log2-ratio
    Output_step1 Format:
        Title: name of each column
        data lines: chromosome_name num_of_probes chromosome_std
 	
Note:   The delimiter should be Space or Tab.    
	The input file must be sorted by chromosome_name and probe_position (in increasing order).
    	
STEP 2: Smooth data and Do segmention to find CNVs (Copy-Number Variations)
    USAGE: ./seg input_step1 output_step1 output_step2 [-q fdr] [-m min-mean] [-s min-size]
    	[-w init-size][-d deviation][-k #DP-segment][-n 1]
    Input_step1 : 
    	the same file as STEP1
    output_step1:
        the output from STEP1
    Parameters:
	
    	-q fdr: set desired FDR control level in Benjamini-Hochberg procedure (default value=0.005) 
    	-m min-mean: minimum mean log2-ratios of segments to be labeled as significant (1 or-1 in output_step2 column 6,default=genomic_std from step 1)
    	-s min-size: minimum number of probes of segments to be labeled as significant. default=5 (probes). 
    	-w init-size: initial probe # in pre-assigned segments to detect change-points. default =6 (probes).
    	-d deviation: std value used to lable segments. (default=genomic-std).
    	-n 1: Do a linear transformation for input-data, so that the mean log2-ratio of whole data is 0
    	-k Km: number of pre-assigned segments in a window for DP. SEG will detect Km-1 change-points in successive Km*W probes. (default=101)
            
	*NOTE: Computation-INTENSIVE parameter. The Higer, the better. If the probe number(M) in the largest chromosome is not huge (<20k), it is strongly recommended 
	that set the value of Km and w (with options -k and -w) such that Km*W>=M. Then DP will be applied to the whole chromosome, which is most
	accurate/sensitive.

    OutPut Format:
       Column 1-5: chr_name start_index end_index start_position end_position
       Column 6-10: label mean*sqrt(np) mean np(number of probe) p-value
       
Alternative of STEP2: Step2 may be run in two stages: DP process change-points detection and segments labeling. 
	If large k (>= 1000) is applied and users want to test different sets of parameters 
without doing computation-intensive DP process each time, this alternative is recommended.

STEP 2A: use DP to detect (candicate) change-points.
    Usage:
 	./segA input_step1 output_step1 output_step2A [-w init-size][-k DP_segments][-n 1]
    
    output format:
        chr-name start-position log2-ratio label
    	
	Where labels(0 or 1) indicate positions of detected change-points.Other parameter are same as STEP2.

STEP 2B: Label the segments with customized parameters.
    Usage: 
        ./segB input_step1 output_step1 output_step2A output_step2B [-q fdr] [-m min-mean] [-s min-size][-d deviation]
        
    output_step2A is output of STEP 2A. Other parameters and options are same as STEP2.
	  
----------------- Tips ----------------------------------------------------

If User want to compare the results across a group of samples,then
    (1) the option
		-d pooled-deviation
	is recommended.
     	The pooled standard deviation will be used in labeling every sample within the group.
    
    (2) if Users want to use the same cut-off value of segment p-values for labeling(not recommended), 
	then use the option
        	-f 0 -q Q0        
	to skip B-H procedure and use the option and specify Q0 as the cut-off value.
        
---------------------- EXAMPLEs --------------------------------------------------

using "chr22-input-data" in the demo-data folder

step 1: 
 ./SEG_executables/stat ./demo-data/chr22-input-data chr22-input-data_out 1

step 2:
 ./SEG_executables/seg ./demo-data/chr22-input-data chr22-input-data_out chr22-input-data_out_seq  -q 0.1 -m 0.525018 -s 5 -w 6 -k 101 -n 1

using "K9-aCGH_data" in the demo-data folder

Dog has 39 chromosomes
Here is format of inputfile 
CHROMOSOME CHR_POSITION LOG2_RATIO
chr1 3212001 0.1090
chr1 3225657 -0.1221
chr1 3238689 -0.1199
chr1 3243589 -0.0846
chr1 3248064 -0.1770

step 1: 
 ./SEG_executables/stat ./demo-data/K9-aCGH_data K9-aCGH_data.stat 39 
	
step2:
./SEG_executables/seg ./demo-data/K9-aCGH_data K9-aCGH_data.stat K9-aCGH_data_segOutput -k 4000

chromsome 1, the largest chromosome in dog, has 20820 probes.
when Km=4000, Km*w=4000*6=24000>20820, the DP will be applied on whole chromosomes. 
