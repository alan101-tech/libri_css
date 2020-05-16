# LibriCSS
Continuous speech separation (CSS) is an approach to handling overlapped speech in conversational audio signals. Most previous speech separation algorithms were tested on artificially mixed pre-segmented speech signals and thus bypassed overlap detection and speaker counting by implicitly assuming overlapped regions to be already extracted from the input audio. CSS is an attempt to directly process the continuously incoming audio signals with online processing. The main concept was established and its effectiveness was evaluated on real meeting recordings in [1]. As these recordings were proprietary, a publicly available dataset, called LibriCSS, has been prepared by the same research group in [2]. This repository contains the programs for LibriCSS evaluation. The LibriCSS dataset can be used for evaluation offline algorithms. 

[1] T. Yoshioka et al., "Advances in Online Audio-Visual Meeting Transcription," 2019 IEEE Automatic Speech Recognition and Understanding Workshop (ASRU), SG, Singapore, 2019, pp. 276-283. 

[2] Z. Chen et al., "Continuous speech separation: dataset and analysis," ICASSP 2020 - 2020 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), Barcelona, Spain, 2020, accepted for publication.


## Requirements

We use SCTK (https://github.com/usnistgov/SCTK), the NIST Scoring Toolkit, for evaluation and PyKaldi2 (https://github.com/jzlianglu/pykaldi2), a Python internface to Kaldi for ASR. They can be installed as follows. Note that you need to have Docker enabled on your machine as it is required by PyKaldi2. 
```
./install.sh
source ./path.sh
```
The second command defines some environmental variables, where path.sh is generated as a result of the first command. 

We also use some Python packages. Assuming you are using conda, the simplest way to install all required dependencies is to create a conda environment as follows. 
```
conda env create -f conda_env.yml
conda activate libricss_release
```
The second command activates the newly created environment named libricss_release. 


## Getting Started


### Continuous input evaluation
To perform continuous input evaluation, you may follow the steps below. 
1. First, the data can be downloaded and preprocessed as follows. 
    ```
    ./dataprep/scripts/dataprep.sh
    ```
2. Then, ASR can be run by taking the following steps. 
    ```
    ./asr/scripts/gen_asrinput_raw_continuous.sh  # performing VAD
    sh activate.sh  # activating PyKaldi2 Docker environment
    source path.sh
    source asr/scripts/asr_path.sh
    cd exp/data/baseline/segments/decoding_cmd
    . decode.sh  # running ASR (If you want to specify the GPU to use, add export "CUDA_VISIBLE_DEVICES=N" at the top of decode.sh, where N is an integer corresponding to the GPU index.)
    ```
    This will generate CTM files for each mini session, under exp/data/baseline/segments/decoding_result.sorted. If you want to use your own ASR system, you may skip this step. 
    
    Also you might want to change the permission of intermediate files before you exit the PyKaldi2 Docker environment by Ctrl-d, as by default files generated within the Docker environment are owned by root. 
    ```
    chmod -R 777 $EXPROOT
    ```
    
3. Finally, the ASR results can be scored as follows. 
    ```
    ./scoring/scripts/eval_continuous.sh exp/data/baseline/segments/decoding_result.sorted/13_0.0
    python ./scoring/python/report.py --inputdir exp/data/baseline/segments/decoding_result.sorted/13_0.0
    ```  
    The Python script scoring/python/report.py will print out the results as follows. 
    ```  
    Result Summary
    --------------
    Condition: %WER
    0S       : 15.4
    0L       : 11.4
    10       : 21.7
    20       : 27.2
    30       : 34.3
    40       : 40.4    
    ```  
    This corresponds to the "no separation" results of Table 2 in [2]. If you have skipped step 2, you may use the sample CTM files provided under "sample" directory to see how the scoring script works. 


### Utterance-wise evaluation

We assume that you have already downloaded the AM and PyKaldi2 as described above. 

1. Activate the PyKaldi2 Docker environment by running:
    ```
    sh activate.sh
    source path.sh
    source asr/scripts/asr_path.sh
    ```

2. Then, the decoding command can be generated, and perform decoding as follows. 
    ```
    cd asr/script
    ./gen_asrinput_raw_utterance.sh
    cd ../exp
    . decode_raw_utterance.sh    
    ```
  
    Also you might want to change the permission of the intermediate files before you exit the Docker environment by Ctrl-d, as by default the files generated within the Docker environment are owned by root. 
    ```
    chmod -R 777 $EXPROOT
    ```
  
3. Finally, collect the WERs with following command: 
    ```
    cd ../scripts
    . run_wer_raw_utterance.sh
    ```
    The result will be shown and can be found exp/data/baseline/utterance/decoding_result.    
    ```
    0S       : 0.11458333333333333
    0L       : 0.11254386680812863
    OV10       : 0.1828377230246389
    OV20       : 0.2641803896243677
    OV30       : 0.34600058314705023
    OV40       : 0.43238971784502395
    ```

## Example CSS Results

The steps described above generate the baseline results without speech separation nor overlapped speech recognition. 
We are also publishing example output waveforms of the CSS algorithm as well as the scripts that perform ASR and WER scoring for these signals. 
Those focusing on the front-end speech separation algorithms will be able to test their own algorithms by replacing the example waveforms by their own ones. 


### Continuous input evaluation
To perform continuous input evaluation, you may follow the steps below. 
1. First, the example waveforms can be downloaded as follows. 
    ```
    ./dataprep/scripts/dataprep_separation.sh
    ```
2. Then, ASR can be run by taking the following steps. 
    ```
    ./asr/script/gen_asrinput_separated_continuous.sh  # performing VAD
    sh activate.sh  # activating PyKaldi2 Docker environment
    source path.sh
    source asr/scripts/asr_path.sh
    cd exp/data/separation_baseline/decoding_cmd
    . decode.sh  # running ASR (If you want to specify the GPU to use, add export "CUDA_VISIBLE_DEVICES=N" at the top of decode.sh, where N is an integer corresponding to the GPU index.)
    exit  # quitting the Docker environment
    ```    
    
3. Finally, the ASR results can be scored as follows. 
    ```
    ./scoring/scripts/eval_continuous.sh exp/data/separation_baseline/decoding_result.sorted/13_0.0/
    python scoring/python/report.py --inputdir exp/data/separation_baseline/decoding_result.sorted/13_0.0/
    ```  
    The Python script scoring/python/report.py will print out the results as follows. 
    ```  
    Result Summary
    --------------
    Condition: %WER
    0S       : 11.9
    0L       : 9.7
    10       : 13.4
    20       : 15.1
    30       : 19.7
    40       : 22.0    
    ```  
    This corresponds to the "7ch" results of Table 3 in [2]. 


### Utterance-wise evaluation

TBD.


## Some Details

### Data
NOTE: WE RECOMMEND THAT YOU USE dataprep/scripts/dataprep.sh MENTIONED ABOVE TO OBTAIN THE DATA.

LibriCSS consists of distant microphone recordings of concatenated LibriSpeech utterances played back from loudspeakers in an office room, which enables evaluation of speech separation algorithms that handle long form audio. See [2] for details of the data. The data can be downloaded at https://drive.google.com/file/d/1Piioxd5G_85K9Bhcr8ebdhXx0CnaHy7l/view. The archive file contains only the original "mini-session" recordings (see Section 3.1 of [2]) as well as the source signals played back from the loudspeakers. By following the instruction described in the README file, you should be able to generate the data for both utterance-wise evaluation (see Section 3.3.2 of [2]) and continuous input evaluation (Section 3.3.3 of [2]). 

The original directory structure is different from that the ASR/scoring tools expect. Python script dataprep/python/dataprep.py reorganizes the recordings. To see how it can be used, refer to dataprep/scripts/dataprep.sh (which executes all the necessary steps to start experiments). 



### Task (continuous input evaluation)
As a result of the data preparation step,  the 7-ch and 1-ch test data are created by default under $EXPROOT/7ch and $EXPROOT/monaural, respectively (EXPROOT is defined in path.sh). 
These directories consist of subdirectories named overlap_ratio\_\*\_sil\*\_\*\_session\*\_actual\*, each containing chunked mini-session 
audio files segment\_\*.wav (see Section 3.3.3 of [2]). 

The task is to trascribe each file and save the result in the CTM format as segment\_\*.ctm. Refer to http://my.fit.edu/~vkepuska/ece5527/sctk-2.3-rc1/doc/infmts.htm#ctm_fmt_name_0 for the CTM format specification. The result directory has to retain the original subdirectory structure, as in the "sample" directory. Then, your ASR CTM files can be evaluated with scoring/scripts/eval_continuous.sh. 

### Task (utterance-wise evaluation)

The data stucture is the same as the continuous evaluation, organized by mini session, where each utterance in the mini session is pre-segmented with ground truth boundary information.

The utterance-wise evaluation transcribes each utterance individually. As with many existing speech separation/enhancement tasks, when multiple separation results are generated for one input utterance mixture, the transcription result with the lowest WER is picked as the final result.


