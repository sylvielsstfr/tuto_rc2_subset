# =============================================================
# README.md
# author Sylvie Dagoret-Campagne
# creation date 2024-02-18
# update 2024-02-19
# =============================================================



export DISPLAY=

source /cvmfs/sw.lsst.eu/linux-x86_64/lsst_distrib/w_2024_07/loadLSST.bash
setup lsst_distrib -t w_2024_07
source ~/notebooks/.user_setups

# add the sample in current stack config
setup -j -r /sdf/home/d/dagoret/rubin-user/2024/rc2_subset
# and don't forget to put it in .user_setups

# notice that the pipeline yaml file is here
$DRP_PIPE_DIR/pipelines/HSC/DRP-RC2_subset.yaml

#to run the first step 
$DRP_PIPE_DIR/pipelines/HSC/DRP-RC2_subset.yaml#singleFrame 

# doc on htcondor
# https://developer.lsst.io/usdf/batch.html#ctrl-bps-htcondor
# https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#submitting-a-run

source batch_settings.sh 

eups list -s | grep LOCAL
eups list -s | grep ctrl_bps_htcondor

export BPS_WMS_SERVICE=lsst.ctrl.bps.htcondor.HTCondorService

allocateNodes.py -n 20 -c 120 -m 1-00:00:00 -q roma,milano -g 900 s3df

squeue -u dagoret
condor_status

### launch the pipeline the 2024/02/18 on butler with bps and htcondor
nohup bps submit bps_htc_hsc_20240219.yaml  > bps_hsc_20240219.out &



# interactive run https://developer.lsst.io/usdf/batch.html#ctrl-bps-htcondor
# srun --pty --cpus-per-task=8 --mem-per-cpu=4G --time=01:00:00 --partition=roma bash
nohup pipetask run --register-dataset-types -b $RC2_SUBSET_DIR/SMALL_HSC/butler.yaml -i HSC/RC2/defaults -o u/$USER/single_frame -p $DRP_PIPE_DIR/pipelines/HSC/DRP-RC2_subset.yaml#singleFrame -j 8 > pipetask.log	 &



#pipetask run -b /sdf/group/rubin/repo/oga/ -p ./processStar_auxtel_atmo_202401_v3.0.3_doGainsOPTC_rebin2.yaml -i LATISS/raw/all,refcats,LATISS/calib -o u/dagoret/spectro/w_2023_44_spec3.0.3/holo_2024_01_29 -d "exposure.day_obs=20240129 and visit_system=0 and exposure.seq_num in (293,294,316,317,323,324,330,331,337,338) and instrument='LATISS'" --register-dataset-types -j 8
