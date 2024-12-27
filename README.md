### Docker

An easy way to hit the ground running and a good jumping off point depending on your use case.

Requirements:
- AWS account
- Instance type g4dn
- Ubuntu based OS with NVidia drivers pre-configured (available in AWS catalogue)
- 150 GB of storage

```sh
git clone https://github.com/neonbjb/tortoise-tts.git
cd tortoise-tts

docker build . -t tts

docker run --gpus all \
    -e TORTOISE_MODELS_DIR=/models \
    -v /mnt/user/data/tortoise_tts/models:/models \
    -v /mnt/user/data/tortoise_tts/results:/results \
    -v /mnt/user/data/.cache/huggingface:/root/.cache/huggingface \
    -v /root:/work \
    -it tts
```
This gives you an interactive terminal in an environment that's ready to do some tts. Now you can explore the different interfaces that tortoise exposes for tts.

For example:

```sh
cd app
conda activate tortoise
time python tortoise/do_tts.py \
    --output_path /results \
    --preset ultra_fast \
    --voice geralt \
    --text "Time flies like an arrow; fruit flies like a bananna."
```

## Apple Silicon

On macOS 13+ with M1/M2 chips you need to install the nighly version of PyTorch, as stated in the official page you can do:

```shell
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cpu
```

Be sure to do that after you activate the environment. If you don't use conda the commands would look like this:

```shell
python3.10 -m venv .venv
source .venv/bin/activate
pip install numba inflect psutil
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cpu
pip install transformers
git clone https://github.com/neonbjb/tortoise-tts.git
cd tortoise-tts
pip install .
```

Be aware that DeepSpeed is disabled on Apple Silicon since it does not work. The flag `--use_deepspeed` is ignored.
You may need to prepend `PYTORCH_ENABLE_MPS_FALLBACK=1` to the commands below to make them work since MPS does not support all the operations in Pytorch.


### do_tts.py

This script allows you to speak a single phrase with one or more voices.
```shell
python tortoise/do_tts.py --text "I'm going to speak this" --voice random --preset fast
```
### do socket streaming
```socket server
python tortoise/socket_server.py 
```
will listen at port 5000


### faster inference read.py

This script provides tools for reading large amounts of text.

```shell
python tortoise/read_fast.py --textfile <your text to be read> --voice random
```

### read.py

This script provides tools for reading large amounts of text.

```shell
python tortoise/read.py --textfile <your text to be read> --voice random
```

This will break up the textfile into sentences, and then convert them to speech one at a time. It will output a series
of spoken clips as they are generated. Once all the clips are generated, it will combine them into a single file and
output that as well.

Sometimes Tortoise screws up an output. You can re-generate any bad clips by re-running `read.py` with the --regenerate
argument.

### API

Tortoise can be used programmatically, like so:

```python
reference_clips = [utils.audio.load_audio(p, 22050) for p in clips_paths]
tts = api.TextToSpeech()
pcm_audio = tts.tts_with_preset("your text here", voice_samples=reference_clips, preset='fast')
```

To use deepspeed:

```python
reference_clips = [utils.audio.load_audio(p, 22050) for p in clips_paths]
tts = api.TextToSpeech(use_deepspeed=True)
pcm_audio = tts.tts_with_preset("your text here", voice_samples=reference_clips, preset='fast')
```

To use kv cache:

```python
reference_clips = [utils.audio.load_audio(p, 22050) for p in clips_paths]
tts = api.TextToSpeech(kv_cache=True)
pcm_audio = tts.tts_with_preset("your text here", voice_samples=reference_clips, preset='fast')
```

To run model in float16:

```python
reference_clips = [utils.audio.load_audio(p, 22050) for p in clips_paths]
tts = api.TextToSpeech(half=True)
pcm_audio = tts.tts_with_preset("your text here", voice_samples=reference_clips, preset='fast')
```
for Faster runs use all three:

```python
reference_clips = [utils.audio.load_audio(p, 22050) for p in clips_paths]
tts = api.TextToSpeech(use_deepspeed=True, kv_cache=True, half=True)
pcm_audio = tts.tts_with_preset("your text here", voice_samples=reference_clips, preset='fast')
```

## Acknowledgements

This project has garnered more praise than I expected. I am standing on the shoulders of giants, though, and I want to
credit a few of the amazing folks in the community that have helped make this happen:

- Hugging Face, who wrote the GPT model and the generate API used by Tortoise, and who hosts the model weights.
- [Ramesh et al](https://arxiv.org/pdf/2102.12092.pdf) who authored the DALLE paper, which is the inspiration behind Tortoise.
- [Nichol and Dhariwal](https://arxiv.org/pdf/2102.09672.pdf) who authored the (revision of) the code that drives the diffusion model.
- [Jang et al](https://arxiv.org/pdf/2106.07889.pdf) who developed and open-sourced univnet, the vocoder this repo uses.
- [Kim and Jung](https://github.com/mindslab-ai/univnet) who implemented univnet pytorch model.
- [lucidrains](https://github.com/lucidrains) who writes awesome open source pytorch models, many of which are used here.
- [Patrick von Platen](https://huggingface.co/patrickvonplaten) whose guides on setting up wav2vec were invaluable to building my dataset.

## Notice

Tortoise was built entirely by the author (James Betker) using their own hardware. Their employer was not involved in any facet of Tortoise's development.

## License

Tortoise TTS is licensed under the Apache 2.0 license.

If you use this repo or the ideas therein for your research, please cite it! A bibtex entree can be found in the right pane on GitHub.
