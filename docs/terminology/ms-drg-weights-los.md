---
id: ms-drg-weights-los
title: "MS DRG Weights and LOS"
---

import { JsonDataTable } from '@site/src/components/JsonDataTable';
import { JsonDataTableNoTerm } from '@site/src/components/JsonDataTableNoTerm';

<JsonDataTableNoTerm  jsonPath="nodes.seed\.the_tuva_project\.terminology__ms_drg_weights_los.columns" />

<a href="https://tuva-public-resources.s3.amazonaws.com/versioned_terminology/latest/ms_drg_weights_los.csv_0_0_0.csv.gz">Download CSV</a>

## Maintenance Instructions

1. Navigate to the [MS DRG Weights and LOS](https://www.cms.gov/medicare/payment/prospective-payment-systems/acute-inpatient-pps)
2. Go to the Acute Inpatient PPS section on the left and scroll down until you find "FY \{latest_year} IPPS Final Rule Home Page". For eg: "**FY 2025 IPPS Final Rule Home Page**"
3. Now search for Table 5 (it might contains description as *MS-DRGs, Relative Weighting Factors and Geometric and Arithmetic Mean Length of Stay*)
4. Click on the link and it will download the zip file
5. Unzip the downloaded folder and rename the XLSX file as '\{year}_input_file.xlsx'
6. Clean the data as:
    - Extract the fiscal year from the second column's name
    - Rename the columns to Tuva standard format
    - Inserting the extracted fiscal year as a new column
    - Save the final data as a csv file

    **Note*: you can use the below script to clean the data as said in **number 6***. 
    
    ```python
    import pandas as pd
    import re

    # Define standard column names
    STANDARD_COLUMNS = [
        "ms_drg", "final_post_acute_drg", "final_special_pay_drg", "mdc", "type",
        "ms_drg_title", "drg_weight_raw", "drg_weight", "geometric_mean_los", "arithmetic_mean_los"
    ]

    # File paths
    input_file_path  = "path/to/your/input_file.xlsx"  # Replace with your input file path
    output_file_path = "path/to/your/output_file.csv"  # Replace with your output file path

    def extract_year_from_column(column_name):
        """Extract the fiscal year from the column name."""
        match = re.search(r'FY\s*(\d{4})', column_name, re.IGNORECASE)
        return match.group(1) if match else None

    def process_file(file_path):
        """Process a single Excel file and return a cleaned DataFrame."""
        try:
            # Load the file with all columns as strings
            df = pd.read_excel(file_path, header=1, dtype=str)
            
            # Extract the year from the second column's name
            second_col_name = df.columns[1]
            year = extract_year_from_column(second_col_name)
            if not year:
                print(f"Skipping {file_path}: could not extract year from column name '{second_col_name}'")
                return None
            
            df.columns = STANDARD_COLUMNS[:len(df.columns)]
            
            # Add the year as a new column at index 1
            df.insert(1, "year", year)
            return df
        
        except Exception as e:
            print(f"Error processing file {file_path}: {e}")
            return None

    def main():    
        df = process_file(input_file_path)
        if df is not None:
            df.to_csv(output_file_path, index=False)

    if __name__ == "__main__":
        main()
    ```

7. Import the csv file into any data warehouse that also has the previous version of ms_drg_weights_los loaded
8. Append the latest data into the historical *ms_drg_weights_los* table

    ***Note**: you can use below query to append latest data to the historical ms_drg_weights_los table.*
    ```sql
    INSERT INTO {your_historical_table}
    SELECT *
    FROM {your_latest_data_table};
    ```
9. Upload the CSV file from the data warehouse to S3 (credentials with write permissions to the S3 bucket are required)
    ```sql
    -- example code for Snowflake
    copy into s3://tuva-public-resources/terminology/ms_drg_weights_los.csv
    from [table_created_in_step_8]
    file_format = (type = csv field_optionally_enclosed_by = '"')
    storage_integration = [integration_with_s3_write_permissions]
    OVERWRITE = TRUE;
    ```
10. Create a branch in [docs](https://github.com/tuva-health/docs).  Update the `last_updated` column in the table above with the current date
11. Submit a pull request

**The below steps are only required if the headers of the file need to be changed.  The Tuva Project does not store the contents of the ms_drg_weight_and_los file in GitHub.**

1. Create a branch in [The Tuva Project](https://github.com/tuva-health/tuva)
2. Alter the headers as needed in [ms_drg_weights_los file](https://github.com/tuva-health/tuva/blob/main/seeds/terminology/terminology__ms_drg_weights_los.csv)
3. Submit a pull request
