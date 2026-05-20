# MediaPipe Gesture Model Training

MediaPipe Model Maker を使って、ジェスチャー画像データセットから
`gesture_recognizer.task` を学習・書き出しするための最小構成です。

## 環境

Apple Silicon Mac では `tensorflow-text==2.15.x` の macOS arm64 wheel がないため、
通常の `pip install mediapipe-model-maker` だけでは依存解決に失敗します。
このリポジトリではジェスチャー認識に必要な依存だけを固定して入れます。

```bash
./setup_env_macos_arm64.sh
```

作成される conda 環境名は `mp-model-maker` です。

## データセット形式

MediaPipe Model Maker の Gesture Recognizer は次の形式を期待します。

```text
dataset/
  none/
    image_001.jpg
    image_002.jpg
  gesture_a/
    image_001.jpg
  gesture_b/
    image_001.jpg
```

注意:

- `none` ラベルは必須です。
- `none` には、どのカスタムジェスチャーにも該当しない手の画像を入れます。
- 対応拡張子は `bmp`, `jpeg`, `jpg`, `png`, `webp` です。
- 手が検出できない画像は前処理時に除外されます。

## 学習

```bash
conda run -n mp-model-maker python train.py \
  --dataset path/to/dataset \
  --export-dir exported_model \
  --epochs 10 \
  --batch-size 2 \
  --overwrite
```

学習後、次のファイルが出力されます。

```text
exported_model/
  gesture_recognizer.task
  metadata.json
  checkpoint...
```

アプリ側で使う主なファイルは `gesture_recognizer.task` です。

## 公式サンプルで動作確認

公式の Rock/Paper/Scissors サンプルデータを使う場合:

```bash
curl -L -o rps_data_sample.zip \
  https://storage.googleapis.com/mediapipe-tasks/gesture_recognizer/rps_data_sample.zip
unzip -q -o rps_data_sample.zip

conda run -n mp-model-maker python train.py \
  --dataset rps_data_sample \
  --export-dir exported_model_test \
  --epochs 1 \
  --batch-size 8 \
  --overwrite
```

この環境では上記コマンドで学習、評価、`.task` 書き出しまで確認済みです。

## よく使うオプション

```bash
python train.py --help
```

主なオプション:

- `--model-name`: 出力する `.task` ファイル名。既定値は `gesture_recognizer.task`。
- `--epochs`: 学習 epoch 数。
- `--batch-size`: バッチサイズ。
- `--learning-rate`: 学習率。
- `--layer-widths`: 追加する隠れ層。例: `--layer-widths 128,64`
- `--min-detection-confidence`: 手検出の信頼度しきい値。
- `--skip-eval`: テスト分割での評価を省略。
- `--overwrite`: 既存の `export-dir` を削除して新規学習。

## 参考

- MediaPipe Gesture Recognizer customization guide:
  https://ai.google.dev/edge/mediapipe/solutions/customization/gesture_recognizer
- mediapipe-model-maker on PyPI:
  https://pypi.org/project/mediapipe-model-maker/
