# TranSurVeyor

## Compiling

To be compiled, TranSurveyor requires g++ 4.7.2.

htslib 1.7 is included in the source, zipped. First of all, you should build it by using the provided script
```
./build_htslib.sh
```
If htslib does not build correctly, please refer to https://github.com/samtools/htslib

Then, run

```
cmake . && make
```

## Preparing the reference

The reference fasta file should be indexed by both bwa and samtools. For example, assuming the file is hg19.fa, you should run
```
bwa index hg19.fa
samtools faidx hg19.fa
```

## Preparing the BAM file

TranSurveyor has currently only been tested using BAM files generated by BWA MEM, therefore we recommend its usage. 
It should also be run through Picard FixMateInformation (http://broadinstitute.github.io/picard/command-line-overview.html#FixMateInformation);
in particular, it should have the MQ and MC tags.
Finally, the file should be sorted and indexed ad usual using samtools.

Supposing file.bam is the file resulting from the alignment:
```
java -jar picard.jar FixMateInformation I=file.bam
samtools sort file.bam > sorted.bam
samtools index sorted.bam
```

## Running 

Once the c++ code is compiled, TranSurveyor can be run. Python and libraries NumPy (http://www.numpy.org/), PyFaidx (https://github.com/mdshw5/pyfaidx) and PySam (https://github.com/pysam-developers/pysam) are required. Python 2.7, NumPy 1.10, PyFaidx 0.4 and PySam 0.12 are the recommended versions.

The bare minimum command for running TranSurveyor is 
```
python surveyor.py /path/to/bamfile /an/empty/working/directory /path/to/reference/fasta
```

An especially important parameter is maxTRAsize, which is the maximum transposition size for which TranSurVeyor will attempt predicting both breakpoints.
Higher values will require higher computational time. Currently the default is 1,000, but on our test datasets a value of 10,000 does not affect much the computational time.
Further experimentation is required, and we may change the default value in the future.

Other parameters which may be important are the number of threads and the location of the bwa and samtools executable.
```
python surveyor.py /path/to/bamfile /an/empty/working/directory	/path/to/reference/fasta --threads 40 --samtools /path/to/samtools --bwa /path/to/bwa --maxTRAsize 10000
```

After TranSurveyor has been successfully run, the calls can be retrieved with the command
```
./filter /path/to/working/directory
```

## Readme

Here is a sample transposition:

```
ID=1 BP1=F:chr5:135114483 BP2=R:chr12:59936709 DISCORDANT=66 SCORE=2.275862
ID=1 BP1=R:chr5:135114528 BP2=F:chr12:59937336 DISCORDANT=55 SCORE=3.666667
```

Each line is a junction, i.e. two regions which are far away on the reference but predicted to be adjacent in the sample. A transposition can be represented by two junctions.

ID is unique for each transposition: when two junctions share the same ID, they refer to the same transposition. Most junctions are paired: unfortunately, sometimes TranSurVeyor is only able to
prdict one junction. We are working to improve this.

BP1 and BP2 are the two breakpoints in a junction. They are in the format STRAND:CHR_NAME:POSITION. F stands for Forward and R stands for Reverse strand. TranSurVeyor predicts the BP1 to be 
breakpoint near the insertion site, while BP2 the breakpoint of the inserted sequence. For instance, the example transposition is predicted to be inserted into chromosome 5, between 135114483 and
135114528, and the predicted inserted sequence is chr12:59936709-59937336.

DISCORDANT is the number of discordant read pairs supporting the junction, and SCORE is the positive-to-negative score, i.e. a measure of confidence: the highest the score, the more confident the 
junction.

## Notes

TranSurveyor is a very young software, and it has only been used by its authours on a handful of datasets. 
If you should encounter problems, please write to e0011356@u.nus.edu ; we will do our best to both improve/fix both the software and the documentation.

