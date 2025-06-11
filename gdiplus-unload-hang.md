

### 🔍 Background

This issue occurs inside an **Eclipse add-in**, written in Java, which invokes c# code through native interop. The C# code further uses the BCG library—a C++ GUI toolkit that internally interacts with GDI for image rendering.

When closing a specific dialog in the add-in, the entire process hanges unexpectedly.

---

### 🧱 Problem

Upon closing the dialog, the system attempts to **unload GDI+** inside the BCG module. However, GDI+ was **never explicitly loaded** beforehand. As a result, its internal shutdown logic fails, leading to a **thread hang** in `GdiplusShutdown`.

> 💥 In short, GDI+’s unload handler is triggered without a matching load, causing the process to hang.

---

### 🔗 Reference

A similar issue was documented here:
[http://sali98.blogspot.com/2012/08/dont-call-gdishutdown-when-unloading.html](http://sali98.blogspot.com/2012/08/dont-call-gdishutdown-when-unloading.html)

---

### 📟 Call Stack Trace 
```
0530fa00 6a8979ec gdiplus!BackgroundThreadShutdown+0x4c, calling KERNEL32!WaitForSingleObject
0530fa20 6a8a9c52 gdiplus!InternalGdiplusShutdown+0x12, calling gdiplus!BackgroundThreadShutdown
0530fa28 6a87573c gdiplus!GdiplusShutdown+0x2c, calling gdiplus!InternalGdiplusShutdown
0530fa34 55f59a5e BCGCBPRO2430140!CBCGPPngImage::~CBCGPPngImage+0xde, calling gdiplus!GdiplusShutdown


