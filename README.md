# OCR Service Module

FastAPI-based OCR service using PaddleOCR for multilingual text extraction from images.

## Credits

This service uses **PaddleOCR** - a practical ultra-lightweight OCR system developed by PaddlePaddle.

- **GitHub Repository**: [https://github.com/PaddlePaddle/PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)
- **PyPI Package**: [https://pypi.org/project/paddleocr/](https://pypi.org/project/paddleocr/)
- **Documentation**: [https://paddlepaddle.github.io/PaddleOCR/](https://paddlepaddle.github.io/PaddleOCR/)

PaddleOCR supports 80+ languages including Latin, Chinese, Arabic, Devanagari, Cyrillic, and more.

## Features

- **Multi-language Support**: Supports multiple Indian languages and English
- **PaddleOCR Integration**: Uses PaddleOCR for accurate text detection and recognition
- **Configurable Confidence Threshold**: Filter results by confidence score2>&1
- **RESTful API**: Easy-to-use FastAPI endpoints
- **Model Caching**: Efficient model loading and reuse

## Directory Structure

```
ocr/
├── server.py           # FastAPI server implementation
├── languages.json      # Language configurations
├── fonts/             # Font files for different languages
├── models/            # PaddleOCR model files (auto-downloaded)
└── README.md          # This file
```

## Language Support

This service supports **all languages available in PaddleOCR** (80+ languages). The `languages.json` file contains configurations for commonly used languages with their respective font mappings for rendering purposes.

For the complete list of supported languages, refer to the [PaddleOCR documentation](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.7/doc/doc_en/multi_languages_en.md).

## Installation

### Dependencies

```bash
uv sync # or pip install -r requirements.txt
```

### Font Files (Optional)

Place corresponding font files in the `fonts/` directory for better text rendering:

- `NotoSans-Regular.ttf` (English)
- `NotoSansDevanagari-Regular.ttf` (Hindi, Marathi)
- `NotoSansTamil-Regular.ttf` (Tamil)
- `NotoSansTelugu-Regular.ttf` (Telugu)
- `NotoSansKannada-Regular.ttf` (Kannada)
- `NotoSansMalayalam-Regular.ttf` (Malayalam)
- `NotoSansBengali-Regular.ttf` (Bengali)

## Running the Server

### Default (Port 8001)

```bash
python server.py
```

### Custom Host/Port

```bash
uvicorn server:app --host 0.0.0.0 --port 8001 --reload
```

## API Endpoints

### 1. Root Endpoint

**GET** `/`

Returns service information and available endpoints.

```bash
curl http://localhost:8001/
```

### 2. Get Supported Languages

**GET** `/languages/`

Returns list of all supported languages with their configurations.

```bash
curl http://localhost:8001/languages/
```

### 3. Health Check

**GET** `/health/`

Returns service health status and loaded models.

```bash
curl http://localhost:8001/health/
```

### 4. Extract Text from Image

**POST** `/extract_text/`

Extract text from an uploaded image.

**Parameters:**

- `file`: Image file (required)
- `lang`: Language code (optional, default: "en")
- `min_confidence`: Minimum confidence threshold 0.0-1.0 (optional, default: 0.7)

**Example Request:**

```bash
# English OCR
curl -X POST "http://localhost:8001/extract_text/?lang=en&min_confidence=0.7" \
  -F "file=@image.jpg"

# Hindi OCR
curl -X POST "http://localhost:8001/extract_text/?lang=hi&min_confidence=0.8" \
  -F "file=@hindi_document.jpg"

# Tamil OCR with lower confidence
curl -X POST "http://localhost:8001/extract_text/?lang=ta&min_confidence=0.6" \
  -F "file=@tamil_image.png"
```

**Response Format:**

```json
{
  "extracted_text": "Complete extracted text\nwith line breaks",
  "details": [
    {
      "text": "Text block 1",
      "confidence": 0.9523,
      "bounding_box": {
        "top_left": [10.5, 20.3],
        "top_right": [100.2, 20.5],
        "bottom_right": [100.0, 45.8],
        "bottom_left": [10.3, 45.5]
      }
    }
  ],
  "metadata": {
    "language": "hi",
    "language_name": "Hindi",
    "image_size": {
      "width": 800,
      "height": 600
    },
    "processing_time_ms": 1234.56,
    "total_detections": 15,
    "confidence_threshold": 0.7
  }
}
```

## Python Client Example

```python
import requests

# Upload and extract text
with open("document.jpg", "rb") as f:
    response = requests.post(
        "http://localhost:8001/extract_text/",
        params={"lang": "hi", "min_confidence": 0.7},
        files={"file": f}
    )

data = response.json()
print("Extracted Text:", data["extracted_text"])
print("Total Detections:", data["metadata"]["total_detections"])
```

## Configuration

### Adding a New Language

Edit `languages.json`:

```json
{
  "languages": {
    "new_lang": {
      "name": "Language Name",
      "code": "new_lang",
      "font": "FontFile.ttf"
    }
  }
}
```

Ensure PaddleOCR supports the language code.

## Performance Tips

1. **GPU Acceleration**: Set `use_gpu=True` in `get_ocr_model()` if CUDA is available
2. **Model Caching**: Models are cached after first use for faster subsequent requests
3. **Confidence Threshold**: Adjust `min_confidence` to balance accuracy vs. coverage
4. **Image Size**: Resize large images before upload for faster processing

## Troubleshooting

### Models Not Downloading

- Ensure internet connectivity
- Check write permissions for `models/` directory
- PaddleOCR downloads models automatically on first use

### Low Accuracy

- Try adjusting `min_confidence` threshold
- Ensure correct language code is specified
- Check image quality (resolution, contrast, clarity)

## API Documentation

Interactive API documentation available at:

- Swagger UI: http://localhost:8001/docs
- ReDoc: http://localhost:8001/redoc

## License

PaddleOCR is licensed under the Apache License 2.0.
