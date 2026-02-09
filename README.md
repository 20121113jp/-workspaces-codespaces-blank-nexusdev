# 🍎 GitHub Codespaces Macクラウド（macOS VM）ガイド（完全版）

GitHub Codespaces上で**クラウドのMac（macOS仮想環境）**を動かすための「最終決定版」ガイドです。

## 0. 準備：容量の確保と保存場所の作成
無料サーバーは容量が厳しいため、まずゴミ掃除をしてから、データを刻む場所を作ります。

```bash
# 不要なデータを消して容量を空ける
docker system prune -a -f

# Macのデータを保存するフォルダを作る
mkdir -p ~/mac-storage
```

## 1. Macを「低燃費・保存モード」で起動
1つ目のターミナルに貼り付けます。  
※メモリを2GBに抑え、データを先ほどのフォルダに紐付けます。

```bash
docker run -it \
    --device /dev/kvm \
    -p 50922:22 \
    -p 5900:5900 \
    -v ~/mac-storage:/var/lib/docker/volumes \
    -e RAM=2 \
    -e VNC_ADDRESS=0.0.0.0:5900 \
    sickcodes/docker-osx:auto
```

> ⏳ 待ち時間: 10分〜20分。数GBのダウンロード後、白い文字が大量に流れ始めます。

## 2. ブラウザで見れるように「窓」を作る
2つ目の新しいターミナルを開いて実行します。

```bash
# ツールをインストール
sudo apt update && sudo apt install -y novnc python3-websockify

# 映像をブラウザ用に変換
websockify --web /usr/share/novnc/ 6080 localhost:5900 &
```

## 3. 画面にアクセスして初期化する
- 下の 「Ports」 タブから 6080 の地球儀マークをクリック。
- ブラウザが開いたら 「Connect」 を押す。
- ディスクの初期化（重要）:
  - Disk Utility を開く。
  - 一番上の大きなディスクを選び Erase（名前: Macintosh HD, 形式: APFS）。
- インストール: Reinstall macOS を選び、気長に待つ（1時間〜）。

## 4. デスクトップ起動後の仕上げ（GenSMBIOS）
Macのデスクトップが表示されたら、いよいよ「本物のMac」にする作業です。

- Mac内のブラウザで GenSMBIOS を検索。
- 実行して、iMacなどのモデル名から「シリアル番号」を生成して適用。
- これで、iCloudやApp Storeが使える可能性が開けます（AWDLやBluetooth LEは物理的に不可）。

## ⚠️ 運用のルール
- 終わる時: ターミナルで Ctrl + C を押して終了。これで ~/mac-storage にデータが残ります。
- 再開する時: 再び「手順1」のコマンドを打つだけです。
- 注意: 動作は非常に重いです。また、GitHubの負荷制限に引っかからないよう、使いすぎには注意してください。
