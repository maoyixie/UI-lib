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

Class SkCanvas provides an interface for drawing, which is the drawing context for Skia. However, it does not store any other drawing attributes in the context (e.g. color, pen size). Rather, these are specified explicitly in each draw call, via a SkPaint. There are multiple <a name="backends"></a> to create SkCanvas.

## SkPaint

Class SkPaint is used to specify attributes in a paint, like color, blend, style, font, etc. For example,

```cpp
    SkPaint paint;
    paint.setStyle(SkPaint::kStroke_Style);
    paint.setStrokeWidth(3.0f);
    paint.setAntiAlias(true);
    paint.setColor(SkColorSetARGB(0xFF, 0xFF, 0x00, 0x00));
    paint.setColor(SkColorSetARGB(0xFF, 0x00, 0x00, 0xFF));
```

SkPaint also supports effects, which are subclasses of different aspects of different aspects of the drawing pipeline. For example, to draw using a gradient instead of a single color, assign a SkShader to the paint.

```cpp
    void draw(SkCanvas* canvas) {
    SkPoint points[2] = {SkPoint::Make(0.0f, 0.0f), SkPoint::Make(256.0f, 256.0f)};
    SkColor colors[2] = {SK_ColorBLUE, SK_ColorYELLOW};
    SkPaint paint;
    paint.setShader(SkGradientShader::MakeLinear(points, colors, nullptr, 2, SkTileMode::kClamp, 0, nullptr));
    canvas->drawPaint(paint);
}
```

There are 6 types of effects that can be assigned to a paint: <a name="SkPathEffect"></a>, <a name="SkRasterizer"></a>, <a name="SkMaskFilter"></a>, <a name="SkShader"></a>, <a name="SkColorFilter"></a> and <a name="SkBlendMode"></a>. Besides, SkPaint supports to draw and measure text.

### [SkPathEffect](#SkPathEffect)

TODO

### [SkRasterizer](#SkRasterizer)

TODO

### [SkMaskFilter](#SkMaskFilter)

TODO

### [SkShader](#SkShader)

TODO

### [SkColorFilter](#SkColorFilter)

TODO

### [SkBlendMode](#SkBlendMode)

TODO

# [Backends](#backends)

## Raster

TODO

## GPU

TODO

## SkPDF

TODO

## SkPicture

TODO

## NullCanvas

TODO

## SkXPS

TODO

## SkSVG

TODO
