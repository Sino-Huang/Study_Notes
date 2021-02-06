# OpenCV simple project

## Project 1 Virtual Paint

```cpp
#include <iostream>
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/objdetect.hpp"
#include "opencv2/videoio.hpp"
#include "fmt/format.h"
using namespace fmt;
using namespace cv;

void findColor(Mat img);
Point getContours(Mat img, int& count);




////////////// Project 1 ////////////////


void drawOnCanvas(Mat img);

// hmin, hmax, smin, smax, vmin, vmax
static std::vector<std::vector<int>> myColors{{74, 179, 134, 242, 129, 255}, // red
                                              {34, 73,  47,  242, 98,  255}, // green
};

static std::vector<Scalar> myColorValues{{0, 0,   255},
                                         {0,   255, 0}};

static int hmin=0, smin=0, vmin=0;
static int hmax=179, smax=255, vmax=255;

static Mat imgHSV;
static Mat mask;
static Mat imgDil;

// {{x, y, color value}}
static std::vector<std::vector<int>> newPoints;


auto onChangePrint = [](int a, void *pt) {
    print("{}, {}, {}, {}, {}, {}\n", hmin, hmax, smin, smax, vmin, vmax);
    fflush(stdout); //flush out since we have while loop
};



int main() {
    VideoCapture cap(0, CAP_V4L2);
    // set mjpg format faster fps
    cap.set(cv::VideoCaptureProperties::CAP_PROP_FOURCC, cv::VideoWriter::fourcc('M', 'J', 'P', 'G'));
    Mat img;

    // now give trackbar and show img for color filter
    namedWindow("UsefulWindow", (640, 200)); //create Window
    createTrackbar("Hue Min", "UsefulWindow", &hmin, 179, onChangePrint);
    createTrackbar("Hue Max", "UsefulWindow", &hmax, 179, [](int a, void* pt){
        print("{} {} {} {} {} {}", hmin, hmax, smin, smax, vmin, vmax);
    });
    createTrackbar("Sat Min", "UsefulWindow", &smin, 255, onChangePrint);
    createTrackbar("Sat Max", "UsefulWindow", &smax, 255, onChangePrint);
    createTrackbar("Val Min", "UsefulWindow", &vmin, 255, onChangePrint);
    createTrackbar("Val Max", "UsefulWindow", &vmax, 255, onChangePrint);

    cap.read(img);
    imshow("Image", img);
    // set mouse double click event callback
    setMouseCallback("Image",  [](int event, int x, int y, int flags, void *userdata){
        if (event == EVENT_LBUTTONDBLCLK) {
            newPoints.clear();
        }
    });

    while (true) {
        cap.read(img);
//        print("img size {}, {}\n", img.size[0], img.size[1]);
//        fflush(stdout);
        // flip mirror
        flip(img, img, 1);
        findColor(img);
        drawOnCanvas(img);
        imshow("Image", img);

        waitKey(1);
    }
    return 0;
}

void drawOnCanvas(Mat img) {
    for (int i = 0; i < newPoints.size(); ++i) {
        circle(img, Point(newPoints[i][0], newPoints[i][1]), 10, myColorValues[newPoints[i][2]], FILLED);
    }
}


void findColor(Mat img) {
    // change to color to HSV
    cvtColor(img, imgHSV, COLOR_BGR2HSV);
    // loop myColors
    int count = 0;

    for (int i = 0; i < myColors.size(); ++i) {
        // set Scalar pair
        Scalar lower(myColors[i][0], myColors[i][2], myColors[i][4]);
        Scalar upper(myColors[i][1], myColors[i][3], myColors[i][5]);

        // create mask by using inRange
        inRange(imgHSV, lower, upper, mask);
        Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3));
        dilate(mask, imgDil, kernel);
        imshow(to_string(i) + "imgDIL", imgDil);
//        imshow(to_string(i) + "imgMask", mask);
        // now we can get contours from the mask
        auto myPoint = getContours(img, count);
        if (myPoint.x != 0) {
            newPoints.push_back({myPoint.x, myPoint.y, i});
        }

    }

//    print("Objec num: {}\n", count);
//    fflush(stdout);
}

Point getContours(Mat img, int& count) {
    std::vector<std::vector<Point>> contours;
    std::vector<Vec4i> hierarchy;
    Point myPoint(0, 0);
    // find contours
    findContours(imgDil, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
    // prox poly
    for (int i = 0; i < contours.size(); ++i) {
        int area = contourArea(contours[i]);
//        print("contour area {}\n", area);
//        fflush(stdout);

        std::vector<std::vector<Point>> conPoly(contours.size());
        std::vector<Rect> boundRect(contours.size());
        std::string objectType;
        // large poly can be processed
        if (area > 1000) {
            count++;
            float peri = arcLength(contours[i], true);
            // get approx poly
            approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true);
//            print("poly area {}\n", conPoly[i].size());
//            fflush(stdout);

            // get bounding box
            boundRect[i] = boundingRect(conPoly[i]);
            // setting the drawing point
            myPoint.x = boundRect[i].x + boundRect[i].width / 2;
            myPoint.y = boundRect[i].y;
            // draw poly
            drawContours(img, conPoly, i, Scalar(255, 0, 255), 2);
            // draw bounding box on int
            rectangle(img, boundRect[i].tl(), boundRect[i].br(), Scalar(0, 255, 0));
        }
    }
    return myPoint;

}

```

## Project 2 Document scanner 

```cpp
#include <iostream>
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/objdetect.hpp"
#include "fmt/format.h"
#include <algorithm>
#include "boost/sort/sort.hpp"
#include "vector"

using namespace cv;
using namespace std;
using std::vector;
using std::optional;
using fmt::print;

///////// Project 2 ////////////

static Mat img, imgGray, imgBlur, imgCanny, imgDil;
static Mat imgThre, imgWarp, imgCrop;
static Mat imgOriginal;

static float w = 420, h = 596; // A4 paper width and height

Mat preProcessing(Mat img);

optional<vector<Point>> getContours();

void getWarp(Mat img, vector<Point> docPoints, float w, float h);

void docscan();

int main() {
    ///////////// static image ////////////////////
    string_view path = "Resources/paper.jpg";
    img = imread(path.data());

    // resize the image
    resize(img, img, Size(), 0.5, 0.5);

    imgOriginal = img.clone();
    docscan();
    waitKey(0);
////////////////////////////////////////////////////

    ////////////// webcam /////////////////////
//    VideoCapture cap(0, CAP_V4L2);
/*    VideoCapture cap(0);
    cap.set(CAP_PROP_FOURCC, VideoWriter::fourcc('M', 'J', 'P', 'G'));
    while (true) {
        cap.read(img);
        imgOriginal = img.clone();
        docscan();
        waitKey(1);
    }*/
    //////////////////////////////////////////
    return 0;
}

void docscan() {// Preprocessing
// you will get the edges of the objects in image
    imgThre = preProcessing(img);
    // Get Contours - biggest
    optional<vector<Point>> cornerPoints;
    cornerPoints = getContours();

    if (cornerPoints) {
        // draw cornerPoints

        for (int i = 0; i < cornerPoints->size(); ++i) {
            circle(img, (*cornerPoints)[i], 15, Scalar(0, 0, 255), FILLED);

        }

/*
        vector<int> sortorderadd;
        vector<int> sortordersub;
        for (int i = 0; i < cornerPoints->size(); ++i) {
            sortorderadd.push_back((*cornerPoints)[i].x + (*cornerPoints)[i].y);
            sortordersub.push_back((*cornerPoints)[i].x - (*cornerPoints)[i].y);
            print("sum x y is {}\n", (*cornerPoints)[i].x + (*cornerPoints)[i].y);
            print("sub x y is {}\n", (*cornerPoints)[i].x - (*cornerPoints)[i].y);
            fflush(stdout);
        }


        vector<int> add_idex;
        vector<int> sub_idex;

        for (int i = 0; i < 4; ++i) {
            add_idex.push_back(i);
            sub_idex.push_back(i);
        }

        std::sort(add_idex.begin(), add_idex.end(),
                  [&sortorderadd](int a, int b) {
                      return sortorderadd[a] > sortorderadd[b];
                  });

        std::sort(sub_idex.begin(), sub_idex.end(),
                  [&sortordersub](int a, int b) {
                      return sortordersub[a] > sortordersub[b];
                  });
        print("add index sort");
        for (int & i : add_idex) {
            print("{}\t", i);
        }
        print("{}\n", "");
        print("sub index sort");
        for (int & i : sub_idex) {
            print("{}\t", i);
        }
        print("{}\n", "");
        int zero = add_idex[3];
        int one = sub_idex[0];
        int two = sub_idex[3];
        int three = add_idex[0];
        putText(img, to_string(0), (*cornerPoints)[zero], FONT_HERSHEY_PLAIN, 5, Scalar(255, 255, 255), 4);
        putText(img, to_string(1), (*cornerPoints)[one], FONT_HERSHEY_PLAIN, 5, Scalar(255, 255, 255), 4);
        putText(img, to_string(2), (*cornerPoints)[two], FONT_HERSHEY_PLAIN, 5, Scalar(255, 255, 255), 4);
        putText(img, to_string(3), (*cornerPoints)[three], FONT_HERSHEY_PLAIN, 5, Scalar(255, 255, 255), 4);
*/
        // a better way of getting the max and min element index
        ///////////////////// Tutor version ////////////////////

        vector<Point> docPoints;
        vector<int> sumPoints, subPoints;
        for (int i = 0; i < 4; ++i) {
            sumPoints.push_back((*cornerPoints)[i].x + (*cornerPoints)[i].y);
            subPoints.push_back((*cornerPoints)[i].x - (*cornerPoints)[i].y);
        }
        // min/max_element method, return address of the max/min element, thus we can continue to subtract array starting address to get index number
        docPoints.push_back((*cornerPoints)[min_element(sumPoints.begin(), sumPoints.end()) - sumPoints.begin()]);
        docPoints.push_back((*cornerPoints)[min_element(subPoints.begin(), subPoints.end()) - subPoints.begin()]);
        docPoints.push_back((*cornerPoints)[max_element(sumPoints.begin(), sumPoints.end()) - sumPoints.begin()]);
        docPoints.push_back((*cornerPoints)[max_element(subPoints.begin(), subPoints.end()) - subPoints.begin()]);
        for (int i = 0; i < 4; ++i) {
            putText(img, to_string(i), docPoints[i] , FONT_HERSHEY_PLAIN, 5, Scalar(255, 255, 255), 4);
        }
        // Warp
        getWarp(img, docPoints, w, h);

        //Crop
        Rect roi(5, 5, w - (2 * 5), h - (2 * 5));
        imgCrop = imgWarp(roi);
        imshow("Doc", imgCrop);
    }
    imshow("prepro", imgThre);
    imshow("image", img);
}

void getWarp(Mat img, vector<Point> docPoints, float w, float h) {
    Point2f src[4] = {docPoints[0], docPoints[1], docPoints[2], docPoints[3]};
    Point2f dst[4] = {{0.0f, 0.0f},
                      {0.0f,    h},
                      {w, h},
                      {w,    0.0f}};
    Mat transformMatrix = getPerspectiveTransform(src, dst);
    warpPerspective(imgOriginal, imgWarp, transformMatrix, Point(w, h));
}

// use optional since you may not find the document in a image
optional<vector<Point>> getContours() {
    vector<vector<Point>> contours;
    vector<Vec4i> hierarchy;
    int counter = 0;
    // find contour
    findContours(imgThre, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
    vector<vector<Point>> conPoly(contours.size());
    int outputPolyIndex = 0;
    int maxArea = 0;
    for (int i = 0; i < contours.size(); ++i) {
        // somehow we only have contourArea method to calculate area
        int area = contourArea(contours[i]);
//        print("Area is {}\n", area);
        if (area > 1000) {
            if (area > maxArea) {
                auto peri = arcLength(contours[i], true);
                approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true);
                // only leave the rectangle Poly
                if (conPoly[i].size() == 4) {
                    maxArea = area;
                    outputPolyIndex = i;
                    drawContours(img, conPoly, i, Scalar(255, 0, 255), 2);
                    counter++;
                }
            }
        }
    }
    print("poly number is {}\n", counter);
    fflush(stdout);
    if (counter > 0) {
        return {conPoly[outputPolyIndex]};
    } else {
        return std::nullopt;
    }
}

Mat preProcessing(Mat img) {
    cvtColor(img, imgGray, COLOR_BGR2GRAY);
    //Gaussian allows you to remove some noise edges
    GaussianBlur(imgGray, imgBlur, Size(3, 3), 3, 0);
    Canny(imgBlur, imgCanny, 25, 75);
    Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3));
    dilate(imgCanny, imgDil, kernel);

    return imgDil;
}

```

