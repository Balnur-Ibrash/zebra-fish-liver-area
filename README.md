# Zebrafish Liver Area Analysis

This repository contains microscopy images, raw Leica imaging data, and Jupyter notebooks for estimating zebrafish liver area from fluorescence images. The analysis detects bright liver regions, measures their contour areas, converts the measurements from pixels to square micrometres (µm²), and compares wild-type (`wt`) and knockout (`ko`) groups.

## Repository contents

- `image5.ipynb` — current analysis notebook. It supports colour and near-greyscale images, visualises detected contours, produces a results table, and runs a two-sample t-test.
- `image4.ipynb` — earlier version of the analysis workflow, retained for reference.
- `wt oc lipans clutch 1/` — JPEG microscopy images analysed by `image5.ipynb`.
- `lasx/` — raw Leica `.lif` microscopy data.
- `liver_report.pdf` — project report.

## Analysis workflow

For each image, the notebook:

1. Loads the image with OpenCV and converts it to HSV colour space.
2. Determines whether the image contains enough green pixels to be treated as a colour image.
3. Isolates the relevant signal and improves local contrast using CLAHE.
4. Applies a brightness threshold and finds external contours.
5. Filters contours by shape and keeps the largest expected regions.
6. Sorts detected regions from top to bottom to match the fish order encoded in the filename.
7. Converts contour area to µm² using the supplied pixel calibration.
8. Labels each measurement as wild type or knockout from the image filename.

The notebook then calculates group means and performs an independent two-sample t-test with equal variance assumed.

## Requirements

- Python 3
- JupyterLab or Jupyter Notebook
- OpenCV (`opencv-python`)
- NumPy
- pandas
- Matplotlib
- SciPy

Install the Python dependencies with:

```bash
python -m pip install jupyter opencv-python numpy pandas matplotlib scipy
```

## Running the analysis

1. Clone the repository and enter its directory:

   ```bash
   git clone git@github.com:Balnur-Ibrash/zebra-fish-liver-area.git
   cd zebra-fish-liver-area
   ```

2. Start Jupyter:

   ```bash
   jupyter lab
   ```

3. Open `image5.ipynb`.
4. Set `folder_path` to the microscopy image directory. For a clone of this repository, use:

   ```python
   folder_path = "wt oc lipans clutch 1"
   ```

5. Confirm the analysis parameters before running all cells:

   ```python
   data = process_images_in_folder(
       folder_path,
       brightness=60,
       pixel_per_mm=882,
       to_plot=True,
   )
   ```

`brightness` controls the segmentation threshold, `pixel_per_mm` is the image calibration used for conversion to µm², and `to_plot=True` displays detected contours for visual quality control.

## Image filename convention

The batch analysis extracts fish numbers and experimental groups from filenames with this general structure:

```text
fish <numbers> top <count> wt and bottom <count> ko.jpg
```

For example:

```text
fish 12 13 14 top three wt and bottom zero ko.jpg
```

The number of detected liver regions is inferred from the count of numeric tokens in the filename. Keep this naming convention when adding images so measurements are assigned to the correct fish and genotype.

## Notes

- Always inspect the plotted contours; threshold-based segmentation can be sensitive to brightness, contrast, and acquisition settings.
- Use the correct `pixel_per_mm` calibration for each image set before interpreting areas.
- The `.lif` file is about 79 MB. GitHub accepts it, but recommends Git LFS for files larger than 50 MB.
- The included statistical output is exploratory and depends on the segmentation parameters and experimental design.

