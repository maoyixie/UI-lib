In this document, targeting cpp APIs, I would introduce how to use Skia based on my understanding. Firstly, I would introduce necessary modules/classes in Skia. Then, I would present the fundamental workflow of Skia, other usages for some specific scenarios.

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

Class SkCanvas provides an interface for drawing, which is the drawing context for Skia. However, it does not store any other drawing attributes in the context (e.g. color, pen size). Rather, these are specified explicitly in each draw call, via a SkPaint. There are multiple backends<a name="backends"></a> to create SkCanvas.

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

There are 6 types of effects that can be assigned to a paint: [SkPathEffect](#SkPathEffect), [SkRasterizer](#SkRasterizer), <a name="SkMaskFilter"></a>, <a name="SkShader"></a>, <a name="SkColorFilter"></a> and <a name="SkBlendMode"></a>. Besides, SkPaint supports to draw and measure text.

### SkPathEffect

Modifications to the geometry (path) before it generates an alpha mask (e.g. dashing). TODO...

### SkRasterizer

Composing custom mask layers (e.g. shadows). TODO...

### [SkMaskFilter](#SkMaskFilter)

Modifications to the alpha mask before it is colorized and drawn (e.g. blur). TODO...

### [SkShader](#SkShader)

Gradients (linear, radial, sweep), bitmap patterns (clamp, repeat, mirror), etc. TODO...

### [SkColorFilter](#SkColorFilter)

Modify the source color(s) before applying the blend (e.g. color matrix). TODO...

### [SkBlendMode](#SkBlendMode)

Porter-duff transfermodes, blend modes. TODO...

## [Backends](#backends)

Skia has multiple backends which receive SkCanvas drawing commands. Each backend has a unique way of creating a SkCanvas.

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

TODO...

## Runtime Effects

TODO...

# Workflow

The basic/fundamental workflow is that:

```
initialize canvas => set parameters => draw
```

- Initialize canvas: You can call canvas->save(), canvas->clear(SkColor) or canvas->drawColor(Sk_ColorWHITE) to initialize the canvas. Do remember to call canvas->restore() at the end if you call canvas->save() at the beginning.
- Set parameters: You can only call SkPaint without regarding to SkPath, etc.
- Draw: canvas->drawXXX().

Next, I would summarize some combination methods based on my observations.

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
