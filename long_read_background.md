# Long-read sequencing background
## What can I do with it?
The most common applications of long read sequencing is to be able to directly study regions of the genome that are still problematic for 2nd generation technologies, i.e. highly repetitive regions such as the pericentromeric heterochromatin, multi-copy gene families such as MHC genes, regions of structural variation, or sequences known to be affected by PCR bias. Previously the high cost of 3rd generation long read sequencing limited its application to organisms with small genomes. However, improvements in data output and error rate has allowed accessible sequencing of more complex genomes. Recent examples the kinds of questions you can answer using long reads:

-	Characterize structure of difficult to assemble regions such as tandem repeat arrays (e.g. satellite DNA) or other repeat-dense areas such as centromeres and telomeres
  -	Ultra-long Nanopore reads used to describe properties of satellite arrays in grass pea plant species with large genome enriched for satellites (6 Gb) (Vondrak 2019)
  -	PacBio used to characterize structure of complex satDNA loci in D. melanogaster (Khost 2017)
-	Detecting structural variation and copy number variation
  -	Sequenced several great ape genomes using high-coverage long reads plus Illumina sequencing and identified structural variants unique to each lineage, as well as compared the differences in repetitive element landscapes (Kronenberg 2018)
  -	Identify complex structural variants such as nested SVs, inverted tandem duplications, or inversions flanked by indels that are difficult to find with short reads (Sedlazeck 2018)
-	Use circular consensus sequencing protocol (multiple sequencing passes over single template) for PacBio to generate extremely accurate long reads to identify SNPs, indels, and structural variants in human population data (Wenger et al 2019)
- Use MinION to sequence in the field in real time, such as in disease outbreaks

## What do I need?
One important consideration when designing a long-read sequencing experiment is whether you will be using only long reads for your de novo assembly (long read only) or whether supplementary Illumina reads will be used as well (hybrid assembly). Due to lowering costs and improved error rate of new long read technologies, many contemporary sequencing projects have adopted the long read only approach, and are able to obtain more complete assemblies than hybrid. However, some groups  
-	If using long reads only:
  - PacBio: to obtain accuracies close to Illumina, need very high coverage (>90X)
  -	Nanopore: ~30X coverage is the rule of thumb for long read-only assemblies. However, still has accuracy issues and biases in homopolymer regions (see below)
    - One paper developed a protocol to enrich for ultra-long reads (up to 886kb), which at 5X coverage doubled the assembly contiguity

-	If doing hybrid assembly:
  -	Even low coverage long reads (5-10X) can greatly improve the contiguity of short read assemblies
  - At least ~60X Illumina coverage

## What assemblers can I use?
There are many to choose from, which can be overwhelming!
- *wtdbg2*: A *de novo* assembler useable for noisy ONT or PacBio reads. It uses a "fuzzy Brujin graph" approach where the genome is broken up into segments and similar segments are merged and connected, while still allowing mismatches and gaps. It does not perform initial error correction of reads, but claims to achieve "comparable" base accuracy to other assembler while being an order of magnitude faster. It is also possible to perform error correction using other tools (e.g. Canu), then pass corrected sequences to wtdbg2.
- *Minimap/miniasm*: Read mapper and assembler, respectively. Specialized for fast assembly of noisy ONT reads, especially in larger genomes. Note: does not attempt to assemble repetitive regions! Also, similar to wtdbg2, miniasm does not perform an initial error correction of reads like, resulting in lower per-base accuracy. Can also take error corrected reads similar to wtdbg2.
- *SMARTdenovo*: Another rapid assembler usable for ONT and PacBio. Like miniasm, it does not perform error-correction, instead doing an all-by-all alignment of raw reads. Consensus polishing tools (e.g. Quiver/Arrow for PacBio or Nanopolish for ONT) are required for accurate assembly.
- *Canu*: Assembler specialized for noisy long-read sequences, useable for PacBio or ONT. First finds overlaps between reads using a hashing algorithm (MHAP) for faster alignment, then generates a corrected consensus based on overlaps and performs an assembly using the corrected sequences. Due to the correction step, it is more computationally intensive than assemblers without a read correction step (e.g. wtdbg2 or miniasm), and can be about an order of magnitude slower. However, it does produce higher base accuracy assemblies compare to wtdbg2, miniasm, etc.
- *Falcon*: Specialized for PacBio. Diploid-aware, allowing phasing of genotypes with sufficient coverage. Specialized for larger genomes. Similar to Canu, performs an error correction step where smaller reads are aligned against largest subset of reads to generate corrected consensus reads that are used for assembly. In addition to primary contigs, also produces “haplotigs” in regions of divergent haplotypes.
- *Shasta*: ONT only, recently developed in response to heavy computational requirements for human ONT assembly. Developed to be fast (human assembly < 1 day) and more resilient to errors in homopolymer regions that ONT has trouble with. Loads every single base into memory all at once, which for large genomes can require a HUGE amount of processing power, e.g. 128 CPUs and ~1950 GB of RAM for a human genome at 60X coverage!!
- *MaSuRCa*: Useable for PacBio, ONT and Illumina data. Allows hybrid assembly of long reads: combines short Illumina reads together to form super-reads, which are then approximately aligned using k-mers (similar to MHAP algorithm) with long reads to produce long, accurate “mega-reads”. These reads are then used for assembly using a modified Celera assembler (CABOG). MaSuRCa has been used on difficult, repeat-dense genomes such as *A. tauschii*, a plant related to wheat.


## How do I correct the reads?
To compensate for the high error rates of the reads, need to either use self-correction or using short reads (e.g. from Illumina). Correction can occur before assembly on the reads themselves, on the finished assembly, or both.
- *Quiver/Arrow*: specialized for PacBio, generates a consensus using self-correction of PacBio reads: the longest subset of reads is corrected by aligning shorter reads. Used after assembly is constructed as a final polishing step, can be done in conjunction with other polishing software (e.g. Pilon).
- *Pilon*: Correction using short reads, performed on finished assembly. Input a BAM file of short reads mapped against genome, which allows Pilon to correct SNPs and indels, as well as fill gaps and identify local mis-assemblies.
- *Racon*: PacBio and ONT. Intended for use with assemblers that do not perform an initial error correction step (e.g. miniasm, SMARTdenovo). Can be used for correction of the reads or as final assembly polishing.
- *Nanopolish*: self-correction specialized for ONT data, run on finished assembly. Unlike other polishing programs, requires the raw signal-level data from ONT in addition to the output from the base-calling software. Base-called reads are mapped against the draft assembly, which is input to Nanopolish.

## Should I use PacBio or Nanopore?
Both PacBio and ONT are very active being developed, and as such the pros and cons of each technology can change rapidly. Currently, there is no clear-cut answer as to which technology is “better,” and which one you choose will depend on your needs and resources. However, there are several things to keep in mind:
-	Cost: PacBio sequencers are prohibitively expensive for most labs, making it necessary to rely on sequencing cores. ONT’s MinION sequencer, however, can be purchased for $1000, and produces a similar amount of data per flow cell.
-	Genome size: while long read sequencing is improving data output and creating more efficient assembly algorithms, and there have been several assemblies of complex genomes published, many protocols are still geared towards smaller genomes.
  -	For smaller genomes (e.g. bacteria, some invertebrates, etc.), either technology is fairly accessible and can be assembled using standard benchtop computers. ONT’s MinION is probably the best route due to the high output from a single flow cell and low cost.
  -	For larger genomes, likely need access to computing cluster to assemble in practical amount of time
-	ONT offers higher data output, but groups have reported error rate is still a problem and PacBio produces better results
-	Accuracy:
  -	One recent study found that even with custom trained base-calling model and polishing with Nanopolish, the max accuracy was 99.94% for ONT
  -	Unlike PacBio, ONT errors are not randomly distributed: it has trouble with long homopolymers due to uncertainty in base-calling. While recent base callers have improved this bias, some groups still report it as an issue.
  -	Most groups, even when using high coverage sequencing, still correct their assemblies with Illumina
-	Material: PacBio require much more input for sequencing
-	High error rate of MinION is fine for SV detection, but if looking at SNPs or indels or haplotype variation, accuracy is still problem
-	**Take-away: ONT produces longer reads and more data for less money, at the cost of an increased error rate; PacBio is more expensive and produces shorter reads, but has a better accuracy**

For our example assemblies and protocols, we will use **ONT reads***.

## Example assembly: *Drosophila melanogaster*
Which assembly process you use will depend on your organism and a variety of other factors, and each method has a wide variety of advanced usage. To provide a base-level of comparison, we performed an ONT assembly of *D. melanogaster*, an organism with a well-established reference genome, using several different approaches. The scripts used to generate the assemblies are provided.

Some facts about *D. melanogaster*:
- Genome size ~140Mb
- Relatively low repeat density
- 5 chromosomes (inc. Y), chromosome arms ~24Mb
- ~30X coverage from a single MinION run

| Assembly | N50	| Size	| # contigs	| Largest contig	| BUSCO complete	| Time to run |
| --- | --- | --- | --- | --- | --- | --- |
D. mel miniasm	| 7.90Mb	| 135.3Mb	| 264	| 24.9Mb	| 0.50%	| ~1 hr |					
D. mel wtdbg2	| 11.8Mb	| 135.8Mb	| 532	| 19.3Mb	| 43%	| 2 hrs |
D. mel wtdbg2 + canu corr	| 4.89Mb	| 127.4Mb	| 270	| 18.8Mb | 	59.50%	| 3 hrs |
D. mel shasta	| 253kb	| 106.2Mb	| 1080	| 1.49Mb	| 25%	| ~10 min |
D. mel shasta low cov	| 404kb	| 125Mb	| 1365	| 1.69Mb	| 23.1% |	~10 min |
D. mel Canu	| 3.85Mb	| 135.7Mb	| 278	| 18.3Mb	| 62%	| 34 hrs |

As evident from the above table, there is no clear-cut best assembly method. For example, the Canu assembler produces the highest base accuracies (based on the BUSCO completeness score) without any polishing; however, it takes a day and a half to assemble even a relatively small eukaryotic genome, and the N50 value is lower than other assemblies. The Shasta assembler is extremely fast, but is missing parts of the genome and has far lower N50 values (likely due to the lower than recommended coverage, ~30X vs 60X). *The wtdbg2 assembler seems to represent a good all-around method*

Following polishing, the BUSCO completeness scores improve as follows:

| Assembly | BUSCO complete	| Time to run |
| --- | --- | --- |
D. mel miniasm + pilon 1x	| 5% | ~2.5 days	|
D. mel wtdbg2 + pilon 1x	| 98.1% | ~20 hrs	|
D. mel wtdbg2 + pilon 2x	| 98.6% | ~6 hrs	|
D. mel wtdbg2 + racon	| 80% | ~3 hrs	|
D. mel wtdbg2 + racon -> pilon	| 97.6% | ~9 hrs	|
D. mel wtdbg2 + pilon -> racon	| 84% | ~1 hrs	|
D. mel shasta + pilon 1x	| 78% | ~10 hrs	|
D. mel Canu + pilon 1x	| 97.5% | ~10 hrs	|
