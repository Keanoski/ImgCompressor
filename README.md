# In-Browser Image Compressor

## Table of Contents

- [Key Features](#key-features)
- [How to Use](#how-to-use)
- [Dependencies](#dependencies)
- [File Structure](#file-structure)
- [Technical Breakdown](#technical-breakdown)
  - [UI Components](#ui-components)
  - [Core Logic (JavaScript)](#core-logic-javascript)
  - [The Compression Process](#the-compression-process)
  - [Batch Downloading with JSZip](#batch-downloading-with-jszip)

## Key Features

- ✅ **100% Client-Side:** All image processing happens in your browser. Files are never uploaded to a server, ensuring privacy.
- ⚙️ **Adjustable Compression:** Fine-tune image optimization with controls for both JPEG quality and image dimensions (scale).
- 🖼️ **Interactive Previews:** See thumbnails of your images before compression, complete with file size and dimension info.
- ✨ **Drag & Drop:** Easily add images by dragging them anywhere onto the page.
- 🗂️ **Batch Processing:** Compress multiple images at once and download them as a single `.zip` file for convenience.
- 📱 **Responsive Design:** A clean, modern interface that works on both desktop and mobile devices.

## How to Use

1.  **Open [Image Compressor](https://keanoski.github.io/img-compression-and-scaling-webapp/)** in any modern web browser.
2.  **Add Images:**
    -   Click the **"Select Images"** button to open a file dialog.
    -   Or, **drag and drop** your image files directly onto the page.
3.  **Adjust Settings:**
    -   Use the **Image Quality** and **Image Scale** toggle switches to enable/disable those options.
    -   Move the sliders or type in the input boxes to set your desired compression levels. The changes will be reflected in the preview dimensions.
4.  **Select Images:** By default, all added images are selected. You can click on any preview thumbnail to de-select it from the compression batch.
5.  **Compress:** Click the **"Compress X Image(s)"** button.
6.  **Download:**
    -   Download individual images from the "Results" section.
    -   If you compressed more than one image, a **"Download All (.zip)"** button will appear for a batch download.

## Dependencies

The application relies on two external libraries delivered via CDN.

| Library        | Version | Purpose                                        |
| :------------- | :------ | :--------------------------------------------- |
| **Tailwind CSS** | v3      | For utility-first CSS styling.                 |
| **JSZip** | v3.10.1 | To create `.zip` archives for batch downloads. |

They are included in the `<head>` section as follows:

```html
<!-- Tailwind CSS for styling -->
<script src="[https://cdn.tailwindcss.com](https://cdn.tailwindcss.com)"></script>

<!-- JSZip library for zipping files -->
<script src="[https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js](https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js)"></script>
````

## Technical Breakdown

### UI Components

The interface is built with semantic HTML and styled with Tailwind CSS. Key components are identified by IDs for manipulation with JavaScript:

  - `#mainCard`: The central container for all controls.
  - `#qualityControlContainer` & `#scaleControlContainer`: Hold the sliders, inputs, and toggles for compression settings.
  - `#fileInput`: The standard file input button.
  - `#dropOverlay`: A full-screen overlay that appears during a drag-and-drop operation.
  - `#previewContainer`: The grid where thumbnails of selected images are displayed.
  - `#resultsSection`: The container for the final compressed images and download links.

### Core Logic (JavaScript)

All application logic resides in a `<script>` tag at the end of the `<body>`.

  - **DOM Ready:** The script begins by getting references to all necessary DOM elements.
  - **File Handling (`handleFileSelection`):** This function is triggered by both the file input and the drop event. It filters for valid image files, creates a unique object for each, and adds it to the global `fileList` array before generating a UI preview card.
  - **State Management:** The state of the application (which files are selected, what the compression settings are) is managed through the `fileList` array and the values of the input elements. The UI, particularly the main "Compress" button, is updated reactively with the `updateCompressButtonState` function.

### The Compression Process

The core compression logic is handled by the `compressImage` function, which returns a `Promise`.

1.  **Image Loading:** An input `File` object is read into an `Image` element using a `FileReader` and a Data URL.
2.  **Canvas Rendering:** Once the image is loaded, a `<canvas>` element is created in memory. Its dimensions are set based on the user's "Image Scale" setting. The source image is then drawn onto this canvas using `ctx.drawImage()`.
3.  **Encoding:** The magic happens with the `canvas.toDataURL()` method. This function exports the content of the canvas as a new image.
      - `canvas.toDataURL('image/jpeg', quality)`: We specify `'image/jpeg'` as the output format to perform lossy compression. The `quality` parameter (a value between 0.0 and 1.0) dictates the level of compression.

<!-- end list -->

```javascript
function compressImage(file, quality, scale) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = (event) => {
            const img = new Image();
            img.src = event.target.result;
            img.onload = () => {
                const canvas = document.createElement('canvas');
                const scaleFactor = scale / 100;
                canvas.width = Math.round(img.width * scaleFactor);
                canvas.height = Math.round(img.height * scaleFactor);
                const ctx = canvas.getContext('2d');
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                
                // The core compression step
                const compressedDataUrl = canvas.toDataURL('image/jpeg', quality);
                const compressedSize = compressedDataUrl.length * (3/4) - 2; // Estimate byte size from Base64
                
                resolve({ dataUrl: compressedDataUrl, size: compressedSize });
            };
            img.onerror = reject;
        };
        reader.onerror = reject;
    });
}
```

### Batch Downloading with JSZip

When the "Download All" button is clicked, the `createAndDownloadZip` function is executed.

1.  **Initialize JSZip:** A new `JSZip` instance is created.
2.  **Add Files:** The function iterates through the successfully compressed files. The Base64 data for each image is extracted from its Data URL and added to the zip archive via `zip.file()`.
3.  **Generate Zip:** `zip.generateAsync({ type: "blob" })` creates the complete `.zip` file as a `Blob` in memory.
4.  **Trigger Download:** A temporary `<a>` element is created, its `href` is set to an object URL pointing to the `Blob`, and its `download` attribute is set. A click is programmatically triggered on the link to start the download, after which the temporary elements are cleaned up.

<!-- end list -->
