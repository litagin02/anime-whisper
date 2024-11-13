# Anime Whisper

[**Anime Whisper**](https://huggingface.co/litagin/anime-whisper) is a Japanese speech recognition model specialised for anime-style dialogue.

This model is fine-tuned from [kotoba-whisper-v2.0](https://huggingface.co/kotoba-tech/kotoba-whisper-v2.0) as the base model, using approximately 5300 hours of anime-style voice and transcription from the [Galgame_Speech_ASR_16kHz](https://huggingface.co/datasets/litagin/Galgame_Speech_ASR_16kHz) dataset which contains 3.73 million files.
While it specializes in anime voice acting, it also performs well on other types of audio with unique features and high performance not found in other models.

An easy to use demo is provided here: https://huggingface.co/spaces/litagin/anime-whisper-demo

This GitHub repository is for publishing and regularly updating information not directly related to the model itself (such as training code, detailed evaluations, and analysis). This is separate from the 🤗 model repository to avoid triggering potential model re-downloads with each update.

The following is mostly copy-pasted from the [model card](https://huggingface.co/litagin/anime-whisper):

## Features 🌟

Compared to other models, Anime Whisper generally shows the following characteristics:

- Fewer hallucinations
- Faithfully transcribes stutters, laughter, screams, sighs, and other non-verbal utterances that other models tend to skip
- Punctuation marks like "。、!?…" are appropriately placed according to the rhythm and emotion of the voice, resulting in natural script-like transcriptions
- Particularly high accuracy for anime-style voice acting dialogue
- Properly transcribes NSFW audio (moans, kissing sounds, etc.) in appropriate writing style, which other models cannot transcribe

## Usage Example 🚀

```python
import torch
from transformers import pipeline

pipe = pipeline(
    "automatic-speech-recognition",
    model="litagin/anime-whisper",
    device="cuda",
    torch_dtype=torch.float16,
    chunk_length_s=30.0,
    batch_size=64,
)

audio_path = "test.wav"
result = pipe(audio_path)
print(result["text"])
```

For batch inference of multiple files, simply pass a list of file paths to `pipe`.

## Evaluation 📊

Please refer to [evaluation.md](evaluation.md).

## Biases and Limitations 🚨

- When proper nouns like character names exist in the training data visual novels, they tend to be transcribed using the kanji from those games
- Some vulgar words may contain censorship marks "○"
- Due to [dataset normalization](https://huggingface.co/datasets/litagin/Galgame_Speech_ASR_16kHz#modifications), the following rarely appear in output:
    - Consecutive vowel sounds or "ー": `ああああーーーー`
    - Consecutive exclamation marks: `こらーっ!!!!` `なにそれ!?!?!?!?`
    - Consecutive ellipses: `……` (While two dots `……` is correct in Japanese notation, it almost always outputs just one `…`)
- Numbers, alphabets, and exclamation marks are transcribed in half-width
- Certain words may be transcribed differently from standard usage (e.g., `からだ` → `身体`, and other proper nouns)
- The sentence-ending period "。" is almost always omitted

## Examples 👀

Please refer to [examples.md](examples.md).

## Training Procedure 📚

**Detailed training procedures, hyperparameters, and training code will be published here soon.**

- The last tar file from the training data was reserved as test/evaluation data, with the remaining 3,735,363 files used for training
- First trained for several epochs with the Encoder frozen from the base model, using only the Decoder
- Then unfroze the Encoder and trained the entire model for several epochs
- After training completion, attempted performance improvement by averaging (merging) models from different checkpoints, optimizing CER against benchmark data using Optuna, and used the result as the final model

### Environment 🖥

- Trained on a rented H100 NVL (96GB VRAM) from [vast.ai](https://vast.ai/) for nearly 3 weeks total, including experimentation (this includes time spent with whisper-large-v3-turbo as the initial base model)
- The actual training time used for this model was approximately 11.2 days on H100 NVL (though later portions weren't used in the final merge due to likely overfitting and degraded test data performance)