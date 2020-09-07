# WaveGrad
Implementation (PyTorch) of Google Brain's WaveGrad vocoder (paper: https://arxiv.org/pdf/2009.00713.pdf).

**Status** (contributing is welcome, share your ideas!):
* **Stable versions: 25, 50 and 1000 iterations** with linear betas.
* Conducting experiments on 6-iterations and 1000-iterations training.
* Also preparing real-time factor (RTF) benchmarking.
* Preparing pretrained checkpoints.
* Preparing generated audio. Unfortunately, there is some clicking artefact on the edges of parallel sampling (checkout the last section of README for details). This was my quick solution for the issue of when the model sees the mel sequence longer than training segment. If you have ideas how to resolve it, please share it by making a new issue.d

|         Model        | Stable |     RTF     |
|:--------------------:|:------:|-------------|
| 1000 iterations Base |  True  | In progress |
| 50 iterations Base   |  True  | In progress |
| 25 iterations Base   |  True  | In progress |
| 6 iterations Base    |  False | In progress |

## About

WaveGrad is a conditional model for waveform generation through estimating gradients of the data density. The main concept of vocoder is its relateness to diffusion models based on Langevin dynamics and score matching frameworks. W.r.t. Langevin dynamics WaveGrad achieves super-fast convergence (6 iterations reported in the paper).

## Setup

1. Clone this repo:

```bash
git clone https://github.com/ivanvovk/WaveGrad.git

cd WaveGrad
```

2. Install requirements `pip install -r requirements.txt`

## Train your own model

1. Make filelists of your data like ones included into `filelists` folder.
2. Setup configuration in `configs` folder.
3. Change config path in `train.sh` and run the script `sh train.sh`.
4. To track training process run tensorboard `tensorboard --logdir=logs/YOUR_LOG_FOLDER`.

## Inference and generated audios

Follow instructions provided in Jupyter notebook [`notebooks/inference.ipynb`](notebooks/inference.ipynb). Generated audios will be provided shortly.

## Details, issues and comments

* For Langevin dynamics computation [`Denoising Diffusion Probabilistic Models`](https://github.com/hojonathanho/diffusion) repository has been adopted.
* **Base 25-, 50- and 1000-iteration LJSpeech WaveGrad** models succesfully train on single 12GB GPU. Batch size is modified compared to the paper (256->48: authors trained their model on TPU).
* At some point training might start to behave very weird and crazy (loss explodes), so I introduced learning rate scheduling and gradient clipping.
* Choose betas very carefully. Prefer using standard configs, provided in repos.
* Overall, be careful and tune hyperparameters accurately for your own dataset.
* For the best reconstruction accuracy prefer to use `wavegrad.sample_subregions_parallel(...)` method instead of `wavegrad.sample(...)`. During training the model has seen only the small segments of speech, thus reconstruction accuracy degrades on the longer mel sequences with time (I have determined the reason: because of positional encoding - the model hasn't seen longer encoding during training but just the corresponding one for a small segment of speech). Parallel generation method splits given mel into segments of size from training and processes it concurrently. However, this results in some clicking artefacts on the edges of final concatenation.

## References

* [WaveGrad: Estimating Gradients for Waveform Generation](https://arxiv.org/pdf/2009.00713.pdf)
* [Denoising Diffusion Probabilistic Models](https://arxiv.org/pdf/2006.11239.pdf)