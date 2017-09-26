With this repo you can preprocess an audio dataset (modify phoneme classes, resample audio etc), and train LSTM networks for framewise phoneme classification.
You can achieve 82% accuracy on the TIMIT dataset, similar to the results from [Graves et al (2013)](https://arxiv.org/abs/1303.5778), although CTC is not used here.
Instead, the network generates predictions in the middle of each phoneme interval, as specified by the labels. This is to simplify things, but using CTC shouldn't be too much trouble.

In order to create and train a network on a dataset, you need to:  

0. install software. I recommend using [Anaconda](https//www.anaconda.com/download/) and a virtual environment.
    - create an environment from the provided file: `conda env create -f environment.yml`

1. Generate a binary file from the source dataset. It's easiest if you structure your data as in the TIMIT dataset, although that's not really needed. Just make sure that the wav and its corresponding phn file have the same path except for the extension. Otherwise they won't get matched and your labels will be off.    
    - WAV files, 16kHz sampling rate, folder structure `dataset/TRAIN/speakerName/videoName/`. 
        Each videoName/ directory contains a `videoName.wav` and `videoName.phn`. 
        The phn contains the audio sample (@16kHz) numbers where each phoneme starts and ends.   
    - If your files are in a different format, you can use functions from fixDataset/ to: (use transform.py, with the appropriate arguments, see bottom of file)   
        - fix wav headers, resample wavs. Store them under `dataRoot/dataset/fixed(nbPhonemes)/`  
        `transform.py phonemes -i dataRoot/TIMIT/original/ -o dataRoot/TIMIT/fixed`
        - fix labelfiles: replace phonemes (eg to use a reduced phoneme set; I used the 39 phonemes from Lee and Hon (1989)).  Stored next to fixed wavs, under `root/dataset/fixed(nbPhonemes)/`  
        - create a MLF file (like from HTK tool, and as used in the TCDTIMIT dataset)   
        - the scripts should be case-agnostic, but you can convert lower to uppercase and vice versa by running `find . -depth -print0 | xargs -0 rename '$_ = lc $_'` in the root dataset directory (change 'lc' to 'uc to convert to upper case). Repeat until you get no more output.  
    - Then set variables in datasetToPkl.py (source and target dir, nbMFCCs to use etc), and run the file   
        - the result is stored as `root/dataset/binary(nbPhonemes)/dataset/dataset_nbPhonemes_ch.pkl`. eg root/TIMIT/binary39/TIMIT/TIMIT_39_ch.pkl  
        - the mean and std_dev of the train data are stored as `root/dataset/binary_nbPhonemes/dataset_MeanStd.pkl`. It's useful for normalization when evaluating. 
1. Use RNN.py to start training. Its functions are implemented in RNN_tools_lstm.py, but you can set the parameters from RNN.py.    
    - set location of pkl generated by datasetToPkl.py  
    - specify number of LSTM layers and number of units per layer  
    - use bidirectional LSTM layers   
    - add some dense layers (though it did not improve performance for me)  
    - learning rate and decay (LR is updated at end of RNN_tools_lstm.py). It's decreased if the performance hasn't improved for some time.    
    
    - it will automatically give the model a name based on the specified parameters. A log file, the model parameters and a pkl file containing training info (accuracy, error etc for each epoch) are stored as well. 
      The storage location is`root/dataset/results`  
        
1. to evaluate a dataset, change the test_dataset variable to whatever you want (TIMIT/TCDTIMIT/combined)
    1. You can generate test datasets with noise (either white noise or simultaneous speakers) of a certain level using mergeAudioFiles.py to create the wavs and testdataToPkl.py to convert that to pkl files.
    1. If this noisy audio is to be used for combinedSR, you need to generate the pkl files a bit differently, using audioToPkl_perVideo.py. That pkl file can then be combined with the images and labels generated by combinedSR/datasetToPkl. You can enable this by setting some parameters in `combinedSR/combinedNN.py`

On TIMIT, you should get about 82% accuracy using a 2-layer, 256 units/layer bidirectional STM network
You should get about 67% on TCD-TIMIT.
  
The TIMIT dataset is non-free and available from [https://catalog.ldc.upenn.edu/LDC93S19](https://catalog.ldc.upenn.edu/LDC93S1).    
The TCD-TIMIT dataset is free for research and available from [https://sigmedia.tcd.ie/TCDTIMIT/](https://sigmedia.tcd.ie/TCDTIMIT/).  
If you want to use TCD-TIMIT, I recommend to use my repo [TCDTIMITprocessing](https://github.com/matthijsvk/TCDTIMITprocessing) to download, and extract the database. It's quite a nasty job otherwise. You can use `extractTCDTIMITaudio.py` to get the phoneme and wav files.

If you want to do lipreading or audio-visual speech recognition, check out my other repository [MultimodalSR](https://github.com/matthijsvk/multimodalSR)
