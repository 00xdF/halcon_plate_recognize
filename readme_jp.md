# Halconマシンビジョンに基づくナンバープレート認識 (License Plate Recognition, LPR)

## 概要 (Introduction)

* 手動のスライド式しきい値と面積による位置特定を通じてナンバープレートを認識します。
* 認識成功率は**最大90%**に達します。
* 中国語の文字認識には対応していません。文字認識はテンプレートトレーニングにより対応可能です。

---

## Halconの主要コード (Halcon Key Code Implementation)

このセクションには、画像の前処理、セグメンテーション、位置特定、OCRを含むHalconのコアロジックが含まれています。

```halcon
dev_close_window()
dev_clear_window()
read_image(image,'C:/Users/Administrator/Desktop/halcon/chepai4.jpg')
get_image_size(image, Width, Height)
dev_open_window_fit_size (0, 0, Width, Height, -1, -1, WindowHandle)
dev_display(image)

* 画像をRGB 3チャンネルに変換
decompose3(image, r, g, b)

* HSV (色相、彩度、明度) に変換
trans_from_rgb(r, g, b, h, s, v, 'hsv')

* 画像のコントラストを強調
emphasize(s, ImageEmphasize, Width, Height, 1)

threshold(ImageEmphasize, Region, 255, 255)
connection(Region, ConnectedRegions)
closing_rectangle1(ConnectedRegions, RegionClosing, 50, 50)

* 最大面積に基づいて領域を選択
select_shape_std(RegionClosing, SelectedRegions, 'max_area', 70)

* 選択された領域の穴を埋める (塗りつぶし)
fill_up(SelectedRegions, RegionFillUp)

* 画像領域をナンバープレート部分に限定
reduce_domain(ImageEmphasize, RegionFillUp, ImageReduced)
reduce_domain(image, RegionFillUp, ImageReduced1)

* 認識処理
threshold(ImageReduced, Region1, 0, 100)

* 接続された領域を分離
connection(Region1, ConnectedRegions1)

* 表示領域をフィルタリング (面積による文字領域の絞り込み)
select_shape (ConnectedRegions1, SelectedRegions1, 'area', 'and', 4014, 19840.76)

* 領域を文字順に並び替え
sort_region(SelectedRegions1, SortedRegions, 'character', 'true', 'row')

* 画像を反転
invert_image(ImageReduced1, ImageInvert)

* 認識開始 (OCR)
read_ocr_class_mlp('Industrial_0-9A-Z_NoRej.omc', OCRHandle)
do_ocr_multi_class_mlp(SortedRegions, ImageInvert, OCRHandle, Class, Confidence)

```
## スクリーンショット (Screenshots)
![img](./img/1.png)
![img](./img/2.png)
