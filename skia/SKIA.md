In this document, targeting cpp APIs, I would introduce how to use Skia based on my understanding. Firstly, I would present the fundamental [workflow](#workflow) of Skia. Then, I would deeply illustrate the necessary [module/class](#moduleclass) in Skia. Finally, I would summarize the drawing [model](#model) from the perspective of APIs interaction.

# WORKFLOW

The basic/fundamental workflow is that:

```
(initialize canvas) => set parameters => draw
```

- Initialize canvas: You can call canvas->save(), canvas->clear(SkColor) or canvas->drawColor(Sk_ColorWHITE) to initialize the canvas. Do remember to call canvas->restore() at the end if you call canvas->save() at the beginning. You can also do nothing.
- Set parameters:
  - Define graphic: You can use SkPath or SkXXX (e.g. SkRect).
  - Configure other parameters: You can use SkPaint. There are a lot of other settings.
- Draw: You can call canvas->drawPath() or canvas->drawXXX().

Here are two simple examples:

```cpp
void draw(SkCanvas* canvas) {
    canvas->drawColor(SK_ColorWHITE);
    SkPaint paint;
    paint.setAntiAlias(true);
    SkPath path;
    path.moveTo(124, 108);
    path.lineTo(172, 24);
    canvas->drawPath(path, paint);
}
```

```cpp
void draw(SkCanvas* canvas) {
    canvas->save();
    SkRect rect = SkRect::MakeXYWH(-90.5f, -90.5f, 181.0f, 181.0f);
    SkPaint paint;
    paint.setColor(SK_ColorBLUE);
    canvas->drawRect(rect, paint);
    canvas->restore();
}
```

# MODULE/CLASS

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

Class SkCanvas provides an interface for drawing, which is the drawing context for Skia. However, it does not store any other drawing attributes in the context (e.g. color, pen size). Rather, these are specified explicitly in each draw call, via a SkPaint. There are multiple [backends](#Backends) to create SkCanvas.

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

There are 6 types of effects that can be assigned to a paint: [SkPathEffect](#skpatheffect), [SkRasterizer](#skrasterizer), [SkMaskFilter](#skmaskfilter), [SkShader](#skshader), [SkColorFilter](#skcolorfilter) and [SkBlendMode](#skblendmode). Besides, SkPaint supports to draw and measure text. The following are detailed illustrations to the 6 types of effects.

### SkPathEffect

Class SkPathEffect modifies/affects the geometry (path) of a drawing primitive before it is transformed by the canvas' matrix and drawn. Here is an example:

```cpp
SkPath star() {
    const SkScalar R = 115.2f, C = 128.0f;
    SkPath path;
    path.moveTo(C + R, C);
    for (int i = 1; i < 8; ++i) {
        SkScalar a = 2.6927937f * i;
        path.lineTo(C + R * cos(a), C + R * sin(a));
    }
    return path;
}

void draw(SkCanvas* canvas) {
    SkPaint paint;
    paint.setPathEffect(SkPathEffect::MakeSum(SkDiscretePathEffect::Make(10.0f, 4.0f), SkDiscretePathEffect::Make(10.0f, 4.0f, 1245u)));
    paint.setStyle(SkPaint::kStroke_Style);
    paint.setStrokeWidth(2.0f);
    paint.setAntiAlias(true);
    canvas->clear(SK_ColorWHITE);
    SkPath path(star());
    canvas->drawPath(path, paint);
}
```

### SkRasterizer

Composing custom mask layers (e.g. shadows). Seems to related to GPU, little information. TODO...

### SkMaskFilter

Modifications to the alpha mask before it is colorized and drawn (e.g. blur). Here is an example:

```cpp
void draw(SkCanvas* canvas) {
    canvas->drawColor(SkColorSetARGB(0xFF, 0xFF, 0xFF, 0xFF));
    SkPaint paint;
    paint.setMaskFilter(SkMaskFilter::MakeBlur(kNormal_SkBlurStyle, 5.0f));
    sk_sp<SkTextBlob> blob = SkTextBlob::MakeFromString("Skia", SkFont(nullptr, 120));
    canvas->drawTextBlob(blob.get(), 0, 160, paint);
}
```

### SkShader

Gradients (linear, radial, sweep), bitmap patterns (clamp, repeat, mirror). Shaders specify the source color(s) for what is being drawn. If a paint has no shader, then the paint's color is used. If the paint has a shader, then the shader's color(s) are use instead. Here is an example:

```cpp
// skpaint_compose_shader
void draw(SkCanvas* canvas) {
    SkColor colors[2] = {SK_ColorBLUE, SK_ColorYELLOW};
    SkPaint paint;
    paint.setShader(SkShaders::Blend(
            SkBlendMode::kDifference,
            SkGradientShader::MakeRadial(SkPoint::Make(128.0f, 128.0f), 180.0f, colors, nullptr, 2, SkTileMode::kClamp, 0, nullptr),
            SkPerlinNoiseShader::MakeTurbulence(0.025f, 0.025f, 2, 0.0f, nullptr)));
    canvas->drawPaint(paint);
}
```

### SkColorFilter

Modify the source color(s) before applying the blend (e.g. color matrix). ColorFilters are optional objects in the drawing pipeline. When present in a paint, they are called with the "src" colors, and return new colors, which are then passed onto the next stage (either ImageFilter or Xfermode). Here is an API in class SkColorFilter:

```cpp
// construct a colorfilter whose effect is to first apply the inner filter and then apply this filter, applied to the output of the inner filter.
// result = this(inner(...))
sk_sp<SkColorFilter> makeComposed(sk_sp<SkColorFilter> inner) const;
```

Some other APIs in class SkColorFilters:

```cpp
static sk_sp<SkColorFilter> Compose(const sk_sp<SkColorFilter>& outer, sk_sp<SkColorFilter> inner) {
        return outer ? outer->makeComposed(std::move(inner)) : std::move(inner);
}

// Blends between the constant color (src) and input color (dst) based on the SkBlendMode.
// If the color space is null, the constant color is assumed to be defined in sRGB.
static sk_sp<SkColorFilter> Blend(const SkColor4f& c, sk_sp<SkColorSpace>, SkBlendMode mode);
```

### SkBlendMode

As for enum class SkBlendMode, blends are operators that take in two colors (source, destination) and return a new color. For enum class SkBlendModeCoeff, aka Porter-Duff SkBlendModes, these coefficients describe the blend equation used. The following example demonstrates all of the Skia’s standard blend modes. In this example the source is a solid magenta color with a horizontal alpha gradient and the destination is a solid cyan color with a vertical alpha gradient.

```cpp
void draw(SkCanvas* canvas) {
    SkBlendMode modes[] = {
        SkBlendMode::kClear,
        SkBlendMode::kSrc,
        SkBlendMode::kDst,
        SkBlendMode::kSrcOver,
        SkBlendMode::kDstOver,
        SkBlendMode::kSrcIn,
        SkBlendMode::kDstIn,
        SkBlendMode::kSrcOut,
        SkBlendMode::kDstOut,
        SkBlendMode::kSrcATop,
        SkBlendMode::kDstATop,
        SkBlendMode::kXor,
        SkBlendMode::kPlus,
        SkBlendMode::kModulate,
        SkBlendMode::kScreen,
        SkBlendMode::kOverlay,
        SkBlendMode::kDarken,
        SkBlendMode::kLighten,
        SkBlendMode::kColorDodge,
        SkBlendMode::kColorBurn,
        SkBlendMode::kHardLight,
        SkBlendMode::kSoftLight,
        SkBlendMode::kDifference,
        SkBlendMode::kExclusion,
        SkBlendMode::kMultiply,
        SkBlendMode::kHue,
        SkBlendMode::kSaturation,
        SkBlendMode::kColor,
        SkBlendMode::kLuminosity,
    };
    SkRect rect = SkRect::MakeWH(64.0f, 64.0f);
    SkPaint stroke, src, dst;
    stroke.setStyle(SkPaint::kStroke_Style);
    SkPoint srcPoints[2] = {SkPoint::Make(0.0f, 0.0f), SkPoint::Make(64.0f, 0.0f)};
    SkColor srcColors[2] = {SK_ColorMAGENTA & 0x00FFFFFF, SK_ColorMAGENTA};
    src.setShader(SkGradientShader::MakeLinear(srcPoints, srcColors, nullptr, 2, SkTileMode::kClamp, 0, nullptr));

    SkPoint dstPoints[2] = {SkPoint::Make(0.0f, 0.0f), SkPoint::Make(0.0f, 64.0f)};
    SkColor dstColors[2] = {SK_ColorCYAN & 0x00FFFFFF, SK_ColorCYAN};
    dst.setShader(SkGradientShader::MakeLinear(dstPoints, dstColors, nullptr, 2, SkTileMode::kClamp, 0, nullptr));
    canvas->clear(SK_ColorWHITE);
    size_t N = sizeof(modes) / sizeof(modes[0]);
    size_t K = (N - 1) / 3 + 1;
    SkASSERT(K * 64 == 640);  // tall enough
    for (size_t i = 0; i < N; ++i) {
        SkAutoCanvasRestore autoCanvasRestore(canvas, true);
        canvas->translate(192.0f * (i / K), 64.0f * (i % K));
        canvas->clipRect(SkRect::MakeWH(64.0f, 64.0f));
        canvas->drawColor(SK_ColorLTGRAY);
        (void)canvas->saveLayer(nullptr, nullptr);
        canvas->clear(SK_ColorTRANSPARENT);
        canvas->drawPaint(dst);
        src.setBlendMode(modes[i]);
        canvas->drawPaint(src);
        canvas->drawRect(rect, stroke);
    }
}
```

## Backends

Skia has multiple backends which receive SkCanvas drawing commands. Each backend has a unique way of creating a SkCanvas. The following are comprehensive explanations of various backends.

### Raster

The raster backend draws to a block of memory. This memory can be managed by Skia or by the client. TODO...

### GPU

GPU Surfaces must have a GrContext object which manages the GPU context, and related caches for textures and fonts. TODO...

### SkPDF

The SkPDF backend uses SkDocument instead of SkSurface. So in this case, you draw in a document-based canvas. The general process of using SkPDF is as follows:

```
Create a document, specifying a stream to store the output.
For each "page" of content:
    canvas = doc->beginPage(...)
    draw_my_content(canvas);
    doc->endPage();
Close the document with doc->close().
```

Here is an concrete example of using Skia’s PDF backend (SkPDF) via the SkDocument and SkCanvas APIs.

```cpp
void WritePDF(SkWStream* outputStream, const char* documentTitle, void (*writePage)(SkCanvas*, int page), int numberOfPages, SkSize pageSize) {
    SkPDF::Metadata metadata;
    metadata.fTitle = documentTitle;
    metadata.fCreator = "Example WritePDF() Function";
    metadata.fCreation = {0, 2019, 1, 4, 31, 12, 34, 56};
    metadata.fModified = {0, 2019, 1, 4, 31, 12, 34, 56};
    auto pdfDocument = SkPDF::MakeDocument(outputStream, metadata);
    SkASSERT(pdfDocument);
    for (int page = 0; page < numberOfPages; ++page) {
        SkCanvas* pageCanvas = pdfDocument->beginPage(pageSize.width(), pageSize.height());
        writePage(pageCanvas, page);
        pdfDocument->endPage();
    }
    pdfDocument->close();
}

// Print binary data to stdout as hex.
void print_data(const SkData* data, const char* name) {
    if (data) {
        SkDebugf("\nxxd -r -p > %s << EOF", name);
        size_t s = data->size();
        const uint8_t* d = data->bytes();
        for (size_t i = 0; i < s; ++i) {
            if (i % 40 == 0) { SkDebugf("\n"); }
            SkDebugf("%02x", d[i]);
        }
        SkDebugf("\nEOF\n\n");
    }
}

// example function that draws on a SkCanvas.
void write_page(SkCanvas* canvas, int) {
    const SkScalar R = 115.2f, C = 128.0f;
    SkPath path;
    path.moveTo(C + R, C);
    for (int i = 1; i < 8; ++i) {
        SkScalar a = 2.6927937f * i;
        path.lineTo(C + R * cos(a), C + R * sin(a));
    }
    SkPaint paint;
    paint.setStyle(SkPaint::kStroke_Style);
    canvas->drawPath(path, paint);
}

void draw(SkCanvas*) {
    constexpr SkSize ansiLetterSize{8.5f * 72, 11.0f * 72};
    SkDynamicMemoryWStream buffer;
    WritePDF(&buffer, "SkPDF Example", &write_page, 1, ansiLetterSize);
    sk_sp<SkData> pdfData = buffer.detachAsData();
    print_data(pdfData.get(), "skpdf_example.pdf");
}
```

### SkPicture

The SkPicture backend uses SkPictureRecorder instead of SkSurface. TODO...

### NullCanvas

The NullCanvas is a canvas that ignores all drawing commands and does nothing. TODO...

### SkXPS

The SkXPS(still experimental) canvas writes into an XPS document. TODO...

### SkSVG

The SkSVG(still experimental) canvas writes into an SVG document. TODO...

## SkSL

Shading language. TODO...

## Runtime Effects

TODO...

# MODEL

From the standpoint of APIs interaction, I provide a recap of the drawing models.

## Stacking

Stacking means that in a canvas, after drawing one graphic, the next graphic is drawn, and so on, with no relationship between the graphics. The following is a concrete example:

```cpp
void draw(SkCanvas* canvas) {
    canvas->drawColor(SK_ColorWHITE);

    SkPaint paint;
    paint.setStyle(SkPaint::kStroke_Style);
    paint.setStrokeWidth(4);
    paint.setColor(SK_ColorRED);

    SkRect rect = SkRect::MakeXYWH(50, 50, 40, 60);
    canvas->drawRect(rect, paint);

    SkRRect oval;
    oval.setOval(rect);
    oval.offset(40, 60);
    paint.setColor(SK_ColorBLUE);
    canvas->drawRRect(oval, paint);

    paint.setColor(SK_ColorCYAN);
    canvas->drawCircle(180, 50, 25, paint);

    rect.offset(80, 0);
    paint.setColor(SK_ColorYELLOW);
    canvas->drawRoundRect(rect, 10, 10, paint);

    SkPath path;
    path.cubicTo(768, 0, -512, 256, 256, 256);
    paint.setColor(SK_ColorGREEN);
    canvas->drawPath(path, paint);

    canvas->drawImage(image, 128, 128, SkSamplingOptions(), &paint);

    SkRect rect2 = SkRect::MakeXYWH(0, 0, 40, 60);
    canvas->drawImageRect(image, rect2, SkSamplingOptions(), &paint);

    SkPaint paint2;
    auto text = SkTextBlob::MakeFromString("Hello, Skia!", SkFont(nullptr, 18));
    canvas->drawTextBlob(text.get(), 50, 25, paint2);
}
```
