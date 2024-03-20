In this document, I would introduce how to use Skia based on my understanding. Firstly, I would necessary modules/classes in Skia. Then, I would present the fundamental workflow of Skia, other usages for some specific scenarios.

# Modules/Classes

## SkPath

Class SkPath is used to describe contour, which is constructed by lines and curves. These lines and curves are constituted by functions related to "Verbs" and "Points". For example,

```cpp
    SkPath path;
    path.moveTo(124, 108);          //add beginning of contour at SkPoint (124, 108)
    path.lineTo(172, 24);           //add line from last point (124, 108) to SkPoint (172, 24)
    path.addCircle(50, 50, 30);     //add circle centered at SkPoint (50, 50) with radius 30
    path.moveTo(36, 148);
    path.quadTo(66, 188, 120, 136); //add quadratic Bézier curve from last point (36, 148) towards (66, 120), to (188, 136).
```

## SkCanvas
