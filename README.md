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
plotHeatmap -m matrix.gz -out Pol2_heatmap_over_STAT_peaks.png \
  --regionsLabel "STAT1+STAT3" "STAT1 only" "STAT3 only" \
  --colorMap RdBu_r \
  --zMin 0
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
  --bwfiles BMDM-pSTAT1-ctrl-MB46_S360.filtered.bw \
            BMDM-pSTAT3-ctrl-MB47_S361.filtered.bw \
            BMDM-pSTAT1-2hIFNy-MB47_S364.filtered.bw \
            BMDM-pSTAT3-2hIFNy-MB48_S365.filtered.bw \
  --binSize 10000 \
  --outFileName chipseq_correlation_bins.npz \
  --outRawCounts chipseq_correlation_bins_counts.tab \
  --numberOfProcessors $SLURM_CPUS_PER_TASK

# Plot correlation heatmap
plotCorrelation -in chipseq_correlation_bins.npz \
  --corMethod pearson \
  --skipZeros \
  --plotTitle "ChIP-seq correlation (10kb bins)" \
  --whatToPlot heatmap \
  --colorMap RdYlBu \
  --plotFile chipseq_correlation_bins_heatmap.pdf

conda deactivate

```
