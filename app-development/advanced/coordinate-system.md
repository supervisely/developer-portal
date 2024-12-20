# Coordinate system

## Overview

Sometimes, when labeling an image on the Supervisely platform, you may notice that the labels appear slightly different in the labeling tool compared to the image after using the export app or applying a drawing method from the SDK. This discrepancy is more noticeable when labeling small objects or low-resolution images. The reason for this difference is that the labeling tool uses a more precise subpixel coordinate system (floating-point coordinates) to display the labels, while the Supervisely SDK uses a pixel coordinate system (integer coordinates). This conversion is done for compatibility with OpenCV and other libraries that require integer coordinates for processing data, with a maximum potential offset of 1px.

<div align="center">
<img src="https://github.com/user-attachments/assets/6dd7156f-ba35-469e-b65b-f7e8ed5be395" width="500"/>
  <p><em>Illustration of bbox coordinate conversion after uploading to instance</em></p>
</div>

Subpixel coordinate system is a coordinate system that allows you to specify coordinates with floating-point precision. In this system, the origin is typically at the top-left corner of the image (coordinate (0, 0)), and the x-axis and y-axis extend to the right and downward, respectively. This system is used for more precise positioning of elements, such as labels, within an image, allowing for placement between pixels.

Pixel coordinate system is a coordinate system where coordinates are specified with integer precision, corresponding to the center of pixels. In this system, the origin is usually considered at the top-left corner of the image (coordinate [0, 0]), and each integer coordinate corresponds to a specific pixel, with [0.5, 0.5] representing the center of the top-left pixel. The x-axis and y-axis extend to the right and downward, respectively. This system is used for positioning elements, such as labels, directly on pixels, with each coordinate corresponding to a specific pixel and used by libraries like OpenCV for positioning elements directly on pixels with integer precision.

![coordinate-system-3](https://github.com/user-attachments/assets/ed65b2ff-cd14-4da9-a437-0703f84abe0e)

## How coordinates are converted in Supervisely

Upon downloading annotations from Supervisely, all labels initially defined with floating-point coordinates in the subpixel system are automatically converted to the pixel coordinate system (integer coordinates). This conversion typically involves taking the integer part of the coordinates, effectively rounding them down. However, rectangles undergo a special rounding process to ensure a more accurate representation.

When you are working with the Supervisely SDK, it is important to use the pixel coordinate system when creating geometry object instances, as the SDK expects integer coordinates to maintain compatibility with libraries like OpenCV and ensure your labels are displayed correctly when drawing labels locally on images or displayed in the labeling tool.

When annotations are uploaded back to Supervisely, the system converts the coordinates from the pixel coordinate system back to the subpixel system, allowing for precise display in the labeling tool.

### Bitmap base geometries

For `Bitmap` and `AlphaMask` origin coordinates are always integer, so no conversion is needed since pixel coordinates are always integer, the origin coordinates are already in pixel coordinate system. The bitmap is always aligned to the pixel grid.

### Vector geometries

For `Point`, `Line`, `Polygon`, `Keypoint`, `Cuboid` geometries the coordinates will be converted to pixel coordinate system (integer coordinates) by taken the integer part of the coordinates, in other words coordinates are always rounded down.

For example we have a label with polyline geometry and the following coordinates in subpixel coordinate system will be converted to pixel coordinate system as follows:

![Line](https://github.com/user-attachments/assets/da0d36cd-1569-4a6d-96e7-fcc8903977e5)

### Rectangle

`Rectangle` geometry is a special case. The coordinates of the rectangle are represented by two points: top left and bottom right corners. The corners of the rectangle will be rounded first before converting to pixel coordinate system to provide more accurate representation of the rectangle with the following rules:

**Left and Top:**
- Left will be rounded down if the decimal part is lesser than 0.7, it will include the horizontal pixel if it is in the range, otherwise it will be rounded up and this pixel will not be included.
- Top will be rounded down if the decimal part is lesser than 0.7, it will include the vertical pixel if it is in the range, otherwise it will be rounded up and this pixel will not be included.

**Right and Bottom:**
- Right will be rounded up if the decimal part is greater than 0.3, it will include the horizontal pixel if it is in the range, otherwise it will be rounded down and this pixel will not be included.
- Bottom will be rounded up if the decimal part is greater than 0.3, it will include the vertical pixel if it is in the range, otherwise it will be rounded down and this pixel will not be included.
- Convert right and bottom coordinates to pixel coordinate system by subtracting 1 from the rounded subpixel coordinates.

![Rectangle](https://github.com/user-attachments/assets/4b7f4e9b-d3a4-45fd-a4d4-63ea320b39f5)

If left and right and top and bottom are in the same pixel coordinate, the rectangle will be expanded to cover whole pixel.

![Rectangle coordinates within one pixel](https://github.com/user-attachments/assets/118ef4b1-3569-42ba-9355-89b7fbf58c2b)

These rules ensure that the rectangle is represented as accurately as possible in the pixel coordinate system while maintaining its intended coverage.

Rounding Example:

Input rectangle coordinates in subpixel system:

```text
Subpixel          Rounded (still subpixel)    Pixel
--------------    ------------------------    ------
[6.43, 5.43]   ->          [6, 5]          -> [6, 5]
[8.43, 7.43]   ->          [8, 7]          -> [7, 6]
```
