[中文](./readme_zh.md)

# Halcon Machine Vision-Based License Plate Recognition (LPR)

## Introduction

This project implements a License Plate Recognition (LPR) system using the Halcon machine vision library, focusing on basic character identification and high success rates for standard formats.

### Key Features

* **Localization Method:** Identifies and localizes license plates using manual threshold adjustment and area-based filtering.
* **Success Rate:** Achieves a high recognition success rate of up to **90%**.
* **Character Support:** Currently **does not support Chinese characters**. Localization and character recognition for specific templates can be achieved through additional template training.

---

## Halcon Key Code Implementation

This section contains the core Halcon code logic for image processing, including pre-processing, segmentation, localization, and OCR.

```halcon
dev_close_window()
dev_clear_window()
read_image(image,'C:/Users/Administrator/Desktop/halcon/chepai4.jpg')
get_image_size(image, Width, Height)
dev_open_window_fit_size (0, 0, Width, Height, -1, -1, WindowHandle)
dev_display(image)

* Convert the image to the RGB three-channel format
decompose3(image, r, g, b)

* Convert to HSV (Hue, Saturation, Value)
trans_from_rgb(r, g, b, h, s, v, 'hsv')

* Enhance image contrast (on the saturation channel)
emphasize(s, ImageEmphasize, Width, Height, 1)

threshold(ImageEmphasize, Region, 255, 255)
connection(Region, ConnectedRegions)
closing_rectangle1(ConnectedRegions, RegionClosing, 50, 50)

* Select the region with the maximum area (expected license plate)
select_shape_std(RegionClosing, SelectedRegions, 'max_area', 70)

* Fill holes in the selected region
fill_up(SelectedRegions, RegionFillUp)

* Reduce image domain to the license plate area
reduce_domain(ImageEmphasize, RegionFillUp, ImageReduced)
reduce_domain(image, RegionFillUp, ImageReduced1)

* Start character segmentation/identification
threshold(ImageReduced, Region1, 0, 100)
connection(Region1, ConnectedRegions1)

* Filter the character regions based on expected size/area
select_shape (ConnectedRegions1, SelectedRegions1, 'area', 'and', 4014, 19840.76)

* Sort the regions for sequential OCR reading
sort_region(SelectedRegions1, SortedRegions, 'character', 'true', 'row')

* Invert the image (for better OCR results if characters are dark on light background)
invert_image(ImageReduced1, ImageInvert)

* Perform OCR (using a pre-trained MLP model)
read_ocr_class_mlp('Industrial_0-9A-Z_NoRej.omc', OCRHandle)
do_ocr_multi_class_mlp(SortedRegions, ImageInvert, OCRHandle, Class, Confidence)
```

## Screenshots
Demo images showcasing the system's output and processing steps.
![img](./img/1.png)
![img](./img/2.png)
