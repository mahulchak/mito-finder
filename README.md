# mito-finder
Assembling mitochondrial genomes from long molecule sequences

<b>Why mito-finder?</b>

While assembling the nuclear genomes, I have noticed that Falcon,canu, and all other amazing assemblers either completely fail to assemble the mitochonrial genome or they assemble the mitochondrial genome incorrectly. This is not surprising given that mitochondrial genomes are smaller than bulk of the input DNA (we throw away input DNA under 20kb, some throw away everythng under 30Kb). So instead of relying on the assemblers to assemble the mitochindrial genome for me, I decided to go via a different route.

<b>How does this work?</b>

We use an existing mitochindrial DNA as a bait to pull out all the uncorrected raw reads that map to the bait sequence. You may question why I use an existing mitochondrial sequence to get an unknown mitochondrial genome. The answer is simple. The aligner I use, <i>blasr</i>, allows up to 30% mismatch between the reads and the bait. So all you need is a mitochondrial genome that is 30% or less diverged from your mitochondrial genome. It may still work if the divergence is greater than 30%, you will just have to set the alignment cutoff accordingly.

<b>Workflow</b>

1. Run <i>blasr</i>.
  ```
   blasr raw_reads.fasta mito.fasta -bestn 1 -m 1 -nproc n > mito.m1
  
  ```
  raw_reads.fasta = the fasta file containing your raw long reads
  
  mito.fasta = the bait mitochondrial genome
  
  mito.m1 = the alignments in m1 format.
  
  see <i>blasr</i> documentation for the description of the other parameters.

2. Extract the reads the pass a length cutoff.
  ```
   awk '{if($8-$7>18000) print $1"\t"$8-$7"\t"$12}' mito.m1 |less
  
  ```
  this script uses 18kb as the cutoff for the mitochondria genome length. Replace this length by the lower limit of your mitochondrial genome size. E.g. all you know that your mitochondrial genome is longer than 15kb then use 15000. The output has three tab separated columns.
  
    READ_NAME BAIT_COVERAGE READ_LENGTH
  
3. Extract the read that has the highest length and then align the read to itself. This is done to ensure that we find the ends of the mitochondrial genome in the read. If the read is longer than the mitochondrial genome, then there is a chance that part of the read contains a duplicate of the mitochondrial genome. We want to remove this part from the read. For this you can use the nucmer utility from <i>MUMmer</i> or you can use another program that you like.

  ```
   nucmer -maxmatch --nosimplify mito_read.fasta mito_read.fasta
  
  ```
  Check "out.delta" to find the ends of the mitochondrial genome in your read. The positon where one end of your read aligns to another end, that signifies the two ends of the mitochondrial genome. If this does not make sense, think what happens when you start sequencing a circular DNA and eventually go past your sequencing start point. All this step is doing is identifying the sequencing start point at the two ends of the read. if you do not a match between start and end of the sequence, then you do not need to worry about changing the read.
  
4. Remove the extra DNA. Remove the sequence at the end that goes past the sequencing start point.

5. Polish the read using quiver. The polished read is your mitochondrial genome.
