# HCP-Xcpengine

The XCP imaging pipeline (XCP system) is a free, open-source software package for processing of multimodal neuroimages. XCP Engine provides tools to take FMRIPREP output and perform the next steps required for many functional connectivity and structural analyses. And needs of [fmriprep output](https://github.com/JalilRT/HCPfmriprep).

![XCP](https://xcpengine.readthedocs.io/_images/tsRawToProcessed.png)

## How to run it

in the same way as fmriprep you can install it using

```
pip install xcpengine-container 
```

we highly recommend to use Docker or Singularity container

## Requirements 

Create the container with 
```
singularity build xcpengine_v122.sif docker://pennbbl/xcpengine
```

## Configuration

### BIDS

XCPengine does not necessarily require the data to be organized in BIDS, but it is useful when creating the Pipeline cohort file

### As in fmriprep, you should activate module
```
module load singularityce/3.5.3
```

## Now we can run it

### Single subject run

```
#!/bin/bash

export FSLDIR=/mnt/MD1200A/user/user/fsl_5.0.6
export PATH=${FSLDIR}/bin:${PATH}
. ${FSLDIR}/etc/fslconf/fsl.sh
export FSLPARALLEL=1
export LD_LIBRARY_PATH=${FSLDIR}/lib:${LD_LIBRARY_PATH}

DIR=$(PWD)
OUT_DIR=${DIR}/XCP/XCP_$(date +%d%b%Y)
TMP_DIR=${HOME_DIR}/tmp
FMRIPREP_DIR=$DIR/output_fmriprep/fmriprep
SIMG=$DIR/singularity_images/xcpengine_v122.sif
FULL_COHORT=${HOME_DIR}/XCP/cohorts/FuncCohort.csv
PIPELINE=${HOME_DIR}/XCP/designs/fc-36p_scrub_densikan.dsn

fsl_sub -s openmp,8 -R 16 -N func_xcp \
 singularity run -B /mnt:/mnt \
 ${SIMG} \
 -c $FULL_COHORT \
 -d $PIPELINE \
 -o $OUT_DIR \
 -r $FMRIPREP_DIR \
 -i $TMP_DIR \
 -t 2
```

### Several subjects 

```
#!/bin/bash

export FSLDIR=/mnt/MD1200A/lconcha/lconcha/fsl_5.0.6
export PATH=${FSLDIR}/bin:${PATH}
. ${FSLDIR}/etc/fslconf/fsl.sh
export FSLPARALLEL=1
export LD_LIBRARY_PATH=${FSLDIR}/lib:${LD_LIBRARY_PATH}

## Variables
HOME_DIR=/mnt/MD1200B/egarza/public
OUT_DIR=${HOME_DIR}/addimex_conn/derivatives/XCP/XCP_$(date +%d%b%Y)
TMP_DIR=${HOME_DIR}/tmp
FMRIPREP_DIR=${HOME_DIR}/addimex_conn/derivatives/fmriprep/output_16FEB2020_fsl/fmriprep
SIMG=/mnt/MD1200B/egarza/egarza/singularity_images/pennbbl_xcpengine-2020-01-15-bbc77518d878.img
FULL_COHORT=${HOME_DIR}/addimex_conn/data/sourcedata/XCP/cohorts/funcCohort2.csv
PIPELINE=${HOME_DIR}/addimex_conn/data/sourcedata/XCP/designs/fc-36p_spkreg.dsn

#select lines to run
nls=$(cat $FULL_COHORT | wc -l)
lines=$(seq 2 $nls)

for i in ${lines}; do

echo "running line $i"
TMPORT1=$(awk 'FNR == 1 || FNR == '$i' {print}' < $FULL_COHORT)
printf '%s\n%s' "$TMPORT1" > ${HOME_DIR}/addimex_conn/data/sourcedata/XCP/cohorts/TMP_COHORT_${i}.csv

TMP_COHORT=${HOME_DIR}/addimex_conn/data/sourcedata/XCP/cohorts/TMP_COHORT_${i}.csv

fsl_sub -s openmp,8 -R 12 -N func_xcp_s${i} \
 singularity run -B /mnt:/mnt \
 ${SIMG} \
 -c $TMP_COHORT \
 -d $PIPELINE \
 -o $OUT_DIR \
 -r $FMRIPREP_DIR \
 -i $TMP_DIR \
 -t 2

rm ${DIR}/XCP/cohorts/TMP_COHORT_${i}.csv

# random sleep so that jobs dont start at _exactly_ the same time
sleep $(( $RANDOM % 20 ))

done
```
