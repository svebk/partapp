PARTAPP - Human Pose Estimation
=====

This short documentation describes steps necessary to compile and run the human pose estimation model presented in the paper:

**Leonid Pishchulin, Micha Andriluka, Peter Gehler and Bernt Schiele  
Strong Appearance and Expressive Spatial Models for Human Pose Estimation  
In _IEEE International Conference on Computer Vision (ICCV'13)_, Sydney, Australia, December 2013**

If you have any questions, send us an email to leonid@mpi-inf.mpg.de with a topic "partapp code" and visit our project web page https://www.d2.mpi-inf.mpg.de/poselet-conditioned-ps
We may also run our model on your data.

Required Libraries
---

The following libraries are required to compile and run the code:

   - Qt (tested with version 4.8.2)
   - [Boost] (http://www.boost.org/) (tested with version 1.34)
   - Matlab (tested with Matlab 2008b)
   - MATLAB Compiler Runtime (MCR) (tested with R2013a (8.1)
   - Goolgle's [Protocol Buffers] (http://code.google.com/p/protobuf/) (tested with version 2.0.1rc1)

Compiling
---

This code was developed under Linux (Debian _wheezy_, 64 bit) and was tested only in this environment.  

1. Switch to the top level directory of the source code (the one with the script `run_partapp.sh`), issue commands:

    ```
    ln -s ./external_include/matlab-2008b include_mat  
    ln -s ./external_lib/matlab-2008b-glnxa64-nozlib lib_mat  
    ln -s ./external_include include_pb  
    ln -s ./external_lib lib_pb```

2. Add the full path of folder _include_mat_ to the `LD_LIBRARY_PATH` environment variable  
3. Download and install Matlab Compiler Runtime (MCR)
4. Point to your MCR by editing the file `src/libs/libPrediction/matlab_runtime.h`  
5. Issue commands `qmake-qt4 -recursive; make; ./compileMatlab.sh` in the _src/libs directory
6. Issue commands `qmake-qt4 -recursive; make` in the _src/apps_ directory

Testing
---

1. Issue the following commands in the _code_test_ subdirectory:
```
    unzip code_test.zip
    ../run_partapp.sh --expopt ./expopt/exp-code-test-local-app-model.txt --head_detect_dpm --part_detect_dpm --find_obj
    ../run_partapp.sh --expopt ./expopt/exp-code-test-poselets.txt --save_resp_test
    ../run_partapp.sh --expopt ./expopt/exp-code-test-full-model.txt --find_obj --eval_segments --vis_segments```

This will run local appearance model, compute poselet responses and finally run full model to estimate body parts on a provided image and visualize the results. Compare the image in the _./log_dir/exp-code-test-full-model/part_marginals/seg_eval_images_ with the image in _./images_result_

Running pose estimation experiments
---

   Our model requires that training and testing images contain single persons being roughy 200 px high.
   Download the experiments package from our project webpage: https://www.d2.mpi-inf.mpg.de/poselet-conditioned-ps
   Unpack the package in the separate directory EXP_DIR. Here is a short description of the contents:

   - _./images/LSP_ - upscaled images from the LSP dataset, so that every person is roughly 200 px high
   - _./expopt_ - configuration files which control diverse parameters of the system
   - _./log_dir/exp-lsp-local-app-model/class_ - pretrained AdaBoost models (classifiers)
   - _./log_dir/exp-lsp-local-app-model/dpm_model_ - pretrained DPM models
   - _./log_dir/exp-lsp-local-app-model/sparial_ - pretrained generic spatial model
   - _./log_dir/exp-lsp-local-app-model/test_dpm_unary/torso_ - responses of the DPM torso detector
   - _./log_dir/exp-lsp-train-torso/part_marginals_ - torso detections on the train set
   - _./log_dir/exp-lsp-poselets/class_ - pretrained AdaBoost poselet models
   - _./log_dir/exp-lsp-poselets/resp_train_ - poselet responses on train images
   - _./log_dir/exp-lsp-poselets/pred_data_ - pretrained LDA classifiers to predict poselet conditioned parameters
   - _./log_dir/exp-lsp-full-model/spatial_ - precomputed poselet conditioned spatial model

### Test
Assuming that PARTAPP_DIR is the directory where you have unpacked and compiled the source code, you can run the system on a single image by issuing the commands:

    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --head_detect_dpm --part_detect_dpm --find_obj --first <IMGIDX> --numimgs 1
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --save_resp_test --first <IMGIDX> --numimgs 1
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --find_obj --first <IMGIDX> --numimgs 1

where IMGIDX is index of the image (if `--first` and `--numimgs` parameters are omitted the whole dataset will be processed).

In order to evaluate the number of correctly detected parts and visualize the results run:

     <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --eval_segments --vis_segments --first <IMGIDX> --numimgs 1

WARNING: this model evaluates AdaBoost detectors at every 4th pixel to speed up the computation. This results into overall performance of _68.7%_ PCP vs _69.2%_ PCP reported in the paper. To run slower dense classifiers, set the value of `window_desc_step_ratio` to _0.125_ in abcparams_rounds500_dense05_2000.txt

### Train + Test

 1. **local appearance model**

 1) train AdaBoost model for part PARTIDX 0 - 21 (22 parts total)
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --train_class --pidx <PARTIDX>
```
 2) train generic spatial model
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --pc_learn
```
 3) compute torso position prior
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --compute_pos_prior
```
 4) train DPM part detectors (_in matlab_)
```
     cd <PARTAPP_DIR>/src/libs/libDPM
     trainDPM(<PARTIDX>, <EXP_DIR>)
```
    where <EXP_DIR> is the root of the experiments package

 5) train DPM head detector (in matlab)
```
     cd <PARTAPP_DIR>/src/libs/libDPM
     trainDPM(11, <EXP_DIR>, true)
```
 6) run local appearance model
```
     <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --head_detect_dpm --part_detect_dpm --find_obj
```

 2. **poselets**

  1) run torso detector on training images
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-train-torso.txt --find_obj --first <IMGIDX> --numimgs 1
```
  2) train poselet AdaBoost detectors
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --train_class --pidx <PARTIDX> --tidx <TYPEIDX>
```
    where PARTIDX is poselet id, 0 - 20; TYPEIDX is poselet type, 0 - ~100

  3) collect poselet responses on training and testing images
```
     <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --save_resp_train --first <IMGIDX> --numimgs 1
     <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --save_resp_test --first <IMGIDX> --numimgs 1
```
  4) train LDA classifiers
```
     <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt  --train_lda_urot --train_lda_upos --train_lda_pwise
```
 3. **full model**

  1) compute poselet conditioned mixtures
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --pc_learn_types
```
  2) run full model
```
    PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --find_obj --first <IMGIDX> --numimgs 1
```
  3) evaluate and visualize the results
```
    <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --find_obj --first <IMGIDX> --numimgs 1
```

### Managing annotations

Loading/saving annotation files in matlab:

    cd <PARTAPP_DIR>/src/scripts/matlab
    annotations = loadannotations(<annotations.al>);
    saveannotations(annotations, <annotations.al>);