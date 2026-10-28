# OCArm

OCR for handwritten Armenian. You take a photo of a handwritten page, and OCArm turns it into text you can copy, or into a file (TXT, PDF, CSV or Excel).

## Why

In Armenia people spend a lot of time retyping handwritten papers by hand. OCArm tries to save that time.

## What it does

- **Upload image:** finds every word on the page, reads it, and puts the words back into lines.
- **Single word:** reads one cropped word.
- **Grammar fix (optional):** sends the raw OCR text to a language model that fixes spelling and decides if the page is a table or normal text.
- **Export:** tables become XLSX or CSV, text becomes PDF or TXT.
- **Web app:** a React front end that shows the image with a box around each word and the predicted text.

## How it works

```
photo -> CRAFT (find words) -> crop each word -> ResNet + BiLSTM + Attention (read word)
      -> sort words into lines -> (optional) LLM correction -> text or file
```

- **Detection and recognition** use the open-source [armenian-ocr](armenian-ocr/README.md) package by Portmind's team (CC BY-NC 4.0). It uses CRAFT to find words and a ResNet-BiLSTM-Attention model to read them. Its recognizer was originally trained on printed Armenian text.
- **Fine-tuning:** I fine-tuned the armenian-ocr model on handwritten Armenian that my friends and I collected.
- **Written for this project:** the FastAPI service (`app.py`), line sorting, the LLM post-processing (`llm_processor.py`, via OpenRouter), file export (`file_export.py`), the Docker setup, and the web front end (`src/`).

An earlier version (`hf-space/app.py`, `crnn.pt`) used a YOLOv8 word detector ([armvectores/yolov8n_handwritten_text_detection](https://huggingface.co/armvectores/yolov8n_handwritten_text_detection)) and a CRNN recognizer.
<!-- TODO (Alen): did you train crnn.pt yourself? On what data? -->

## API

| Endpoint | What it does |
|---|---|
| `GET /` | Health check |
| `POST /detect` | Word boxes only |
| `POST /recognize` | Read one word image |
| `POST /ocr` | Full page to text |
| `POST /ocr/correct` | Full page to text, then LLM correction |
| `POST /ocr/export` | Full page to a downloadable file |

## How to run

### Back end (Python)

```bash
git clone https://github.com/AlenGevorgyan/OCArm.git
cd OCArm
pip install -r requirements.txt
uvicorn app:app --port 7860
```

Model weights (`.pth`) are not stored in this repo. Put the armenian-ocr detection and recognition weights in `armenian-ocr/detection/` and `armenian-ocr/recognition/`, or set `DET_MODEL_DIR` and `REC_MODEL_DIR`.
<!-- TODO (Alen): add the link where people can download the weights. -->

Optional environment variables:

- `OPENROUTER_API_KEY` turns on grammar correction.
- `API_TOKEN` requires a bearer token on every request.

Or use Docker:

```bash
docker build -t ocarm .
docker run -p 7860:7860 ocarm
```

### Front end

```bash
npm install
npm run dev
```

The front end calls the API URL set in `src/lib/hf.ts`.

## Limits

- The handwriting dataset is small, so messy handwriting still gives errors.
- Line sorting is simple. Slanted or curved lines can come out in the wrong order.
- The LLM step can change words. Always check the result against the photo.
- No measured accuracy on handwriting yet.

## Next steps

- Collect and label more handwritten Armenian samples.
- Keep fine-tuning the recognizer as the dataset grows.
- Measure the error rate on a fixed test set.

## License

The `armenian-ocr/` folder is by Portmind's team under CC BY-NC 4.0 (non-commercial use only). See [armenian-ocr/LICENSE-CC-BY-NC-4.0.md](armenian-ocr/LICENSE-CC-BY-NC-4.0.md).
