# Nashville Housing Data Cleaning Project

This project focuses on cleaning and preparing a Nashville housing dataset for further analysis. Using SQL, various data cleaning techniques are applied to ensure the dataset is standardized, accurate, and ready for analysis. This project involves handling missing data, transforming date formats, splitting address fields, and removing duplicates.

## Project Objectives:
1. Standardize the date formats for consistent reporting.
2. Populate missing property addresses by referencing Parcel IDs.
3. Split addresses into separate columns (Address, City, State) for easier analysis.
4. Clean and transform specific fields, such as changing 'Y' and 'N' values to 'Yes' and 'No'.
5. Identify and remove duplicate records based on key columns.
6. Remove unused columns to streamline the dataset.

## Key Data Cleaning Steps:

### 1. **Standardize Date Format**:
   The date format is standardized using SQL's `CONVERT()` function. A new column (`SaleDateConverted`) is created to store the cleaned date values.

   ```sql
   UPDATE NashvilleHousing
   SET SaleDate = CONVERT(DATE, SaleDate);
   
   ALTER TABLE NashvilleHousing
   ADD SaleDateConverted DATE;

   UPDATE NashvilleHousing
   SET SaleDateConverted = CONVERT(DATE, SaleDate);
   ```

### 2. **Populate Missing Property Addresses**:
   Missing property addresses are populated by referencing Parcel IDs and updating the `PropertyAddress` field where applicable.

   ```sql
   UPDATE a
   SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
   FROM NashvilleHousing a
   JOIN NashvilleHousing b
   ON a.ParcelID = b.ParcelID
   AND a.[UniqueID] <> b.[UniqueID]
   WHERE a.PropertyAddress IS NULL;
   ```

### 3. **Splitting Address into Individual Columns**:
   Addresses are split into separate columns for easier analysis. Fields such as `PropertySplitAddress`, `PropertySplitCity`, and `PropertySplitState` are created to store the split data.

   ```sql
   ALTER TABLE NashvilleHousing
   ADD PropertySplitAddress NVARCHAR(255), PropertySplitCity NVARCHAR(255), PropertySplitState NVARCHAR(255);

   UPDATE NashvilleHousing
   SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1);
   ```

### 4. **Transform 'Sold as Vacant' Field**:
   The values in the `SoldAsVacant` field are standardized by converting 'Y' to 'Yes' and 'N' to 'No' using a `CASE` statement.

   ```sql
   UPDATE NashvilleHousing
   SET SoldAsVacant = CASE 
       WHEN SoldAsVacant = 'Y' THEN 'Yes'
       WHEN SoldAsVacant = 'N' THEN 'No'
       ELSE SoldAsVacant
   END;
   ```

### 5. **Remove Duplicate Records**:
   Duplicate records are identified using a `ROW_NUMBER()` function, and then removed based on key fields like `ParcelID`, `PropertyAddress`, `SalePrice`, and `SaleDate`.

   ```sql
   WITH RowNumCTE AS (
       SELECT *,
       ROW_NUMBER() OVER (PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference ORDER BY UniqueID) row_num 
       FROM NashvilleHousing
   )
   DELETE FROM RowNumCTE
   WHERE row_num > 1;
   ```

### 6. **Delete Unused Columns**:
   To streamline the dataset, unnecessary columns such as `OwnerAddress`, `TaxDistrict`, and the original `SaleDate` are dropped.

   ```sql
   ALTER TABLE NashvilleHousing
   DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
   ```

## Tools & Technologies:
- **SQL**: For data transformation, cleaning, and management of the Nashville Housing dataset.
- **Microsoft SQL Server**: The platform used to run queries and manage the database.

## Insights & Benefits:
1. **Standardized Dates**: Ensures consistency in date formatting for time-based analysis.
2. **Accurate Address Data**: Populating missing addresses ensures that each record is complete and reliable.
3. **Improved Data Structure**: Splitting address fields into separate columns simplifies geographical analysis.
4. **Uniform Field Values**: Standardizing values like 'Yes' and 'No' provides clarity and consistency.
5. **Cleaned Dataset**: Removing duplicates and unnecessary columns results in a more efficient dataset for future analysis.

## Conclusion:
This project showcases the use of SQL for data cleaning and preparation, ensuring that the dataset is ready for more advanced analysis, reporting, or machine learning tasks. By cleaning the data, we can make better and more accurate decisions based on the Nashville Housing dataset.

## How to Access:
The SQL queries used for this project can be found in this repository. Feel free to explore, modify, or run the queries to replicate the data cleaning process.
