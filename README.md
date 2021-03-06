# Saturated mutagenesis of NER proteins via chimeragenesis

Note: This workflow utilizes a singularity image file containing various `R` and `python` scripts. The pre-built image file can be downloaded by direct link below. The code base for the image file is located [here](https://github.com/cory-weller/Xmera).

## Install Singularity
Instructions included at this [`gist`](https://gist.github.com/cory-weller/ae515627436596f7e82d96864df134aa)
```bash
wget https://gist.githubusercontent.com/cory-weller/ae515627436596f7e82d96864df134aa/raw/install-singularity.sh
bash ./install-singularity.sh
```

## Retrieve singularity image file
```bash
cid="77DD71E598E5B51B"
key="ANaVkMo47BvD0G8"
wget -O 'Xmera.sif' "https://onedrive.live.com/download?cid=${cid}&resid=${cid}%2119128&authkey=${key}"
# sha512sum 7fc67fb906fa4cfe1d57ed1bc1d3fd7061de1e6641c2943ffae2063ac7c9c0404cce609b70ada013394a2664300d716921797a9a182a5839c57567f52c9b301a
chmod +x Xmera.sif
```

## Generate codon-shuffled sequences
Below code block generates [`XPA-shuffled.fasta`](XPA/XPA-shuffled.fasta)
```bash
(
# Generate Sequence
cd XPA/ && \
../Xmera.sif shuffle.R \
    XPA-cds.fasta \
    XPA-cds.fasta \
    ../data/external/Scer-codons.tsv && \
    fold -w 60 XPA-cds.min.fasta \
    > XPA-min-homology.fasta && rm XPA-cds.*.fasta

# Remove homopolymers
../Xmera.sif homopolymers.py \
    XPA-min-homology.fasta \
    XPA-cds.fasta \
    > XPA-shuffled.fasta && rm XPA-min-homology.fasta
)
```

Below code block generates [`ERCC1-shuffled.fasta`](ERCC1/ERCC1-shuffled.fasta)
```bash
(
# Generate Sequence
cd ERCC1/ && \
../Xmera.sif shuffle.R \
    ERCC1-cds.fasta \
    ERCC1-cds.fasta \
    ../data/external/Scer-codons.tsv && \
    fold -w 60 ERCC1-cds.min.fasta \
    > ERCC1-min-homology.fasta && rm ERCC1-cds.*.fasta

# Remove homopolymers
../Xmera.sif homopolymers.py \
    ERCC1-min-homology.fasta \
    ERCC1-cds.fasta \
    > ERCC1-shuffled.fasta && rm ERCC1-min-homology.fasta
)
```



## Generate Repair Templates 

Below code block generates [`XPA-DMS-RTs.fasta`](XPA/XPA-DMS-RTs.fasta)

```bash
./Xmera.sif buildMutagenicRTs.py \
    --first XPA/XPA-shuffled.fasta \
    --second XPA/XPA-cds.fasta \
    --length 200 \
    --codons data/external/Scer-codons.tsv \
    --upstream XPA/XPA-upstream.fasta \
    --downstream XPA/XPA-downstream.fasta \
    > XPA/XPA-DMS-RTs.fasta
```

Below code block generates [`ERCC1-DMS-RTs.fasta`](ERCC1/ERCC1-DMS-RTs.fasta)
```bash
./Xmera.sif buildMutagenicRTs.py \
    --first ERCC1/ERCC1-shuffled.fasta \
    --second ERCC1/ERCC1-cds.fasta \
    --length 200 \
    --codons data/external/Scer-codons.tsv \
    --upstream ERCC1/ERCC1-upstream.fasta \
    --downstream ERCC1/ERCC1-downstream.fasta \
    > ERCC1/ERCC1-DMS-RTs.fasta
```


Below code block generates [`XPA-indel-RTs.fasta`](XPA/XPA-indel-RTs.fasta)
```bash
./Xmera.sif parseHGVS.py \
    XPA \
    shuffled \
    cds \
    indels \
    data/external/Scer-codons.tsv \
    200 | fold -w 80 \
    > XPA/XPA-indel-RTs.fasta
```


Below code block generates [`ERCC1-indel-RTs.fasta`](ERCC1/ERCC1-indel-RTs.fasta)
```bash
./Xmera.sif parseHGVS.py \
    ERCC1 \
    shuffled \
    cds \
    indels \
    data/external/Scer-codons.tsv \
    200 | fold -w 80 \
    > ERCC1/ERCC1-indel-RTs.fasta
```

Below code block generates frame shifts in XPA, saving as [`XPA-frameshift-RTs.fasta`](XPA/XPA-frameshift-RTs.fasta):
```bash
./Xmera.sif Xmera/src/buildFrameShiftRTs.py \
    --first XPA/XPA-shuffled.fasta \
    --second XPA/XPA-cds.fasta \
    --length 200 \
    --upstream XPA/XPA-upstream.fasta \
    --downstream XPA/XPA-downstream.fasta \
    --frameshift -1 \
    --prepend XPA \
    --wrap > XPA/XPA-frameshift-RTs.fasta

./Xmera.sif Xmera/src/buildFrameShiftRTs.py \
    --first XPA/XPA-shuffled.fasta \
    --second XPA/XPA-cds.fasta \
    --length 200 \
    --upstream XPA/XPA-upstream.fasta \
    --downstream XPA/XPA-downstream.fasta \
    --frameshift -2 \
    --prepend XPA \
    --wrap >> XPA/XPA-frameshift-RTs.fasta

./Xmera.sif Xmera/src/buildFrameShiftRTs.py \
    --first XPA/XPA-shuffled.fasta \
    --second XPA/XPA-cds.fasta \
    --length 200 \
    --upstream XPA/XPA-upstream.fasta \
    --downstream XPA/XPA-downstream.fasta \
    --frameshift -3 \
    --prepend XPA \
    --wrap >> XPA/XPA-frameshift-RTs.fasta
```

Below code block generates frame shifts in ERCC1, saving as [`ERCC1-frameshift-RTs.fasta`](ERCC1/ERCC1-frameshift-RTs.fasta):
```bash
./Xmera.sif Xmera/src/buildFrameShiftRTs.py \
    --first ERCC1/ERCC1-shuffled.fasta \
    --second ERCC1/ERCC1-cds.fasta \
    --length 200 \
    --upstream ERCC1/ERCC1-upstream.fasta \
    --downstream ERCC1/ERCC1-downstream.fasta \
    --frameshift -1 \
    --prepend ERCC1 \
    --wrap > ERCC1/ERCC1-frameshift-RTs.fasta

./Xmera.sif Xmera/src/buildFrameShiftRTs.py \
    --first ERCC1/ERCC1-shuffled.fasta \
    --second ERCC1/ERCC1-cds.fasta \
    --length 200 \
    --upstream ERCC1/ERCC1-upstream.fasta \
    --downstream ERCC1/ERCC1-downstream.fasta \
    --frameshift -2 \
    --prepend ERCC1 \
    --wrap >> ERCC1/ERCC1-frameshift-RTs.fasta

./Xmera.sif Xmera/src/buildFrameShiftRTs.py \
    --first ERCC1/ERCC1-shuffled.fasta \
    --second ERCC1/ERCC1-cds.fasta \
    --length 200 \
    --upstream ERCC1/ERCC1-upstream.fasta \
    --downstream ERCC1/ERCC1-downstream.fasta \
    --frameshift -3 \
    --prepend ERCC1 \
    --wrap >> ERCC1/ERCC1-frameshift-RTs.fasta
```

## Combine into single fasta
```bash
./Xmera.sif Xmera/src/addPrimers.py XPA/XPA-indel-RTs.fasta 301 > NER-RTs.txt
./Xmera.sif Xmera/src/addPrimers.py XPA/XPA-DMS-RTs.fasta 302 | grep "SNP=True" >> NER-RTs.txt 
./Xmera.sif Xmera/src/addPrimers.py XPA/XPA-DMS-RTs.fasta 303 | grep "SNP=False" >> NER-RTs.txt
./Xmera.sif Xmera/src/addPrimers.py XPA/XPA-frameshift-RTs.fasta 304 >> NER-RTs.txt
./Xmera.sif Xmera/src/addPrimers.py ERCC1/ERCC1-DMS-RTs.fasta 305 | grep "SNP=True" >> NER-RTs.txt
./Xmera.sif Xmera/src/addPrimers.py ERCC1/ERCC1-DMS-RTs.fasta 306 | grep "SNP=False" >> NER-RTs.txt
./Xmera.sif Xmera/src/addPrimers.py ERCC1/ERCC1-frameshift-RTs.fasta 307 >> NER-RTs.txt
./Xmera.sif Xmera/src/addPrimers.py ERCC1/ERCC1-indel-RTs.fasta 308 >> NER-RTs.txt

wc -l NER-RTs.txt 
# 14402
```