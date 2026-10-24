# AGENTS.md

## 更新手順

- ※現在メンテナンス対象としているバージョン
    - Blender 5.2: ./buckets/blender5.2.json
    - Blender 4.5: ./buckets/blender4.5.json
- Blenderのreleaseページを確認する
    - https://download.blender.org/release/Blender5.2/
    - https://download.blender.org/release/Blender4.5/
    - 一覧は辞書順なので「最後の行が最新」にはならない（`blender-4.5.10-windows-x64.zip` は `blender-4.5.9` より**前**、`4.5.1` の直後に並ぶ）。`-windows-x64.zip` のファイル名からバージョン番号を取り出して数値比較し、最大値を選ぶこと
- 新しいバージョンがあれば sha256 を取得する
    - 取得元は `https://download.blender.org/release/Blender5.2/blender-<version>.sha256` のように、release 直下にある `Blender<メジャー.マイナー>` ディレクトリ内の `blender-<version>.sha256` を直接叩く。全プラットフォームの hash 一覧が返るので、その中の `blender-<version>-windows-x64.zip` の行を使う
    - ※`blender-<version>-windows-x64.zip.sha256` という名前では存在しない（404 になる）。対象ファイル名に `.sha256` を接尾辞した URL は作らないこと
    - ※blender.org のダウンロードページの Checksums リンクは最新版に追いついていないことがある（新しいリリースでも旧バージョンの `.sha256` を指す）。hash の一次ソースは download.blender.org 上の `.sha256` にする
- JSON内の下記の項目を更新する
    - version
    - architecture.64bit.url
    - architecture.64bit.hash
    - architecture.64bit.extract_dir ※バージョン番号だけ更新すればよい。zipを実際にダウンロードして構造を確認する必要はない。
- 更新後に JSON の形式チェックをする（scoop install で実際にダウンロードして確認する必要はない）
    - `Get-Content buckets/blender5.2.json -Raw | ConvertFrom-Json`
- コミットする
    - メッセージは既存の慣習に合わせて `Update (<5.2系の最新版>, <4.5系の最新版>)` の形にする（例: `Update (5.2.3, 4.5.15)`）
    - 両方の bucket を同時に更新した場合は 1 コミットにまとめてよい（過去の履歴がそうなっている）

## 参考コマンド（Windows / PowerShell）

```powershell
# 対象系の最新バージョン番号（数値ソートするので辞書順の罠を回避できる）
foreach ($d in "Blender5.2", "Blender4.5") {
    $v = (Invoke-WebRequest "https://download.blender.org/release/$d/" -UseBasicParsing).Links.href |
        Where-Object { $_ -like "blender-*-windows-x64.zip" } |
        ForEach-Object { [version]($_.Split("-")[1]) } |
        Sort-Object -Descending | Select-Object -First 1
    "$d latest => $v"
}

# sha256 の取得（この中の windows-x64.zip の行を architecture.64bit.hash に入れる）
(Invoke-WebRequest "https://download.blender.org/release/Blender4.5/blender-4.5.15.sha256" -UseBasicParsing).Content
```

## 参考コマンド（macOS / bash・zsh）

```bash
# 対象系の最新バージョン番号（数値ソートするので辞書順の罠を回避できる）
for d in Blender5.2 Blender4.5; do
    v=$(curl -s "https://download.blender.org/release/$d/" \
        | grep -oE 'blender-[0-9]+(\.[0-9]+)+-windows-x64\.zip' \
        | sed -E 's/blender-(.*)-windows-x64\.zip/\1/' \
        | sort -V | tail -1)
    echo "$d latest => $v"
done

# sha256 の取得（この中の windows-x64.zip の行を architecture.64bit.hash に入れる）
curl -s "https://download.blender.org/release/Blender4.5/blender-4.5.15.sha256" | grep 'windows-x64.zip'

# JSON の形式チェック
jq empty buckets/blender5.2.json
# jq が入っていない環境では python3 でもよい
python3 -c 'import json,sys; json.load(open(sys.argv[1]))' buckets/blender5.2.json
```

- `sort -V` は macOS 標準の `sort` が対応していない古い環境がある。その場合は `sort -V` を `python3` のバージョン比較に置き換える。
