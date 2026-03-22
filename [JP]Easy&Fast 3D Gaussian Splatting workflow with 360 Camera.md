# 概要
本ワークフローは全天球画像:Equirectangular(正距円筒図法)画像を使って、ロバストかつ比較的高速にカメラアラインメント(SfM)を行い、3D Gaussian Splatting(3DGS)の学習を行うワークフロー例を示します
# 参考
* https://x.com/naribubu/status/2034937726756430125
* https://x.com/naribubu/status/2015376645360849394
* https://x.com/naribubu/status/2020138127084695876
* https://x.com/naribubu/status/2017883648075391214 (As for --yaw-offset otpion)
# 必要な物
* 360° カメラ
    * DJI OSMO360
    * Insta360
 
* Metashape Standard
    * 全天球画像を直接SfM可能で、非常に高速&ロバストです
    * https://www.agisoft.com/features/standard-edition/

* 3D Gaussian Splatting software
    * Postshot: https://www.jawset.com/
    * LichtFeld Studio(LFS): https://github.com/MrNeRF/LichtFeld-Studio
    * Brush: https://github.com/ArthurBrussee/brush
* 動画から静止画切り出しツール
    * Extract Sharpest Frame
        * https://github.com/Kotohibi/Extract_sharpest_frame
    * BOOTH版 Windows Binary Edition: https://kotohibi-cg.booth.pm/
* Metashape 360 SfMからCOLMAP形式のCubemap変換ツール
    * Metashape 360 to COLMAP Converter
        * https://github.com/Kotohibi/Metashape_360_to_COLMAP_plane
    * BOOTH版 Windows Binary Edition: https://kotohibi-cg.booth.pm/

# 動画撮影する(e.g. OSMO360)
カメラを自撮り棒に付けて、キャプチャしたい範囲をゆっくりと歩きます
動画設定はD-Log M, 30fps以上の撮影がお勧めです
# 動画を現像する
### DJI Studioに撮影データを取り込み、現像処理(色復元)をします
下記画像の赤枠の設定を実施。それ以外はデフォルトでOKです
![](https://storage.googleapis.com/zenn-user-upload/60fc28c6c26e-20260322.png)
### 動画を書き出す
全天球動画としてMP4ファイルで動画を書き出します。設定例を下図に示します
![](https://storage.googleapis.com/zenn-user-upload/b80d49f8fef6-20260322.png)
# 動画から静止画を切り出す
動画から静止画を切り出す方法は様々あります。お好きな方法を調査、選択してください
ここでは私が公開しているツールをご紹介します
**Extract Sharpest Frame**は指定フレーム間隔で一番シャープな画像を切り出すツールです
* **新機能はBOOTH版を優先的にupdateしております**
![](https://storage.googleapis.com/zenn-user-upload/a130e195b3a2-20260322.png)

|主要項目|説明|
|---|---|
|Video file|全天球動画を選択|
|Output folder|静止画を切り出す先のフォルダを指定|
|Scale width|全動画フレームの画像のシャープさを計算する時の画サイズです。大きくした方が緻密に計算されます。注）切り出し画像は常にオリジナル動画と同じ画サイズで切り出されます|
|Chunk size|静止画を切り出す間隔を指定。30fps動画で30を指定すると1秒間隔で切り出されます|
|Workers|画像のシャープさを計算する時のプロセス数を指定。4前後がお勧めです。|
|Analysis only|画像のシャープさの計算のみ行います。出力フォルダに計算結果（メタ情報）が保存されます。次回から出力フォルダにメタ情報があると、解析フェーズをスキップして画像切り出しができます。Chunk sizeを調整した場合に有効です。|
|Run|処理実行|

### 実行結果
このように静止画が切り出されます。次StepのSfMの結果を見てChunk sizeを再調整してください
![](https://storage.googleapis.com/zenn-user-upload/0b75a72fb38b-20260322.png)
# カメラアラインメント(SfM)をする
SfMは全天球画像をダイレクトに処理できるMetashape Standardを使います
### 切り出した全天球画像をロードします
[Workflow]->[Add Folder]
![](https://storage.googleapis.com/zenn-user-upload/b39e5a8c4bc7-20260322.png)
### Camera TypeをSphericalに変更
[Tools]->[Camera Calibration]を選択して、Camera typeをSphericalを選択します
![](https://storage.googleapis.com/zenn-user-upload/25897fe5c593-20260322.png)
### SfMのパラメータ設定
[Workflow]->[Align Photos]
私が良く使うパラメータ例を示します
![](https://storage.googleapis.com/zenn-user-upload/40f4ec313cf7-20260322.png)
### 実行
OKボタンを押して、SfMを実行します
結果例を示します。球体マークがそれぞれの全天球画像に相当します
![](https://storage.googleapis.com/zenn-user-upload/026d9b7f7c2b-20260322.png)
### SfM結果のエクスポート
    * Camera情報のエクスポート
      [File]->[Export]->[Export Cameras]でAgisoft XML(*.xml)を選択して保存します
    * Point Cloudのエクスポート
      [File]->[Export]->[Export Point Cloud]でStandord PLY(*.ply) を選択して保存します
# COLMAP Cubemapに変換する
MetashapeのSfM結果からCOLMAP形式に6方向画像のCubemap展開します
ここでは私が公開しているツールをご紹介します
**Metashape 360 to COLMAP Converter**
* **新機能はBOOTH版を優先的にupdateしております**

### 設定①
![](https://storage.googleapis.com/zenn-user-upload/e7cf4c196027-20260322.png)

|主要項目|説明|
|---|---|
|Input Images Folder|切り出した全天球画像フォルダを指定|
|Metashape XML|SfM結果のCamera.xmlを指定|
|PLY File|SfM結果のpoint_cloud.plyを指定|
|Output Folder|Cubemap展開先のフォルダを指定|
|Crop Size|6方向に切り出す画サイズ、OSMO360の8K動画の場合は1920でOK|
|FoV|6方向に切り出す視野角、90°でOK|
|Max Images|全天球画像の処理枚数上限、動作テストする際に小さい値を指定下さい|
|Image Range|処理対象の全天球画像を範囲指定可能、部分的に処理する際に指定ください|
|Workers|処理のプロセス数、お使いのCPUのコア数に応じて増減させてください|
|Yaw Offset|CubemapのYaw角度にバリエーションを持たせることが可能。Cubemap毎に指定角度が加算されます。5～30°の範囲がお勧めです|
|Save Config|上記設定値を設定ファイルとして保存可能|
|Run Conversion|Cubemap変換処理を開始|

### 設定②
* 人物や自動車等の動体をマスクすることが可能です
* 特に360 Cameraは自身が映りこむ為、マスク生成は重要な作業になります


* **下記設定内容はBOOTH版で説明します、Github版より機能強化されています**
![](https://storage.googleapis.com/zenn-user-upload/51916a668c5a-20260322.png)

|主要項目|説明|
|---|---|
|Mask Pass Mode|Singleは全天球画像で動体を認識します。高速ですが精度は低いです。Dualは全天球画像とCubemap画像両方で動体を認識します。処理量は多いですが、認識精度が高いです|
|Merge Mode|Dualモード時にマスクを結合させるモードです。unionは双方の単純結合、refineはCubemapのマスクをベースに全天球マスクが統合されます。refineがお勧めです|
|YOLO Class IDs|検出した動体IDを指定します。 0: person, 1: bicycle, 2: car, etc.. 様々な動体を指定可能です。https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/datasets/coco.yaml|
|YOLO Confidence|閾値を下げると認識率は上がりますが、ノイズも増えます|
|Enable overexposure mask|白飛び画素は3DGS学習時にノイズ成分になる場合があります、除去したい場合に有効にしてください|

### 実行
処理実行後、正常に完了すると出力フォルダに以下のようなフォルダとファイルが生成されます
![](https://storage.googleapis.com/zenn-user-upload/fc379b61d4eb-20260322.png)
  
# 3D Gaussian Splattingの学習
ここではPostshotを使って説明します
### Cubemapの取り込み
![](https://storage.googleapis.com/zenn-user-upload/cb6564b73383-20260322.png)
* まず、Imagesフォルダ, cameras.txt, images.txt, points3D.txtを選択してPostshotにドラッグ&ドロップします
### Mask設定
![](https://storage.googleapis.com/zenn-user-upload/14a3f080601f-20260322.png)

* 次にmasksフォルダをPostshotのImage Masksの領域にドラッグ&ドロップします
Mask Modeは**Remove Background**を選択します


### Cubemapの取り込み結果
![](https://storage.googleapis.com/zenn-user-upload/41e8ba454bde-20260322.png)
* Cubemapが正常に取り込みされると、上記の様な画面が出ます
### 3DGS学習開始
![](https://storage.googleapis.com/zenn-user-upload/d298d3ed5248-20260322.png)
* 私が広域3DGSで使う学習パラメータ例を示します。シーンに合わせてパラメータは調整してください
### 3DGS学習結果
![](https://storage.googleapis.com/zenn-user-upload/420154d101ed-20260322.png)
* 学習が進むと、3DGSが見えてくると思います！

# 最後に
3DGSの手法は様々で、本記事は一例に過ぎません。最新の情報は私のXで発信していきます
是非ご自身でも調査してより良い手法を開発してください。Enjoy 3DGS :) 
* my X: https://x.com/naribubu