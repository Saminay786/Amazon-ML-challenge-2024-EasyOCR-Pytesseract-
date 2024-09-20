# Amazon-ML-challenge-2024-EasyOCR-Pytesseract-
**Problem Statement:**

The objective of this project is to build a machine learning model that extracts entity values from product images. This capability is critical in industries such as healthcare, e-commerce, and content moderation, where obtaining accurate product information directly from images is essential. As digital marketplaces grow, many products lack detailed textual descriptions, making it necessary to extract key information from images, including weight, volume, voltage, wattage, and dimensions, among other attributes. The solution will help digital stores streamline product information management.

Data Description
The dataset provided includes the following key columns:

1.index: A unique identifier (ID) for each data sample.

2.image_link: A URL linking to the product image, available for download. Example: Image URL. The images can be downloaded using the download_images function from src/utils.py.

3.group_id: Category code for the product.

4.entity_name: Product entity name (e.g., "item_weight").

5.entity_value: Product entity value (e.g., "34 grams"). This column is not present in the test data, as it is the target variable.


**Output Format**

The output should be a CSV file with the following two columns:

1.index: The unique identifier of the data sample, matching the test record index.

2.prediction: A string in the format "x unit", where x is a float and unit is one of the allowed units. For example, valid outputs include "2 grams", "12.5 centimetres", and "2.56 ounces". If no value is found in the image, return an empty string ("").

Invalid Cases:

Example: "2 gms", "60 ounce/1.7 kilogram", "2.2e2 kilogram" are invalid formats.

Make sure the output format is consistent with the sample provided and passes through the src/sanity.py checker script to ensure validity.


Steps we took to solve our problem: Here is a breakdown of the code in four parts: 

**1. Processing and downloading Images in Parallel**

In this section, we start by downloading the image from the given URL using the requests.get method. The image is converted into a NumPy array, then decoded with OpenCV’s cv2.imdecode function. This is the main function that coordinates the entire process of downloading, processing, and saving images. It reads data from the input excel_file and uses ThreadPoolExecutor to process images in parallel with up to 8 threads (to increase efficiency). For each row of data in the Excel file, the process_image function is called to process the image link. Successfully processed images are stored in a list, and the code prints a progress update every 10 images. After all images have been processed, the results are saved to the output Excel file. 2. Extract Text from Images Using EasyOCR and Save Results to CSV:

Libraries Used: cv2, numpy, easyocr, os, pandas, torch.

Process:

1. Images are loaded from a folder using os.listdir().

2. Text is extracted from each image using EasyOCR, a deep learning-based OCR library that supports GPU acceleration via CUDA.

3. The recognize_text() function uses EasyOCR to extract text from each image.

4. The extracted text is saved in a CSV file along with the corresponding image names. The text is only saved if the recognition probability is above 0.2.

5. The GPU (CUDA) is used for text recognition if available, otherwise, the CPU is used.

6. The resulting CSV contains two columns: Image Name and Detected Text.


**2. Extract and Standardize Units from Detected Text:**

Libraries Used: pandas, re.

Process:

1. A dictionary, entity_unit_map, defines unit types (e.g., length, weight, volume) and their possible variations (e.g., "cm", "mm", "kg").

2. A unit_conversion_map converts unit abbreviations or variants into a standardized form.

3. The correct_o_to_0() function corrects errors where the character 'O' is mistakenly used in numeric values.

4. The extract_numbers_with_units() function extracts numbers followed by units (e.g., "20 kg") from the detected text.

5. A CSV with image names and detected text is read, and for each row, unit-specific information (e.g., width, weight, height) is extracted and standardized into the desired units.

6. The results are saved in a new CSV file with the standardized values.

**3. Merging Two CSV Files on Image Names:**

Libraries Used: pandas.

Process:

1. Two CSV files are loaded: one with image links and entity names, the other with image names and detected text from OCR.

2. The image name is extracted from the image link (assuming the file name is at the end of the URL).

3. The two datasets are merged on the common field, 'Image Name', using a left join.

4. The merged dataset is saved as a new CSV file, containing the image name, link, entity name, and OCR-detected text.

**4. Concatenating Multiple CSV Files:**

Libraries Used: pandas, os.

Process:

1. The folder path containing multiple CSV files is provided.

2. Each CSV file in the folder is read into a DataFrame and stored in a list.

3. All DataFrames are concatenated into a single large DataFrame.

4. The combined data from all CSV files can be processed or saved for further analysis.

**5. Entity-Unit Extraction and Assignment**

Summary: This code extracts and assigns entity values and their units from text for attributes like weight, volume, voltage, and wattage. Key Components:

Libraries: Uses spaCy (NLP), regex (for matching), and pandas (CSV handling).

Entity-Unit Map: A dictionary linking entity types (e.g., weight, voltage) to possible units (e.g., grams, kilograms).

Text Extraction: The extract_and_detect_first function identifies numbers followed by valid units for each entity type.

Value Assignment: The assign_values function formats and assigns the first detected value-unit pair to the appropriate column.

CSV Processing: The process_csv function processes a CSV file, extracting and assigning entity values row by row, and saves the updated file. 

Usage: Call process_csv with the file path to update the CSV. 

**6. Measurement Extraction and Conversion**

Summary: This code extracts and converts size measurements (e.g., height, width, depth) from various units (cm, mm, meters, inches, feet) to millimeters for uniformity. Key Components:

Libraries: Uses spaCy, regex, and pandas.

Conversion Factors: A dictionary holds conversion ratios for units (e.g., 1 cm = 10 mm, 1 foot = 304.8 mm).

Unit Priority: A list determines which unit takes precedence when multiple are detected.

Extraction and Conversion: The extract_and_convert function extracts values and converts them to millimeters, sorted by size.

Value Assignment: The assign_values function assigns the largest value to height, the second largest to width, and so on.

CSV Processing: The process_csv function processes a CSV file, extracts and converts size measurements, and saves the updated file. 

Usage: Call process_csv with the file path to update the CSV. Extras: Using Tesseract for those images (561) whose text have not been detected by EasyOCR. Functions are made to rotate image clockwise and, counterclockwise to detect text from image.
