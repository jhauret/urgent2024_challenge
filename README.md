# urgent2024_challenge
Official data preparation scripts for the URGENT 2024 Challenge

## Requirements

- `>8` Cores
- At least `1` GPU (4 or 8 GPUs are recommended for speedup in DNSMOS or other DNN-based metric calculation)
- At least 1.2 TB of free disk space
  - Speech
    - DNS5 speech (original 131 GB + resampled 187 GB): 318 GB
    - CommonVoice English speech (original mp3 82 GB + resampled 509 GB): 591 GB
    - LibriTTS (original 44 GB + resampled 7 GB): 51 GB
    - VCTK: 12 GB
    - WSJ (original sph 24GB + converted 31 GB): 55 GB
  - Noise
    - DNS5 noise (original 58 GB + resampled 35 GB): 93 GB
    - WHAM! noise (48 kHz): 76 GB
    - [optional] EPIC-Sounds noise (original video 1.3 TB + original audio 33 GB + resampled 370 GB): 1.7 TB
  - RIR
    - DNS5 RIRs (48 kHz): 6 GB
  - Others
    - default simulated validation data: ~16 GB

With minimum specs, expects the whole process to take YYY hours.

## Instructions

0. After cloning this repository, run the following command to initialize the submodules:
    ```bash
    git submodule update --init --recursive
    ```

1. Install environmemnt. Python 3.10 and Torch 2.0.1 are recommended.
   With Anaconda, just run

    ```bash
    conda env create -f environment.yaml
    conda activate urgent
    ```

2. Download Commonvoice dataset v11 from https://commonvoice.mozilla.org/en/datasets

    a. Select `Common Voice Corpus 11.0`

    b. Enter your email and check the two mandatory boxes

    c. Right-click the `Download Dataset Bundle` button and select "Copy link"

    d. Enter the following commands in your terminal

    ```bash
    URL="<paste-link>"
    wget $URL -O ./datasets_cv11_en/cv-corpus-11.0-2022-09-21-en.tar.gz
    python ./utils/tar_extractor.py -m 5000 \
        -i ./datasets_cv11_en/cv-corpus-11.0-2022-09-21-en.tar.gz \
        -o ./datasets_cv11_en \
        --skip_existing --skip_errors
    ``` 

3. Download WSJ0 and WSJ1 datasets from LDC
    > You will need a LDC license to access the data.
    >
    > For URGENT Challenge participants who want to use the data during the challenge period, please contact the organizers for a temporary LDC license.

    a. Download WSJ0 from https://catalog.ldc.upenn.edu/LDC93s6a

    b. Download WSJ1 from https://catalog.ldc.upenn.edu/LDC94S13A

    c. Uncompress and store the downloaded data to the directories `./wsj/wsj0/` and `./wsj/wsj1/`, respectively.

4. Run the script

    ```bash
    ./prepare_espnet_data.sh
    ```

5. Install eSpeak-NG (used for the phoneme similarity metric computation)
   - Follow the instructions in https://github.com/espeak-ng/espeak-ng/blob/master/docs/guide.md#linux

## Optional: Prepare webdataset

The script `./utils/prepare_wds.py` can store the audio files in a collection
of tar files each containing a predefined number of audio files. This is useful
to reduce the number of IO operations during training. Please see the
[documentation](https://github.com/webdataset/webdataset) of `webdataset` for
more information.

```bash
OMP_NUM_THREADS=1 python ./utils/prepare_wds.py \
    /path/to/urgent_train_24k_wds \
    --files-per-tar 250 \
    --max-workers 8 \
    --scps data/tmp/commonvoice_11.0_en_resampled_filtered_train.scp \
    data/tmp/dns5_clean_read_speech_resampled_filtered_train.scp \
    data/tmp/vctk_train.scp \
    data/tmp/libritts_resampled_train.scp
```
The script can also resample the whole dataset to a unified sampling frequency
with `--sampling-rate <freq_hz>`. This option will not include samples with
sampling frequency lower than the prescribed frequency.
