# Computing matrix 
computeMatrix reference-point \
  -S Pol2_ifny_merged.bw \
  -R STAT1_STAT3_shared.bed STAT1_unique.bed STAT3_unique.bed \
  --referencePoint center -b 2000 -a 2000 \
  --skipZeros \
  --numberOfProcessors 4 \
  -o matrix_pol2_ifny_merged.gz \
  --outFileSortedRegions sorted_regions.bed
  
