# Anime Whisper

[**Anime Whisper**](https://huggingface.co/litagin/anime-whisper) は、特に日本語のアニメ調演技セリフドメインに特化した日本語音声認識モデルです。

このモデルは [kotoba-whisper-v2.0](https://huggingface.co/kotoba-tech/kotoba-whisper-v2.0) をベースモデルとして、約5,300時間373万ファイルのアニメ調の音声・台本データセット [Galgame_Speech_ASR_16kHz](https://huggingface.co/datasets/litagin/Galgame_Speech_ASR_16kHz) でファインチューニングしたものです。
特にアニメ演技音声ドメインに特化していますが、それ以外の音声でも、他のモデルにはない特徴や高い性能を持っています。

気軽にお試しできるデモはこちらから: https://huggingface.co/spaces/litagin/anime-whisper-demo

このGitHubリポジトリは、モデル本体とは直接関係がない情報（学習コードや評価詳細や考察等）を随時更新しながら公開するためのものです（🤗モデルリポジトリを更新するとそのたびにモデルの再ダウンロードが起きる可能性があるため隔離）。

以下だいたい[モデルカード](https://huggingface.co/litagin/anime-whisper)からコピペ：

## 特徴 🌟

Anime Whisperは、他モデルに比べて一般的に次のような傾向があります。

- ハルシネーションが少ない
- 他のモデルでスキップされがちな言い淀みや、笑い声や叫びや吐息などの非言語発話も忠実に書き起こす
- 「。、!?…」の句読点が音声のリズムや感情に合わせて適切に付き、セリフ台本として違和感がない自然な文体で書き起こされる
- アニメ調な演技セリフに対しては特に精度が高い
- 他モデルでは書き起こしが不可能なNSFW音声（喘ぎ声やチュパ音等）もきちんとした文体で書き起こされる

## 使い方例 🚀

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

複数ファイルを一気に推論する場合は `pipe` に単にファイルパスのリストを渡せばよいです。

## 評価 📊

[evaluation.md](evaluation.md) を参照してください。


## バイアス等 🚨

- 人名等の固有名詞が学習データのビジュアルノベルに存在する場合、その登場人物名の漢字で書き起こされることが多い
- 一部卑語の書き起こしに伏せ字「○」が含まれることがある
- [データセットの正規化](https://huggingface.co/datasets/litagin/Galgame_Speech_ASR_16kHz#modifications) により、以下のものは出力結果にほぼ現れない:
    - 母音や長音符の連続: `ああああーーーー`
    - 同じ感嘆符の連続: `こらーっ!!!!` `なにそれ!?!?!?!?`
    - 三点リーダーの連続: `……` （日本語表記としては2個用いる `……` が正しいが、ほぼ常に1個のみ `…` で出力される）
- 数字とアルファベットと感嘆符は半角で書き起こされる
- 一部特定の単語が通常とは異なる書き起こしになることがある（例: `からだ` → `身体` 等や、その他固有名詞等）
- 文末の「。」はほぼ常に省略される

## 例 👀

[examples.md](examples.md) を参照してください。

## 学習手順 📚

**詳しい学習手順やハイパーパラメータや学習コードはそのうちここで公開予定です。**

- 全データのうち1番最後のtarファイルをtestデータとして残し、それ以外の3,735,363ファイルで学習
- まずはベースモデルからEncoderを凍結してDecoderのみで数エポックを学習
- その後Encoderの凍結を解除し、全体で数エポックを学習
- 学習打ち切り後に、「ある時点から別の時点までのモデルの平均（マージ）」を取る操作で性能向上を試み、Optunaを使ってベンチマークデータに対するCERで最適化し、その結果を最終モデルとした

### 環境 🖥

- 自腹で[vast.ai](https://vast.ai/)で H100 NVL (VRAM 96GB) を借りて合計3週間弱、試行錯誤しながら学習をした（当初はベースモデルをwhisper-large-v3-turboにしていたので、その分も含まれる）
- 実際にこのモデルに使われた学習時間は、H100 NVL * 11.2日 程度（ただし後半の方はおそらく過学習によりテストデータに対する性能が悪かったため、最終マージには用いなかった）
