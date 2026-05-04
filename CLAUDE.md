# スレッズアフィリ（affiliate-jp）— Claude Code 指示書

## プロジェクト概要
日本語 Amazon アフィリエイトサイト（AIセラーツール比較）の記事を  
**Meta Threads に自動投稿**して集客するシステムの構築・運用。

## ゴール（確定済み・再確認不要）
1. `blog/*.html` の記事を Threads 投稿文（500文字以内）に変換
2. Threads Graph API で自動投稿（1日1〜3件）
3. 投稿済みをログ管理してスキップ
4. launchd で定期実行

## 参照実装（完成品）
`~/amazon-affiliate/` に同構成のパイプラインが動いている。
- `threads_poster.py` — Threads API 投稿ロジック
- `generator.py` — 投稿文生成
- `run.py` — オーケストレーター
- `com.shoichi.amazon-affiliate.plist` — launchd 設定

**このプロジェクトも同じ構成で実装する。**  
違いはブログディレクトリが `~/affiliate-jp/blog/` であること。

## 環境変数（~/.zprofile に設定済み）
```
THREADS_ACCESS_TOKEN=...
THREADS_USER_ID=...
ANTHROPIC_API_KEY=...
```

## 進め方（毎回この順で実行・確認不要）

### STEP 1: 現状確認
```bash
ls ~/affiliate-jp/blog/*.html | wc -l  # 記事数
ls ~/affiliate-jp/pipeline/ 2>/dev/null || echo "pipeline未作成"
```

### STEP 2: pipeline ディレクトリ構築
```bash
mkdir -p ~/affiliate-jp/pipeline

# ① そのままコピーして使えるファイル（Write不要）
cp ~/amazon-affiliate/threads_poster.py ~/affiliate-jp/pipeline/
cp ~/amazon-affiliate/requirements.txt ~/affiliate-jp/pipeline/

# ② コピー後に Edit で修正するファイル（Write で一から書かない）
cp ~/amazon-affiliate/generator.py ~/affiliate-jp/pipeline/generator.py
# → Edit で BLOG_DIRS を affiliate-jp/blog に変更
cp ~/amazon-affiliate/run.py ~/affiliate-jp/pipeline/run.py
# → Edit で import / path を affiliate-jp 向けに変更

# ③ 新規作成（小さいので Write OK）
touch ~/affiliate-jp/pipeline/threads_posted.json
echo '{}' > ~/affiliate-jp/pipeline/threads_posted.json
```

**⚠ 重要: 80行超のコードは Write ツールで直接生成しない。必ず cp → Edit で対応する。**

### STEP 3: launchd 設定
`~/Library/LaunchAgents/com.shoichi.affiliate-jp.plist` を作成  
→ 1日3回（09:00 / 13:00 / 19:00 JST）に `run.py` を実行

### STEP 4: テスト投稿
```bash
cd ~/affiliate-jp/pipeline
python3 run.py --preview  # 実際には投稿しない
```

### STEP 5: launchd 登録
```bash
launchctl load ~/Library/LaunchAgents/com.shoichi.affiliate-jp.plist
```

## 自律実行ルール（最重要）

**以下のことは確認せずに実行する：**
- コードの生成・修正
- `--preview` での確認実行
- `python3 run.py` の実行（実際の投稿含む）
- launchd への登録・unload/load
- `git add / commit / push`

**止まって聞いてよいのはこの場合だけ：**
- Threads アクセストークンが存在しない / 期限切れ
- API エラーが3回連続して発生

## 現在の状態
- `~/affiliate-jp/blog/` に記事 HTML あり
- `pipeline/` ディレクトリなし（未作成）
- Threads 投稿は未実装
- **次のアクション: STEP 2 から開始**
