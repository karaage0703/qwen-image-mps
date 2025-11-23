# Qwen Image for DGX Spark

DGX Spark上でQwen/Qwen-Imageを使用してテキストから画像を生成・編集するツールです。CUDA 12.9に対応し、GGUF量子化によるメモリ最適化をサポートしています。

## DGX Sparkでの動作確認

- **GPU**: NVIDIA GB10 (119GB VRAM)
- **CUDA**: 12.9
- **量子化**: Q4_0使用時、メモリ使用量を約120GB→30GBに削減
- **生成速度**: 高速モード（8ステップ）で約37秒

## 主な機能

- **CUDA 12.9対応**: DGX Spark環境でのネイティブサポート
- **GGUF量子化サポート**: メモリ使用量を大幅削減（Q2_K〜Q8_0）
- **画像生成**: テキストプロンプトから新しい画像を生成
- **画像編集**: 既存の画像をテキスト指示で編集
- **高速モード**: Lightning LoRAによる8ステップ生成
- **超高速モード**: Lightning LoRAによる4ステップ生成
- **Gradio UI**: ブラウザベースのグラフィカルインターフェース

## セットアップ

### 前提条件

- Python 3.10以上
- CUDA 12.9環境（DGX Spark）
- [uv](https://docs.astral.sh/uv/)（推奨）またはpip

### インストール手順

#### 1. リポジトリのクローン

```bash
git clone https://github.com/karaage0703/qwen-image-mps.git
cd qwen-image-mps
```

#### 2. uvを使用したセットアップ（推奨）

```bash
# uvのインストール（まだの場合）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 依存関係のインストール
uv sync
```

これでCUDA 12.9対応のPyTorchが自動的にインストールされます。

#### 3. モデルのダウンロード

初回実行時、約58GBのモデルがHugging Faceから自動ダウンロードされます（`~/.cache/huggingface/hub/`に保存）。

## 使い方

### CLI（コマンドライン）

#### 基本的な画像生成

```bash
# デフォルトプロンプトで生成
uv run qwen-image-mps generate

# カスタムプロンプトで生成
uv run qwen-image-mps generate -p "富士山と桜、美しい風景、超高解像度"

# 高速モード（8ステップ）
uv run qwen-image-mps generate -p "猫の写真" --fast

# 超高速モード（4ステップ）
uv run qwen-image-mps generate -p "猫の写真" --ultra-fast
```

#### 量子化を使用した生成（推奨）

**DGX Sparkでメモリ不足が発生する場合は、必ず量子化オプションを使用してください。**

```bash
# Q4_0量子化（推奨：メモリ使用量を約1/4に削減）
uv run qwen-image-mps generate -p "猫の写真" --fast --quantization Q4_0

# Q2_K量子化（最も軽量：メモリ使用量を最小化）
uv run qwen-image-mps generate -p "猫の写真" --fast --quantization Q2_K

# Q8_0量子化（高品質：メモリ削減は控えめだが品質維持）
uv run qwen-image-mps generate -p "猫の写真" --fast --quantization Q8_0
```

**利用可能な量子化レベル:**
- `Q2_K`: 最も軽量（品質は低下）
- `Q3_K_S`, `Q3_K_M`: 軽量
- `Q4_0`, `Q4_1`, `Q4_K_S`, `Q4_K_M`: バランス型（推奨）
- `Q5_0`, `Q5_1`, `Q5_K_S`, `Q5_K_M`: 高品質
- `Q6_K`, `Q8_0`: 最高品質

#### 複数画像の生成

```bash
# 4枚の画像を生成
uv run qwen-image-mps generate -p "未来都市" --num-images 4 --fast --quantization Q4_0

# シード値を指定して再現可能な生成
uv run qwen-image-mps generate -p "宇宙ステーション" --seed 42 --num-images 2 --quantization Q4_0
```

#### アスペクト比の指定

```bash
# 正方形（1:1）
uv run qwen-image-mps generate -p "ポートレート写真" --aspect 1:1 --fast --quantization Q4_0

# 縦長（9:16）
uv run qwen-image-mps generate -p "高層ビル" --aspect 9:16 --fast --quantization Q4_0

# ワイド（16:9、デフォルト）
uv run qwen-image-mps generate -p "パノラマ風景" --aspect 16:9 --fast --quantization Q4_0
```

#### 画像編集

```bash
# 基本的な編集
uv run qwen-image-mps edit -i input.jpg -p "空を夕焼けに変更"

# 高速編集
uv run qwen-image-mps edit -i photo.png -p "山に雪を追加" --fast

# アニメ風変換
uv run qwen-image-mps edit -i photo.jpg --anime --fast
```

### Gradio UI（Webインターフェース）

ブラウザベースのグラフィカルインターフェースを起動：

```bash
uv run qwen-image-mps-gradio
```

ブラウザで http://127.0.0.1:7860 を開き、以下の手順で使用：

1. **Generateタブ**を選択
2. プロンプトを入力（例：「富士山と桜、美しい風景」）
3. **Quantization (GGUF)** ドロップダウンから **Q4_0** を選択
4. **Fast** または **Ultra-fast** にチェック（推奨）
5. **Generate** ボタンをクリック

生成された画像は `output/` ディレクトリに保存されます。

## パフォーマンスチューニング

### メモリ不足が発生する場合

1. **量子化を使用**: `--quantization Q4_0` を追加
2. **より軽量な量子化**: `--quantization Q2_K` に変更
3. **生成枚数を減らす**: `--num-images` の値を小さくする
4. **高速モードを使用**: `--fast` または `--ultra-fast` を追加

### 生成速度を上げる場合

1. **高速モード**: `--fast`（8ステップ）
2. **超高速モード**: `--ultra-fast`（4ステップ）
3. **量子化**: `--quantization Q4_0`（推論速度も向上）

## コマンドオプション

### generate コマンド

```bash
uv run qwen-image-mps generate --help
```

主なオプション：
- `-p, --prompt`: プロンプトテキスト
- `-np, --negative-prompt`: ネガティブプロンプト
- `-s, --steps`: 推論ステップ数（デフォルト: 50）
- `-f, --fast`: 高速モード（8ステップ）
- `-uf, --ultra-fast`: 超高速モード（4ステップ）
- `--seed`: ランダムシード（再現性のため）
- `--num-images`: 生成する画像数（デフォルト: 1）
- `--aspect`: アスペクト比（1:1, 16:9, 9:16, 4:3, 3:4, 3:2, 2:3）
- `--quantization`: GGUF量子化レベル（Q2_K〜Q8_0）
- `--outdir`: 出力ディレクトリ（デフォルト: ./output）

### edit コマンド

```bash
uv run qwen-image-mps edit --help
```

主なオプション：
- `-i, --input`: 入力画像パス（必須）
- `-p, --prompt`: 編集指示
- `-o, --output`: 出力ファイル名
- `--anime`: アニメ風変換モード
- `--fast`, `--ultra-fast`: 高速編集モード

## トラブルシューティング

### "CUDA error: out of memory" が発生する場合

量子化オプションを使用してください：

```bash
uv run qwen-image-mps generate -p "your prompt" --quantization Q4_0 --fast
```

### モデルのダウンロードが失敗する場合

Hugging Faceにログインしてください：

```bash
huggingface-cli login
```

### PyTorchがCUDAを認識しない場合

インストールを確認：

```bash
uv run python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

False の場合は、環境を再構築：

```bash
rm -rf .venv
uv sync
```

## 参考情報

### オリジナルリポジトリ

- https://github.com/ivanfioravanti/qwen-image-mps

### モデル

- [Qwen/Qwen-Image](https://huggingface.co/Qwen/Qwen-Image) - メイン生成モデル
- [city96/Qwen-Image-gguf](https://huggingface.co/city96/Qwen-Image-gguf) - GGUF量子化モデル
- [lightx2v/Qwen-Image-Lightning](https://huggingface.co/lightx2v/Qwen-Image-Lightning) - 高速化LoRA

### DGX Sparkセットアップガイド

- [DGX SparkでPyTorch GPU環境を構築する方法](https://zenn.dev/karaage0703/articles/985ddbd8fa15d3)

## ライセンス

オリジナルプロジェクトのライセンスに準拠します。

## 貢献

バグ報告や機能リクエストは、GitHubのIssuesページまでお願いします。
