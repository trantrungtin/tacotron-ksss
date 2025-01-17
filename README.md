# Multi-Speaker Tacotron in TensorFlow

This is a forked version of [multi-speaker-tacotron-tensorflow](https://github.com/carpedm20/multi-speaker-tacotron-tensorflow). The codes are updated to support the latest Tensorflow (1.12) and [Korean Single Speaker Speech](https://www.kaggle.com/bryanpark/korean-single-speaker-speech-dataset/home) dataset.

TensorFlow implementation of:

- [Deep Voice 2: Multi-Speaker Neural Text-to-Speech](https://arxiv.org/abs/1705.08947)
- [Listening while Speaking: Speech Chain by Deep Learning](https://arxiv.org/abs/1707.04879)
- [Tacotron: Towards End-to-End Speech Synthesis](https://arxiv.org/abs/1703.10135)

Samples audios (in Korean) can be found [here](http://carpedm20.github.io/tacotron/en.html).

![model](./assets/model.png)


## Prerequisites

- Python 3.6+
- FFmpeg
- [Tensorflow 1.12](https://www.tensorflow.org/install/)


## Usage

### 1. Install prerequisites

After preparing [Tensorflow](https://www.tensorflow.org/install/), install prerequisites with:

    pip install -r requirements.txt

Other libraries are almost up-to-date, but you should install `pip install librosa==0.5.1` for compatibility.

If you want to synthesize a speech in Korean dicrectly, follow [2-2. Generate Korean datasets](#2-2-generate-korean-datasets).


### 2-1. Generate custom datasets

The `datasets` directory should look like:

    datasets
    ├── son
    │   ├── alignment.json
    │   └── audio
    │       ├── 1.mp3
    │       ├── 2.mp3
    │       ├── 3.mp3
    │       └── ...
    └── YOUR_DATASET
        ├── alignment.json
        └── audio
            ├── 1.mp3
            ├── 2.mp3
            ├── 3.mp3
            └── ...

and `YOUR_DATASET/alignment.json` should look like:

    {
        "./datasets/YOUR_DATASET/audio/001.mp3": "My name is Taehoon Kim.",
        "./datasets/YOUR_DATASET/audio/002.mp3": "The buses aren't the problem.",
        "./datasets/YOUR_DATASET/audio/003.mp3": "They have discovered a new particle.",
    }

After you prepare as described, you should genearte preprocessed data with:

    python -m datasets.generate_data ./datasets/YOUR_DATASET/alignment.json


### 2-2. Generate Korean datasets

Follow below commands. (explain with `Korean Single Speaker Speech` dataset)

1. [Download KSSS dataset](https://www.kaggle.com/bryanpark/korean-single-speaker-speech-dataset/home). Extract the contents under `datasets/ksss/KoreanSingleSpeakerSpeech` directory. The directory should contain `1`, `2`, `3`, `4` directory and `transcript.txt`.

2. Generate the alignment file.

    python datasets/ksss/generator.py

3. Finally, generate numpy files which will be used in training.

    python -m datasets.generate_data ./datasets/ksss/alignment.json

### 2-3. Generate English datasets

1. Download speech dataset at https://keithito.com/LJ-Speech-Dataset/

2. Convert metadata CSV file to json file. (arguments are available for changing preferences)

    python -m datasets.LJSpeech_1_0.prepare

3. Finally, generate numpy files which will be used in training.

    python -m datasets.generate_data ./datasets/LJSpeech_1_0


### 3. Train a model

The important hyperparameters for a models are defined in `hparams.py`.

(**Change `cleaners` in `hparams.py` from `korean_cleaners` to `english_cleaners` to train with English dataset**)

To train a single-speaker model:

    python train.py --data_path=datasets/ksss
    python train.py --data_path=datasets/ksss --initialize_path=PATH_TO_CHECKPOINT

To train a multi-speaker model:

    # after change `model_type` in `hparams.py` to `deepvoice` or `simple`
    python train.py --data_path=datasets/son1,datasets/son2

To restart a training from previous experiments such as `logs/ksss-20171015`:

    python train.py --data_path=datasets/ksss --load_path logs/ksss-20171015

If you don't have good and enough (10+ hours) dataset, it would be better to use `--initialize_path` to use a well-trained model as initial parameters.


### 4. Synthesize audio

You can train your own models with:

    python app.py --load_path ksss-pretrained --num_speakers=1

or generate audio directly with:

    python synthesizer.py --load_path ksss-pretrained --text "이거 실화냐?"

### 4-1. Synthesizing non-korean(english) audio

For generating non-korean audio, you must set the argument --is_korean False.

    python app.py --load_path logs/LJSpeech_1_0-20180108 --num_speakers=1 --is_korean=False
    python synthesizer.py --load_path logs/LJSpeech_1_0-20180108 --text="Winter is coming." --is_korean=False

## Results

Training attention on single speaker model:

![model](./assets/attention_single_speaker.gif)

Training attention on multi speaker model:

![model](./assets/attention_multi_speaker.gif)


## References

- [Keith Ito](https://github.com/keithito)'s [tacotron](https://github.com/keithito/tacotron)
- [DEVIEW 2017 presentation](https://www.slideshare.net/carpedm20/deview-2017-80824162)
