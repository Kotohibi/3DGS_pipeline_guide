# 概要
3D Gaussian Splatting(以下3DGS)の技術は日進月歩で、様々な手法が提案されています。しかし単眼カメラを用いた3DGSは実スケールがでない課題が依然として存在しています。本記事では、実スケールの3DGSを作成するための一例のワークフローを紹介します。

# 対象読者
* 以下の記事を読んで、既に機材を持っている、3DGSの基礎的な知識がある方、実際制作を試したことがある方を対象としています
    * [JP]Easy&Fast 3D Gaussian Splatting workflow with 360° Camera
* LiDARを使った点群空間キャプチャの知識、経験がある方はより理解が深まると思います

# 追加で必要な物
* CloudCompare

* 点群空間キャプチャの機材（例：LiDARセンサー、ToFカメラなど）
    * Livox mid 360 + LiDAR SLAM
    * 既製品のLiDARスキャナー
    * iPhone Pro シリーズのLiDARセンサー




