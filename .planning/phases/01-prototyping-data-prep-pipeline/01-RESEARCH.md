# Phase 01: Prototyping Data Prep Pipeline - Research

## Objective
Research the setup and best practices for comparing Typhoon OCR API and local PaddleOCR within a Jupyter Notebook to prepare for the Thai Election 2026 data extraction.

## Findings

### Typhoon OCR (API)
- Typhoon OCR is a LLM-based API tailored for the Thai language. It typically parses visually structured documents extremely well, converting them directly to markdown or text.
- **Requirements**: Need an API key, handled via `dotenv`. The `requests` or `httpx` library in Python is sufficient to make API calls to their endpoints.
- **Handling Images**: Images usually need to be base64-encoded to be sent to the API.

### PaddleOCR (Local)
- PaddleOCR supports Thai out-of-the-box using the `ch` and `en` models or specifically trained `thai` models depending on the version.
- **Requirements**: `pip install paddlepaddle paddleocr`.
- **Usage**:
  ```python
  from paddleocr import PaddleOCR
  ocr = PaddleOCR(use_angle_cls=True, lang='th')
  result = ocr.ocr(img_path, cls=True)
  ```
- **Handling Images**: Accepts numpy arrays or image paths.

### `pdf2image` Usage
- Converts PDF pages into PIL components using the `convert_from_path` function.
- **System Requirements**: Requires `poppler` installed natively on the OS (`sudo apt-get install poppler-utils` on Linux).

### Strict Template Regex
- Given the ECT form (ส.ส. 5/18), the extraction needs to identify the polling station number, sub-district, etc.
- Forms typically have tables mapping candidate numbers to vote counts. The regex should look for anchors such as "ผู้สมัครหมายเลข" followed by digits, or look across lines assuming the full-page text parsing retains line breaks.

### Side-by-Side Validation Strategy
- We will construct the prototyping notebook to load 1-2 PDF pages using `pdf2image`.
- Convert the image to base64 for Typhoon and pass it directly to PaddleOCR.
- Display the images alongside the raw extracted text of both models.
- Since this is a prototyping notebook, manually computing accuracy (e.g., character error rate or vote count match percentage) will be sufficient to justify the final choice of engine.

## Conclusion
The technical requirements are clear: `pdf2image`, `paddleocr`, `paddlepaddle`, `python-dotenv`, and `requests`. The prototyping will be straightforward.
