# About CPU version

Making changes to interface .py and glow.py

# Acknowledgments
https://github.com/KeigoTakamura/SqueezeWave-for-CPU-forkd-/issues/1
This repository is now complete with these patches, for which I would like to express my deepest gratitude

#  todo
・ Speech is not generated well
I have n’t fixed it yet

## SqueezeWave: Extremely Lightweight Vocoders for On-device Speech Synthesis
By Bohan Zhai *, Tianren Gao *, Flora Xue, Daniel Rothchild, Bichen Wu, Joseph Gonzalez, and Kurt Keutzer (UC Berkeley)

Automatic speech synthesis is a challenging task that is becoming increasingly important as edge devices begin to interact with users through speech. Typical text-to-speech pipelines include a vocoder, which translates intermediate audio representations into an audio waveform. Most existing vocoders are difficult to parallelize since each generated sample is conditioned on previous samples. WaveGlow is a flow-based feed-forward alternative to these auto-regressive models (Prenger et al., 2019). However, while WaveGlow can be easily parallelized, the model is too expensive for real-time speech synthesis on the edge. This paper presents SqueezeWave, a family of lightweight vocoders based on WaveGlow that can generate audio of similar quality to WaveGlow with 61x - 214x fewer MACs.

Link to the paper: [paper]. If you find this work useful, please consider citing

   ```
   @inproceedings{squeezewave,
      Author = {Bohan Zhai, Tianren Gao, Flora Xue, Daniel Rothchild, Bichen Wu, Joseph Gonzalez, Kurt Keutzer},
      Title = {SqueezeWave: Extremely Lightweight Vocoders for On-device Speech Synthesis},
      Journal = {arXiv:2001.05685},
      Year = {2020}
   }
   ```

### Audio samples generated by SqueezeWave
Audio samples of SqueezeWave are here: https://tianrengao.github.io/SqueezeWaveDemo/

### Results
We introduce 4 variants of SqueezeWave in our paper. See the table below.


   | Model           | length | n_channels| MACs  | Reduction | MOS       |
   | --------------- | ------ | --------- | ----- | --------- | --------- |
   |WaveGlow         |  2048  | 8         | 228.9 | 1x        | 4.57±0.04 |
   |SqueezeWave-128L |  128   | 256       | 3.78  | 60x       | 4.07±0.06 |
   |SqueezeWave-64L  |  64    | 256       | 2.16  | 106x      | 3.77±0.05 |
   |SqueezeWave-128S |  128   | 128       | 1.06  | 214x      | 3.79±0.05 |
   |SqueezeWave-64S  |  64    | 128       | 0.68  | 332x      | 2.74±0.04 |

### Model Complexity
A detailed MAC calculation can be found from [here](https://github.com/tianrengao/SqueezeWave/blob/master/SqueezeWave_computational_complexity.ipynb)

## Setup
0. (Optional) Create a virtual environment

   ```
   virtualenv env
   source env/bin/activate
   ```

1. Clone our repo and initialize submodule

   ```command
   git clone https://github.com/tianrengao/SqueezeWave.git
   cd SqueezeWave
   git submodule init
   git submodule update
   ```

2. Install requirements 
```pip3 install -r requirements.txt``` 

3. Install [Apex]
   ```1
   cd ../
   git clone https://www.github.com/nvidia/apex
   cd apex
   python setup.py install
   ```

## Generate audio with our pretrained model

1. Download our [pretrained models]. We provide 4 pretrained models as described in the paper.
2. Download [mel-spectrograms]
3. Generate audio. Please replace `SqueezeWave.pt` to the specific pretrained model's name.

   ```python3 inference.py -f <(ls mel_spectrograms/*.pt) -w SqueezeWave.pt -o . --is_fp16 -s 0.6```


## Train your own model

1. Download [LJ Speech Data]. We assume all the waves are stored in the directory `^/data/`

2. Make a list of the file names to use for training/testing

   ```command
   ls data/*.wav | tail -n+10 > train_files.txt
   ls data/*.wav | head -n10 > test_files.txt
   ```

3. We provide 4 model configurations with audio channel and channel numbers specified in the table below. The configuration files are under ```/configs``` directory. To choose the model you want to train, select the corresponding configuration file.

4. Train your SqueezeWave model

   ```command
   mkdir checkpoints
   python train.py -c configs/config_a256_c128.json
   ```

   For multi-GPU training replace `train.py` with `distributed.py`.  Only tested with single node and NCCL.

   For mixed precision training set `"fp16_run": true` on `config.json`.

5. Make test set mel-spectrograms

   ```
   mkdir -p eval/mels
   python3 mel2samp.py -f test_files.txt -o eval/mels -c configs/config_a128_c256.json
   ```

6. Run inference on the test data. 

   ```command
   ls eval/mels > eval/mel_files.txt
   sed -i -e 's_.*_eval/mels/&_' eval/mel_files.txt
   mkdir -p eval/output
   python3 inference.py -f eval/mel_files.txt -w checkpoints/SqueezeWave_10000 -o eval/output --is_fp16 -s 0.6
   ```
   Replace `SqueezeWave_10000` with the checkpoint you want to test.
   
## Credits
The implementation of this work is based on WaveGlow: https://github.com/NVIDIA/waveglow


[//]: # (TODO)
[//]: # (PROVIDE INSTRUCTIONS FOR DOWNLOADING LJS)
[pytorch 1.0]: https://github.com/pytorch/pytorch#installation
[website]: https://nv-adlr.github.io/WaveGlow
[paper]: https://arxiv.org/abs/2001.05685
[WaveNet implementation]: https://github.com/r9y9/wavenet_vocoder
[Glow]: https://blog.openai.com/glow/
[WaveNet]: https://deepmind.com/blog/wavenet-generative-model-raw-audio/
[PyTorch]: http://pytorch.org
[pretrained models]: https://drive.google.com/file/d/1RyVMLY2l8JJGq_dCEAAd8rIRIn_k13UB/view?usp=sharing
[mel-spectrograms]: https://drive.google.com/file/d/1g_VXK2lpP9J25dQFhQwx7doWl_p20fXA/view?usp=sharing
[LJ Speech Data]: https://keithito.com/LJ-Speech-Dataset
[Apex]: https://github.com/nvidia/apex
