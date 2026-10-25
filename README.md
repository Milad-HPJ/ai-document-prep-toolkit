# AI Document Prep Toolkit

**A practical, cross-platform guide to extracting text from PDFs and images before using AI tools.**

This repository documents simple workflows for turning text-based PDFs, scanned documents, and images into searchable, reviewable text. It focuses on tools that can run locally, including **Poppler (`pdftotext` and `pdftoppm`)**, **Tesseract OCR**, and the optional **OCRmyPDF** utility.

> **Important scope note:** This project does not bypass an AI service's server-side upload limits, access controls, or terms of service. It helps prepare content locally—such as extracting text from a PDF or OCR-processing a scan—so that relevant information can be shared in a supported format where permitted.

## Table of contents

- [The problem](#the-problem)
- [Tools at a glance](#tools-at-a-glance)
- [Choose the right workflow](#choose-the-right-workflow)
- [Installation](#installation)
  - [Ubuntu and Debian](#ubuntu-and-debian)
  - [Fedora and RHEL-based distributions](#fedora-and-rhel-based-distributions)
  - [Arch Linux](#arch-linux)
  - [Windows](#windows)
  - [macOS](#macos)
- [Common workflows](#common-workflows)
  - [Extract text from a text-based PDF](#1-extract-text-from-a-text-based-pdf)
  - [Run OCR on an image](#2-run-ocr-on-an-image)
  - [Recognize Persian and English](#3-recognize-persian-and-english)
  - [Make a scanned PDF searchable on Linux](#4-make-a-scanned-pdf-searchable-on-linux)
  - [OCR a scanned PDF on Windows](#5-ocr-a-scanned-pdf-on-windows)
  - [Extract only selected pages](#6-extract-only-selected-pages)
- [Improve OCR results](#improve-ocr-results)
- [Privacy and responsible use](#privacy-and-responsible-use)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## The problem

AI platforms commonly apply limits or restrictions to file size, attachment count, supported formats, image inputs, or the amount of content that can be processed in a request. The exact rules depend on the provider, account, product, and model, and can change over time.

There is also a document-format problem: a PDF may contain selectable text, or it may be only a collection of scanned page images. A conventional text extractor can read the first type, but it cannot reliably recover text from image-only pages without OCR. Images may also contain useful text that is difficult to quote, search, or organize manually.

A practical workflow is to extract the information locally, verify the result, remove irrelevant pages, and then share only the content needed for the task—in a format accepted by the target service. This can reduce unnecessary file handling and make long documents easier to review. **It does not guarantee that a service will accept an upload or remove any provider-enforced limit.**

## Tools at a glance

| Tool | Main purpose | Best use case |
|---|---|---|
| **`pdftotext`** (Poppler) | Extracts an existing text layer from a PDF | PDFs in which text can already be selected or copied |
| **Tesseract OCR** | Recognizes text in images | Screenshots, scans, photographed pages, and image files |
| **`pdftoppm`** (Poppler) | Converts PDF pages to images | Preparing scanned PDF pages for OCR |
| **OCRmyPDF** (optional) | Adds an OCR text layer to scanned PDFs | Making an image-only PDF searchable before extracting its text |

Prefer `pdftotext` when the PDF already contains text. OCR is slower and can introduce recognition errors, so it should not be the default for every PDF.

## Choose the right workflow

1. **Text-based PDF:** use `pdftotext`.
2. **Image or screenshot:** use Tesseract.
3. **Scanned PDF:** use OCRmyPDF, or convert the pages with `pdftoppm` and process them with Tesseract.
4. **Long document:** extract only the relevant pages or sections, review the result, and preserve page numbers or headings when they matter.

## Installation

Commands below use the operating system's package manager where possible. Package names and available language data vary by distribution and release; consult the linked official documentation if a package is unavailable.

### Ubuntu and Debian

Install Poppler utilities, Tesseract, and English and Persian language data:

```bash
sudo apt update
sudo apt install -y poppler-utils tesseract-ocr tesseract-ocr-eng tesseract-ocr-fas
```

Optional: install OCRmyPDF for scanned PDFs:

```bash
sudo apt install -y ocrmypdf
```

If you do not need Persian OCR, omit `tesseract-ocr-fas`. Additional language packages are commonly named `tesseract-ocr-<language-code>`; check your distribution's package index.

### Fedora and RHEL-based distributions

```bash
sudo dnf install poppler-utils tesseract
```

Find the language packs available for your distribution, then install the ones you need. On Fedora, language packs commonly use names such as `tesseract-langpack-eng` and `tesseract-langpack-fas`:

```bash
dnf search tesseract-langpack
```

For OCRmyPDF, consult its current installation guide for your distribution before installing it.

### Arch Linux

```bash
sudo pacman -S poppler tesseract tesseract-data-eng tesseract-data-fas
```

If a language package name is not available, search the repository for the appropriate `tesseract-data-<code>` package.

### Windows

#### 1. Install Tesseract

Using Windows Package Manager (`winget`):

```powershell
winget install --id UB-Mannheim.TesseractOCR -e
```

Alternatively, use the Windows installation instructions linked in the Tesseract documentation. If the `tesseract` command is not found after installation, add the installation folder (commonly `C:\Program Files\Tesseract-OCR`) to the Windows `Path` environment variable, then restart the terminal.

Ensure the language data you need is installed. English is commonly available by default; for other languages, follow the installer or Tesseract language-data instructions.

#### 2. Install Poppler for Windows

Download a Windows build from the [poppler-windows releases page](https://github.com/oschwartz10612/poppler-windows/releases). This is a third-party packaging project for Poppler binaries, rather than the upstream Poppler project itself.

Extract the archive, locate the directory containing `pdftotext.exe` and `pdftoppm.exe` (often a `Library\bin` directory), and add that directory to `Path`. Open a **new** PowerShell window after changing `Path`.

Verify both installations using the commands below. The folder layout may vary between release packages, so use the actual path in the archive you downloaded.

#### 3. Verify the installation

```powershell
tesseract --version
tesseract --list-langs
pdftotext -v
pdftoppm -v
```

#### Optional: OCRmyPDF on Windows

OCRmyPDF has additional system dependencies and its native Windows setup is more involved than installing Tesseract and Poppler individually. For a Linux-style workflow on a Windows computer, **WSL 2 with Ubuntu** is often a straightforward option. Follow the current [OCRmyPDF installation guide](https://ocrmypdf.readthedocs.io/en/stable/installation.html) rather than relying on an old one-line installation recipe.

### macOS

With [Homebrew](https://brew.sh):

```bash
brew install tesseract poppler
```

For additional Tesseract languages, check the current Tesseract/Homebrew instructions. OCRmyPDF can also be installed with Homebrew; see its official documentation.

## Common workflows

Run commands from the directory containing the input file, or provide full paths to the files. Quote paths that contain spaces.

### 1. Extract text from a text-based PDF

```bash
pdftotext -layout "input.pdf" "output.txt"
```

`-layout` attempts to preserve the physical layout of text. It is useful for some multi-column documents and tables, but it does not guarantee accurate reconstruction of complex layouts.

To extract only pages 3 through 7:

```bash
pdftotext -f 3 -l 7 -layout "input.pdf" "selected-pages.txt"
```

On Windows PowerShell, the same `pdftotext` commands work after Poppler is on `Path`.

### 2. Run OCR on an image

Write recognized text to `output.txt`:

```bash
tesseract "scan.png" "output" -l eng
```

Tesseract uses the output name as a base and writes the result to `output.txt`.

To print recognized text to the terminal or redirect it to a file:

```bash
tesseract "scan.png" stdout -l eng > "output.txt"
```

The same syntax works in PowerShell if Tesseract is installed and available on `Path`.

### 3. Recognize Persian and English

Install both languages' trained data, then run:

```bash
tesseract "scan.png" "output" -l fas+eng
```

Tesseract's language codes and available trained data depend on which language packages are installed. Right-to-left text, mixed-language passages, tables, and complex page layouts may need manual correction. Always compare important results against the original document.

### 4. Make a scanned PDF searchable on Linux

On Debian or Ubuntu, install OCRmyPDF if it is not already installed:

```bash
sudo apt install -y ocrmypdf
```

Create a searchable PDF with an OCR text layer:

```bash
ocrmypdf --deskew --rotate-pages -l eng "scanned.pdf" "searchable.pdf"
```

For Persian and English, provided both language packs are installed:

```bash
ocrmypdf --deskew --rotate-pages -l fas+eng "scanned.pdf" "searchable.pdf"
```

Then extract the recognized text:

```bash
pdftotext -layout "searchable.pdf" "extracted.txt"
```

OCRmyPDF options and dependencies can change. Check `ocrmypdf --help` and the official documentation for advanced options, PDF/A requirements, and version-specific behavior. Keep the original PDF unchanged until the output has been checked.

### 5. OCR a scanned PDF on Windows

First, render each page to a PNG image. This example assumes the `pdftoppm` executable from Poppler is on `Path`:

```powershell
New-Item -ItemType Directory -Force .\pages | Out-Null
pdftoppm -r 300 -png .\scanned.pdf .\pages\page
```

Then use Tesseract to process the generated pages and combine the text in page-number order:

```powershell
$pages = Get-ChildItem .\pages\page-*.png |
    Sort-Object { [int]($_.BaseName -replace '^page-', '') }

$output = foreach ($page in $pages) {
    "===== $($page.Name) ====="
    & tesseract $page.FullName stdout -l eng
    ""
}

$output | Set-Content -Encoding utf8 .\extracted.txt
```

Use `-l fas+eng` instead of `-l eng` if both language packs are installed and the document contains Persian and English. If your PDF has a very large number of pages, process a selected page range first and inspect the output before processing the entire document.

### 6. Extract only selected pages

For a large PDF, extracting only the relevant pages can produce a more manageable text file and preserve focus:

```bash
pdftotext -f 10 -l 18 -layout "large-document.pdf" "relevant-section.txt"
```

This is content selection, not a way to remove an AI provider's upload restrictions. Review the selected range to ensure it includes enough context to interpret the passage correctly.

## Improve OCR results

- **Start with a clean source.** Skewed, blurry, low-resolution, compressed, or shadowed images generally reduce recognition quality.
- **Try higher resolution for scans.** For many printed documents, rendering at around 300 DPI is a reasonable starting point; test higher or lower settings based on source quality and file size.
- **Choose the correct language.** Install the necessary language data and set `-l`, for example `eng` or `fas+eng`.
- **Preserve structure where possible.** Keep page headings, page numbers, section names, and table boundaries when they are necessary to understand the extracted content.
- **Validate high-impact details.** Check names, dates, identifiers, figures, units, negations, code, and table values against the source.
- **Do not expect perfect layout reconstruction.** OCR can confuse similar characters, misread columns, and lose diagram relationships. Provide the original image or a relevant crop when visual information is essential and the target AI service supports it.

## Privacy and responsible use

- Local extraction with these command-line tools can be performed without uploading the source file to an AI service. However, **the text you later paste or upload to an external service may still be transmitted to that provider**.
- Redact passwords, API keys, access tokens, personal data, confidential business information, and other sensitive content before sharing it.
- Follow your organization's data-handling rules and the target platform's current terms, accepted formats, and upload limits.
- Preserve originals and treat OCR output as an imperfect transcription—not as verified source material.
- Do not use this workflow to bypass authentication, access controls, or provider-enforced restrictions.

## Troubleshooting

| Symptom | Likely cause | What to check |
|---|---|---|
| `pdftotext` returns an empty file | The PDF may contain scanned images rather than a text layer | Render pages and use OCRmyPDF or Tesseract |
| `tesseract` or `pdftotext` is “not recognized” on Windows | The executable directory is not on `Path`, or the terminal is old | Correct `Path`, then open a new terminal and re-run verification commands |
| The language is missing | The relevant trained-data package is not installed or discoverable | Run `tesseract --list-langs`; install the correct language data |
| OCR text is garbled | Poor image quality, wrong language, complex layout, or mixed scripts | Improve the source image, select the correct language, and manually verify the output |
| Tables lose their columns | Text extraction/OCR cannot always infer table structure | Preserve layout, extract a smaller section, or manually reconstruct the table |
| The extracted text is still too long | The source includes unnecessary pages or repeated content | Select relevant page ranges and retain the context required for the task |

## References

- [Tesseract OCR documentation](https://tesseract-ocr.github.io/tessdoc/)
- [Tesseract installation guide](https://github.com/tesseract-ocr/tessdoc/blob/main/Installation.md)
- [Ubuntu package: `tesseract-ocr`](https://packages.ubuntu.com/tesseract-ocr)
- [Ubuntu package: `poppler-utils`](https://packages.ubuntu.com/poppler-utils)
- [Poppler for Windows: releases and packaged binaries](https://github.com/oschwartz10612/poppler-windows/releases)
- [OCRmyPDF documentation](https://ocrmypdf.readthedocs.io/en/stable/)
- [OCRmyPDF installation guide](https://ocrmypdf.readthedocs.io/en/stable/installation.html)

---

This repository is a practical guide, not a guarantee of OCR accuracy or AI-platform acceptance. Verify commands and package names against the documentation for your operating system and software versions.

## License

The documentation and original written materials in this repository are licensed under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to share and adapt these materials, including for commercial purposes, provided you comply with the license terms and give appropriate attribution.

Unless otherwise stated, this license does not cover third-party software, trademarks, or materials identified as belonging to other rights holders.
