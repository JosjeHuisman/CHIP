## Merging BigWigs
```bash
# merge BigWig into bedGraph
bigWigMerge BMDM-Pol2-ctrl-old-MB45_S356.filtered.bw BMDM-Pol2-ctrl-new-MB45_S357.filtered.bw Pol2_ctrl_merged.bedGraph
bigWigMerge BMDM-Pol2-2hIFNy-old-MB45_S358.filtered.bw BMDM-Pol2-2hIFNy-new-MB45_S359.filtered.bw Pol2_ifny_merged.bedGraph

# You need a chromosome sizes file for converting bedGraph into BigWig again
fetchChromSizes mm10 > mm10.chrom.sizes

# bedGraphToBigWig requires your BedGraph file to be sorted by chromosome name and start coordinate, in a case-sensitive wa
LC_COLLATE=C sort -k1,1 -k2,2n Pol2_ctrl_merged.bedGraph > Pol2_ctrl_merged.sorted.bedGraph
LC_COLLATE=C sort -k1,1 -k2,2n Pol2_ifny_merged.bedGraph > Pol2_ifny_merged.sorted.bedGraph

bedGraphToBigWig Pol2_ctrl_merged.bedGraph mm10.chrom.sizes Pol2_ctrl_merged.bw
bedGraphToBigWig Pol2_ifny_merged.bedGraph mm10.chrom.sizes Pol2_ifny_merged.bw

rm *.bedGraph


```
## Computing matrix 
```bash
computeMatrix reference-point \
  -S Pol2_ifny_merged.bw \
  -R STAT1_STAT3_shared.bed STAT1_unique.bed STAT3_unique.bed \
  --referencePoint center -b 2000 -a 2000 \
  --skipZeros \
  --numberOfProcessors 4 \
  -o matrix_pol2_ifny_merged.gz \
  --outFileSortedRegions sorted_regions.bed
  ```
## Creating heatmap from matrix
```bash
plotHeatmap -m matrix_pol2_ifny_merged.gz -out Pol2_IFNy_merged_heatmap__STAT_peaks.pdf --regionsLabel "STAT1+STAT3" "STAT1 only" "STAT3 only" --colorMap RdBu_r --zMin 0
```
## Correlation plot
```bash
#!/bin/bash
#SBATCH --job-name=chipseq_corr
#SBATCH --output=chipseq_corr_%j.out
#SBATCH --error=chipseq_corr_%j.err
#SBATCH --time=02:00:00
#SBATCH --mem=16G
#SBATCH --cpus-per-task=4

# Load your conda environment (adjust name if needed)
source ~/miniconda3/etc/profile.d/conda.sh
conda activate deeptools_env

# Run multiBigwigSummary in bins mode
multiBigwigSummary bins \
  --bwfiles \
    BMDM-pSTAT1-ctrl-MB46_S360.filtered.bw \
    BMDM-pSTAT3-ctrl-MB47_S361.filtered.bw \
    BMDM-pSTAT1-2hIFNy-MB47_S364.filtered.bw \
    BMDM-pSTAT3-2hIFNy-MB48_S365.filtered.bw \
    BMDM-IFNy-pSTAT1-MB30_S303.filtered.bw \
    BMDM-IFNy-pSTAT3-MB31_S304.filtered.bw \
    BMDM-ctrl-pSTAT1-MB28_S301.filtered.bw \
    BMDM-ctrl-pSTAT3-MB29_S302.filtered.bw \
    BMDM-Pol2-2hIFNy-new-MB45_S359.filtered.bw \
    BMDM-Pol2-ctrl-old-MB45_S356.filtered.bw \
    BMDM-Pol2-2hIFNy-old-MB45_S358.filtered.bw \
    BMDM-Pol2-ctrl-new-MB45_S357.filtered.bw \
  --binSize 10000 \
  --outFileName chipseq_correlation_bins.npz \
  --outRawCounts chipseq_correlation_bins_counts.tab \
  --numberOfProcessors $SLURM_CPUS_PER_TASK

# Run plotCorrelation on the output from multiBigwigSummary
plotCorrelation -in chipseq_correlation_bins.npz \
  --corMethod pearson \
  --skipZeros \
  --plotTitle "ChIP-seq correlation (10kb bins)" \
  --whatToPlot heatmap \
  --colorMap RdBu_r \
  --plotFile chipseq_correlation_bins_heatmap.pdf \
  --labels \
    pSTAT1_ctrl_R2 pSTAT3_ctrl_R2 pSTAT1_IFNy_R2 pSTAT3_IFNy_R2 \
    pSTAT1_IFNy_R1 pSTAT3_IFNy_R1 pSTAT1_ctrl_R1 pSTAT3_ctrl_R1 \
    Pol2_IFNy_new Pol2_ctrl_old Pol2_IFNy_old Pol2_ctrl_new

# Deactivate conda environment
conda deactivate


```
