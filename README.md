RKMH: Read classification by Kmers or MinHash
--------------------------------------------
Eric T Dawson  
June 2016

### What is it
rkmh performs MinHash (as implemented in [Mash](https://github.com/marbl/Mash)) on a set of reads
and reference genomes to classify each read in the set to its nearest match in the reference set.
It can also classify reads by directly comparing all of their kmers.

### License
MIT, but please cite the repository if you use it.

### Dependencies and build process
The only external dependencies should be zlib and a compiler supporting OpenMP. To download and build:  

                    git clone --recursive https://github.com/edawson/rkmh.git  
                    cd rkmh  
                    make  

This should build rkmh and its library dependencies (mkmh and murmur3).

### Classify 
rkmh requires a set of reads and a set of references in the FASTA/FASTQ format. Reads need not
be in FASTQ format.


To do the same, but use a MinHash sketch of size 1000 instead of just comparing kmers:  
```./rkmh classify -r references.fa -f reads.fq -k 10 -s 1000```

There's also now a filter for minimum kmer occurrence in a read set, compatible with the MinHash sketch.
To only use kmers that occur more than 10 times in the reads:  
```./rkmh classify -r references.fa -f reads.fq -k 10 -s 1000 -M 100```

There is also a filter that will fail reads with fewer than some number of matches to any reference.
It's availble via the `-P` flag:  
```./rkmh -r references.fa -f reads.fq -k 10 -s 1000 -M 100 -N 10```

### Call
Once you've identified which reference a set of reads most closely matches, you may want to figure out the differences between your set of reads
and your reference. `rkmh call` uses a brute-force approach to produce a list of candidate mutations / sequencing errors present in a readset.

```rkmh call -r ref.fa -f reads.fq -k 12 -t 4```  

We advise using only one reference during call, as it's relatively slow. For example, you might first classify your reads using `classify`, then
for the top classification in your set run `rkmh call`.

### Hash
You might want to see the hashes generated by rkmh for debugging purposes. To do so, use the `hash` command.

```rkmh hash -r ref.fa -f reads.fq -k 12 -s 1000``` 


### Other options
These are extra options for the `classify` and `hash` commands. Some of them are also applicable to `call`. For full usage, just
type `./rkmh` or `./rkmh <command>` at the command line to get the help message.


```-t / --threads <INT>               number of OpenMP threads to use (default is 1)```  
```-M / --min-kmer-occurence <INT>    minimum number of times a kmer must appear in the set of reads to be included in a read's MinHash sketch.```  
```-N / --min-matches <INT>           minimum number of matches a read must have to any reference to be considered classified.```  
```-I / --max-samples <INT>           remove kmers that appear in more than <INT> reference genomes.```  
```-D / --min-difference <INT>        flag reads that have two matches within <INT> hashes of each other as failing.```   
```-k / --kmer <INT>                  the kmer size to use for hashing. Multiple kmer sizes may be passed, but they must all use the -k <INT> format (i.e. -k 12 -k 14 -k 16...)```   
```-s / --sketch-size                 the number of hashes to use when comparing reads / references.```    
```-f / --fasta                       a FASTA/FASTQ file to use as a read set. Can be passed multiple times (i.e. -f first.fa -f second.fa...)``` 
```-r / --reference                   a FASTA/FASTQ file to use as a reference set. Can be passed multiple times (i.e. -r ref.fa -r ref_second.fa...)```   



### Performance
On a set of 1000 minION reads from a known HPV strain, rkmh is ~97% accurate (correctly placing the read in the right strain
of 182 input reference strains) and runs in <20 seconds. With the kmer depth and minimum match filters we're approaching 100% accuracy for about the same run time.
We're working on ways to improve sensitivity with further filtering and correction.

rkmh is threaded using OpenMP but the code should be considered minimally tuned. Hashing can handle around 250 reads/second (250 * 7kb means we're running over 1,500,000 basepairs / second).

### Getting help
Please post to the [github](https://github.com/edawson/rkmh.git) for help.
