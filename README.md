# Bulk Image Upscaler

Batch image upscaling and AI title/keyword generation in Google Colab, with a simple Gradio interface.

**Choose settings → Upload images → Start → Download one ZIP**

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AFLAHAFI/bulk_image_upscaler/blob/main/bulk_image_upscaler_automatic_fast.ipynb)

## Features

- **2×, 4× and 8× upscaling** with Real-ESRGAN. The 8× option runs 4× followed by 2×.
- **Two tasks:** upscale with metadata, or generate metadata while preserving original image bytes.
- **Short titles:** maximum 10 words and 70 characters.
- **Single-word English keywords:** for example, `solar, panels, energy, roof`. Duplicates and selected filler words are removed; the maximum is 49 keywords.
- **Short filenames:** 3–4 words, up to 30 characters including the extension and any collision suffix.
- **Live elapsed time** and approximate remaining time for the current stage.
- **Automatic retry** of failed work once, plus per-image status and timing reports.
- **Optional Google Drive persistence** and reuse of verified outputs from matching batches.
- **One final ZIP** containing available images, metadata CSV files and a processing report.

## Quick start in Google Colab

1. Click **Open in Colab** above. Save a copy to your Drive if you want to edit the notebook.
2. Select **Runtime → Change runtime type → T4 GPU** (subject to availability).
3. Run the section labelled **Cell 1**, then **restart the session**.
4. Run **Cell 2** to install dependencies and apply the BasicSR compatibility patch, then **restart the session again**.
5. Optionally run **Mount Google Drive** for persistent batch progress.
6. Run **Cell 3**. Open its Gradio link and sign in with username `owner` and the randomly generated password printed in that cell.
7. Choose the task, scale and output format. Upload images and click **Start → Create final ZIP**.
8. Download the ZIP and check `processing_report.csv` for failures or missing metadata.

**Run the setup sections in order; do not use Run all across the required restarts.** Keep the Colab session running during processing. Test one small image before starting a large batch.

## Basic settings

| Setting | Options / behavior |
|---|---|
| Task | Upscale and metadata; Metadata only |
| Upscale | 2×, 4× or 8× |
| Save as | PNG, JPG or WEBP |
| Save progress to connected Drive | Uses the Drive mounted in the notebook |
| Input formats | JPG, JPEG, PNG and WEBP |

In metadata-only mode, original image bytes and format are retained. Scale and output-format selections do not convert those images.

The automatic interface uses Qwen for metadata. Additional helper functions retained in the notebook are not extra settings in this interface. Face restoration, additional enhancement and white-balance correction are disabled in the automatic workflow.

## Output files

The final `final_images.zip` contains:

| File | Purpose |
|---|---|
| Short-named images | Successfully processed images, in the chosen format |
| `adobe_stock_metadata.csv` | Image filenames, titles and keywords |
| `processing_report.csv` | Per-input status, errors, flags and timings |
| `README.txt` | Export summary and review notes |

Large metadata exports are split into numbered CSV files automatically. The metadata CSV uses UTF-8 and these columns:

```csv
Filename,Title,Keywords,Category,Releases
```

Filenames match the exported image names. `Category` and `Releases` are left blank by the automatic workflow. The CSV name and column layout support a stock-metadata workflow; this app does not upload or submit anything to a stock platform and does not guarantee acceptance.

The model is asked for approximately **20–25 relevant single-word keywords**, with fewer for simple scenes. The filter keeps usable words, up to 49; it does not pad short lists with unrelated terms. Titles and keywords are AI drafts and should be checked for accuracy.

The processing report includes image/metadata status, screening results, possible duplicates, image-stage seconds, metadata-stage seconds and error notes. Entries are explicitly marked `Human_Reviewed=NO`.

If metadata fails, a successfully processed image still goes into the ZIP, but its metadata row is omitted. A completed ZIP can contain fewer metadata rows than images. Screening and technical flags are advisory.

## Performance changes in this version

- Loaded models are cached within the runtime and moved between CPU and GPU for the relevant phase, reducing repeated construction.
- Metadata generation uses a smaller image-token budget and a 256-token response limit.
- PNG compression effort is reduced, and already-compressed images are stored directly in the ZIP. These compression choices preserve pixels but may increase file size.
- Matching saved images can be reused; metadata from an older ruleset is regenerated when necessary.

**The upscaling calculations, model choices and numerical precision are unchanged by these speed-focused changes.** There is no measured seconds-per-image guarantee. The first run downloads and loads models and can be substantially slower. Runtime depends on input dimensions, scale, GPU, host RAM and Drive speed. Cached inactive models also consume host RAM.

## Persistence and privacy

- Without Drive, batch files live in the temporary Colab runtime and can disappear when it resets.
- With Drive enabled, batches are stored under `MyDrive/UpscalerWorkspaceV5`.
- To reuse a matching saved batch, mount the same Drive and upload the same inputs with the same settings. Input hashes, settings and output checksums identify reusable work.
- This is a **cloud Colab workflow**: selected images are uploaded to the Colab runtime. It is not an offline Android app.
- The Gradio share link is protected by a fresh password printed at launch. Keep that password private. Model downloads require internet access.

## Troubleshooting

| Problem | What to try |
|---|---|
| Dependency/import error | Follow both setup restarts in order. If needed, use a fresh runtime and repeat setup. |
| Detailed metadata requires a GPU | Select a GPU runtime and rerun the app cell. |
| GPU memory error | Try a smaller image or lower scale. Automatic tiling retries cannot guarantee every image fits. |
| Native output exceeds the 120 MP guard | Use a smaller source or lower scale. Scaling dimensions by 4× multiplies pixel count by 16. |
| First image is slow | Allow initial model downloads/loading. Estimates become available after measurements. |
| Image exported without a CSV row | Read the metadata error in the report and retry the matching batch. |
| Only a few keywords | Simple scenes may support fewer accurate keywords. The app avoids filler. |
| Gradio link stops working | Reconnect the Colab runtime and rerun the app cell. |

## Models and implementation

- [Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN): image upscaling (`RealESRGAN_x2plus`, `RealESRGAN_x4plus`).
- [Qwen2.5-VL-3B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct): image-based title and keyword generation.
- [Gradio](https://github.com/gradio-app/gradio): browser interface.
- PyTorch, Transformers, Pillow and OpenCV: inference and image handling.

The notebook also retains BLIP and GFPGAN helpers from earlier versions. Dependencies are installed by the notebook; a separate requirements file would not capture its required restarts and compatibility patch. Model weights are downloaded at runtime and are not stored in this repository. Third-party software and models retain their own licenses.

## Validation

For this repository upload, notebook JSON/schema and Python cell syntax were checked, and saved cell outputs were cleared before publishing. Earlier local checks used simulated AI to exercise keyword/title rules, timing updates, cached-image reuse and ZIP consistency. **Actual model output and GPU performance still need testing in Colab.**

## Repository contents

- `bulk_image_upscaler_automatic_fast.ipynb` — complete Colab notebook.
- `README.md` — setup, features, output details and troubleshooting.
- `.gitignore` — excludes runtime artifacts, model weights and local credentials.

Maintained by [AFLAHAFI](https://github.com/AFLAHAFI).
