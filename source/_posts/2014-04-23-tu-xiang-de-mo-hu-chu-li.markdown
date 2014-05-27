---
layout: post
title: "图像的模糊处理"
date: 2014-04-23 14:44
comments: true
sharing: true
footer: true
categories: [OpenCV]
---


+ 一些相关理论的介绍，容易理解

```
http://www.cnblogs.com/justany/archive/2012/11/21/2779978.html
```

<!-- more -->

```
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <cv.h>

using namespace std;
using namespace cv;

int main(int argc, char** argv)
{
    if (argc != 2) {
        cout << "Usage: display_image ImageToLoadAndDisplay" << endl;
        return -1;
    }

    Mat image;
    image = imread(argv[1], CV_LOAD_IMAGE_COLOR);

    if( !image.data )
    {
        cout << "Could not open or find the image" << std::endl;
        return -1;
    }
    Mat outimg;
    //blur(image, outimg, Size(10, 10));
    //GaussianBlur( image, outimg, Size(5,5), 1000, 1000 );
    //medianBlur( image, outimg, 9);
    bilateralFilter ( image, outimg, 9, 3*10, 3*10 );
    //cvtColor(image, outimg, CV_RGB2GRAY);

    namedWindow( "Display Image", CV_WINDOW_AUTOSIZE );
    namedWindow( "Display Out Image", CV_WINDOW_AUTOSIZE );
    imshow( "Display Image", image );
    imshow( "Display Out Image", outimg);
    waitKey(0);
    return 0;
}
```
