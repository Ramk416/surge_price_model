# NYC Yellow Taxi Data Processing

This project processes large NYC Yellow Taxi trip datasets using Python and pandas. It filters, cleans, and saves a smaller, more manageable CSV file for further analysis.

## Project Structure

- `texi_project.ipynb` — Main Jupyter notebook with all data processing code.
- `yellow_tripdata_2015-01.csv` — Raw taxi trip data for January 2015.
- `yellow_tripdata_jan2016.csv` — Filtered and cleaned dataset (output).
- `.ipynb_checkpoints/` — Jupyter notebook checkpoints.

## How It Works

1. **Chunked Reading:**  
   The notebook reads the large CSV file in chunks to avoid memory errors.

2. **Filtering:**  
   Only rows where the pickup month is January are kept.

3. **Cleaning:**  
   Rows with invalid or midnight pickup/dropoff times are removed.

4. **Saving:**  
   The cleaned, filtered data is saved as `yellow_tripdata_jan2016.csv`.

## Usage

1. Open `texi_project.ipynb` in Jupyter Notebook or VS Code.
2. Ensure `yellow_tripdata_2015-01.csv` is in the same directory.
3. Run the notebook cells sequentially.
4. The processed file `yellow_tripdata_jan2016.csv` will be created.

## Requirements

- Python 3.13+
- pandas
- numpy

Install dependencies with:

```sh
pip install pandas numpy
```

## Notes

- The notebook uses chunked reading to handle large files efficiently.
- For more details, see the code and comments in [`texi_project.ipynb`](e:/texi_project/dataset/texi_project.ipynb).