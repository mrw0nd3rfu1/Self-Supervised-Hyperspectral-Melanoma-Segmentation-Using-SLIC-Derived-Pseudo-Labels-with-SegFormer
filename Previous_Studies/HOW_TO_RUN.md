# How to Run This Project

A simple, step-by-step guide for running `HSI_Final_Code.ipynb` on Windows with VS Code.

The repo already includes the training data in `TrainData/` (267 images and 267 masks), so you do **not** need to download the dataset to run it.

---

## Part 1: One-time setup

### Step 1. Install Python 3.12

Use Python **3.12**, not the newest version. Brand-new Python versions are often not yet supported by PyTorch.

If you use the Python Install Manager from python.org, run this in PowerShell:

```powershell
py install 3.12
```

Check that it is installed:

```powershell
py list
```

### Step 2. Install the VS Code extensions

In VS Code, open the **Extensions** panel and install:

- **Python**
- **Jupyter**

### Step 3. Create the project environment

Open a terminal in VS Code (`` Ctrl+` ``) inside the project folder and run:

```powershell
py -3.12 -m venv .venv
```

Check that it uses Python 3.12:

```powershell
.venv\Scripts\python --version
```

### Step 4. Install the libraries

This takes several minutes.

```powershell
.venv\Scripts\python -m pip install --upgrade pip
.venv\Scripts\python -m pip install numpy scipy scikit-learn scikit-image opencv-python matplotlib pillow seaborn tqdm torch torchvision "transformers<5" accelerate albumentations ipykernel
```

Notes:

- `transformers<5` keeps the class names the notebook uses.
- If you have an NVIDIA GPU, get the install command for `torch` from [pytorch.org](https://pytorch.org) instead. Training will be much faster.
- You do **not** need to activate the environment. The commands above call its Python directly.

---

## Part 2: Run the notebook

### Step 5. Open the notebook and pick the kernel

1. Open `HSI_Final_Code.ipynb`.
2. Click **Select Kernel** in the top-right corner.
3. Choose **.venv**.

### Step 6. Go to the training section

Press `Ctrl+F` inside the notebook and search for **`seaborn`**.

This takes you to the large import cell where the training part begins.

### Step 7. Run from that cell to the end

Click inside that cell, then do one of these:

- Open the small dropdown next to the cell's run button and choose **Execute Cell and Below**, or
- Press `Ctrl+Shift+P` and run **Notebook: Execute Cell and Below**.

> **Do not use "Run All".** The middle of the notebook contains an old draft of the training code. On a fresh run it crashes, and it would also train the model twice.

### Step 8. Wait for training to finish

- The first run downloads a small pretrained model, so you need an internet connection.
- Training runs for 20 epochs.
- On a laptop CPU this can take up to about an hour. On a GPU it takes a few minutes.

### Step 9. Check the results

The last cell prints the final metrics. They should be close to:

| Metric   | Expected |
|----------|----------|
| Accuracy | 0.98     |
| Dice     | 0.79     |
| IoU      | 0.66     |

Small differences between runs are normal.

Two new folders are created:

| Folder              | Contents                                      |
|---------------------|-----------------------------------------------|
| `SegformerResults/` | Saved model and `final_metrics.txt`           |
| `Visualizations/`   | `training_metrics.png`, `predictions.png`, `overlay_predictions.png` |

---

## Optional: Rebuild the training data from the raw dataset

Only do this if you want to regenerate the pseudo labels yourself.

1. Download the HSIDermoscopy data using the link at [github.com/heugyy/HSIDermoscopy](https://github.com/heugyy/HSIDermoscopy).
2. Inside the project, create this folder structure and put the `.mat` files in it:

   ```
   Dataset/
   ├── DNCube/
   ├── MMCube/
   └── OtherCube/
   ```

3. Starting from the top of the notebook, run the cells one at a time with `Shift+Enter`.
4. Stop after the cell that prints the **Images** and **Masks** counts. This overwrites `TrainData/`.
5. Continue from **Step 6** above.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `py is not recognized` | Python is not installed, or VS Code was not restarted after installing it. |
| `ModuleNotFoundError` | The notebook is not using the `.venv` kernel. Select it again and restart the kernel. |
| Training is too slow | Search for `num_train_epochs` in the training cell and change `20` to `5` for a quick test. |
| Out of memory | Close other apps. The batch size is already small. |
| Want to type `python` instead of the full path | Run `.venv\Scripts\Activate.ps1`. If PowerShell blocks it, run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once and try again. |

---

## Note on other datasets

This notebook only works with the **HSIDermoscopy** melanoma data in `.mat` format. Other hyperspectral datasets, such as the HistologyHSI breast cancer data, need code changes before they will work:

- A different file reader. ENVI `.hdr`/`.dat` files need a library such as `spectral`.
- Loading one image at a time, because each cube is very large.
- A different labeling method. The current one looks for a single lesion near the image center.
- Enough images to train on.

---

## Part 3: Compare with and without PCA

`HSI_NoPCA_Comparison.ipynb` runs the same pipeline **without PCA** and compares it with the original. It does not change the original notebook or its outputs.

### What it compares

| Model | Model input | Trained on |
|---|---|---|
| **Original** (with PCA) | 3 PCA components | PCA pseudo labels |
| **A**: no PCA in model input | all spectral bands | PCA pseudo labels |
| **B**: no PCA anywhere | all spectral bands | no-PCA pseudo labels |

All three are scored on the same validation images, against both sets of pseudo labels.

### Step 1. Download the raw dataset

The no-PCA version needs the original hyperspectral cubes, not the PNG images in `TrainData/`.

1. Open [github.com/heugyy/HSIDermoscopy](https://github.com/heugyy/HSIDermoscopy).
2. Download the **"Data cube after correction"** folder from the Google Drive link.
3. Inside the project, arrange the `.mat` files like this. Rename the folders if the download uses different names.

   ```
   Dataset/
   ├── DNCube/
   ├── MMCube/
   └── OtherCube/
   ```

### Step 2. Check the original outputs exist

The comparison reuses what the original notebook produced. These must exist:

- `TrainData/images/` and `TrainData/masks/`
- `SegformerResults/final_model/`

### Step 3. Run the comparison notebook

1. Open `HSI_NoPCA_Comparison.ipynb`.
2. Select the **.venv** kernel.
3. Click **Run All**. This notebook is safe to run from top to bottom.

What to expect:

- **Section 3** builds the no-PCA data. It takes a few minutes and uses about 1 to 2 GB of disk.
- **Section 6** trains two models, A and B. **Each takes about as long as the original training.**
- If you stop halfway, run it again. Finished data and trained models are reused, not rebuilt.
- To save time, set `RUN_EXPERIMENT_B = False` in Section 0 to train only model A.

### Step 4. Read the results

The last code cell prints a summary like this:

```
1. Remove PCA from the model input only
   Original vs A, both scored on PCA labels: 0.79 -> ...
2. Remove PCA everywhere
   Original on PCA labels vs B on no-PCA labels: 0.79 -> ...
3. Pseudo labels with vs without PCA agree with mean Dice ...
```

How to read it:

- **Line 1 is the fairest comparison.** Only the input changes, and both models are scored on the same labels. A positive change means all bands work better than PCA.
- **Line 2 is the full no-PCA pipeline.** Each model is scored on its own labels, as in the original report. Treat it as indicative, because the labels differ.
- **Line 3 shows how much PCA changes the pseudo labels.** A value near 1 means the labels are almost the same.
- **None of the labels are manual ground truth.** These scores measure agreement with pseudo labels, not clinical accuracy.

All outputs are saved in the `Comparison/` folder:

| File | What it shows |
|---|---|
| `comparison_results.csv` | Every metric for every model and label set |
| `pseudo_label_agreement.csv` | Per-image agreement between PCA and no-PCA labels |
| `metrics_comparison.png` | Bar chart of Dice and IoU |
| `training_curves.png` | Validation Dice per epoch for each model |
| `visual_comparison.png` | Example images, labels and predictions side by side |

### Notes

- **The original model's score may differ slightly from its own notebook.** The comparison scores without random flips, so results are repeatable. It also uses the standard specificity formula, which the original notebook did not.
- **Check the no-PCA labels in `visual_comparison.png`.** If they look like square blocks, lower `SLIC_COMPACTNESS_NO_PCA` in Section 0. If they look broken or ragged, raise it. Then delete the `NoPCA_TrainData/` and `NoPCA_Results/` folders and run again.

---

## Part 4: Run the comparison on Google Colab, with no download

Use this if downloading the dataset is too slow. Colab reads the dataset straight from Google Drive and has a free GPU, so training is also much faster.

1. **Add the dataset to your Drive.** Sign in to Google and open the [dataset link](https://drive.google.com/drive/folders/13ZXXxtUMjEzLAjoAqC19IJEc5RgpFf2f). Open the menu next to the folder name **HSI_datacube_after_correction**, then choose **Organize → Add shortcut → My Drive**. Nothing is downloaded.
2. **Upload the project files.** Upload `HSI_project_for_colab.zip` from the project folder to **My Drive**. Don't unzip it. It holds `TrainData/` and the original model, about 66 MB.
3. **Open the notebook in Colab.** Go to [colab.research.google.com](https://colab.research.google.com), choose **File → Upload notebook**, and pick `HSI_NoPCA_Comparison.ipynb`.
4. **Turn on the GPU.** Choose **Runtime → Change runtime type → T4 GPU**.
5. **Run everything.** Choose **Runtime → Run all**, and allow Google Drive access when asked.
6. **Get the results.** They are copied to **My Drive → HSI_NoPCA_outputs → Comparison**. Download that folder.

Keep the shortcut and zip names exactly as above. The notebook looks for them by name.

---

## Part 5: HELICoiD dataset, with and without PCA, on Google Colab

`HSI_HELICoiD_PCA_Comparison.ipynb` tests the PCA step on the HELICoiD brain-tumour data with the same SegFormer. Nothing is downloaded to your computer.

1. **Add the data to your Drive.** In Google Drive, open **Shared with me**, right-click **dataset_helicoid**, and choose **Organize → Add shortcut → My Drive**. Keep the name `dataset_helicoid`.
2. **Open the notebook in Colab.** At [colab.research.google.com](https://colab.research.google.com), choose **File → Upload notebook** and pick `HSI_HELICoiD_PCA_Comparison.ipynb`.
3. **Turn on the GPU.** **Runtime → Change runtime type → T4 GPU**.
4. **Run everything.** **Runtime → Run all**, and allow Google Drive access.
5. **Check Section 2's output.** It should say how many images it found. If it finds 0, copy that output and send it.
6. **Wait about 1 to 1.5 hours.** Keep the Colab tab open. If it stops, run all again: finished steps are skipped.
7. **Send the results.** Copy the summary printed by the last cell. The charts and tables are in **My Drive → HELICoiD_PCA_outputs**.

If copying the zips keeps failing, Google may be limiting downloads of the shared files. Right-click each zip, choose **Make a copy**, and move the copies into a folder named `dataset_helicoid` in My Drive. This needs about 12 GB of free Drive space.

---

## Part 6: HELICoiD with MedFormer, on Google Colab

`HSI_HELICoiD_MedFormer_PCA_Comparison.ipynb` repeats Part 5 with MedFormer in place of SegFormer. It uses the same data, folds and scoring. The steps are the same as Part 5:

1. **Sign in to Colab with your school account**, the one that has the `dataset_helicoid` shortcut.
2. **Upload the notebook.** **File → Upload notebook** → `HSI_HELICoiD_MedFormer_PCA_Comparison.ipynb`.
3. **Turn on the GPU.** **Runtime → Change runtime type → T4 GPU**.
4. **Run everything.** **Runtime → Run all**, and allow Google Drive access.
5. **Check the time after the first training.** Each line that starts with `Fold 1/5` ends with the minutes so far. There are 10 trainings in total.
6. **If Colab disconnects,** run all again. Finished trainings are saved on Drive and skipped. Only the data preparation is repeated.
7. **Download the results** from **My Drive → HELICoiD_MedFormer_outputs** and put them in the project folder.

---

## Part 7: HELICoiD with SegFormer + MedFormer, on Google Colab

`HSI_HELICoiD_SegFormer_MedFormer_PCA_Comparison.ipynb` keeps the SegFormer from Part 5 and attaches MedFormer at the end. MedFormer receives the image plus SegFormer's prediction and gives the final result. The combined model is trained with and without PCA, then compared with SegFormer alone (Part 5) and MedFormer alone (Part 6).

**Before you start,** check that these two folders are still in **My Drive**. The notebook reads the earlier results from them for the comparison:

- `HELICoiD_PCA_outputs` (SegFormer alone)
- `HELICoiD_MedFormer_outputs` (MedFormer alone)

If one is missing, upload it from the project folder with exactly that name. Without it, training still runs, but that model is left out of the comparison.

1. **Sign in to Colab with your school account**, the one that has the `dataset_helicoid` shortcut and the two results folders.
2. **Upload the notebook.** **File → Upload notebook** → `HSI_HELICoiD_SegFormer_MedFormer_PCA_Comparison.ipynb`.
3. **Turn on the GPU.** **Runtime → Change runtime type → T4 GPU**.
4. **Run everything.** **Runtime → Run all**, and allow Google Drive access.
5. **Check two lines early on:**
   - `Same images and labels as the earlier SegFormer run: yes`. If it says **NO**, send that output.
   - After the first training, each `Fold ...` line shows the time so far and an estimate of the time left. There are 10 trainings in total.
6. **If Colab disconnects,** run all again. Finished trainings are saved on Drive and skipped.
7. **Send the results.** Copy the summary printed by the last cell, and download **My Drive → HELICoiD_SegFormer_MedFormer_outputs** into the project folder.
