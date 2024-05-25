# Nashville-Housing-Data-Cleaning-EDA (using SQL)

## SQL Query

#-- Check Data (Top 10):

SELECT TOP (10) *
FROM NashvilleHousing

-- Check the Structure of the Table:

SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'NashvilleHousing'

-- Count Data:

SELECT COUNT (*)
FROM NashvilleHousing

-- Check for Missing Values:

SELECT 

    SUM(CASE WHEN UniqueID  IS NULL THEN 1 ELSE 0 END) AS missing_UniqueID,
    SUM(CASE WHEN ParcelID IS NULL THEN 1 ELSE 0 END) AS missing_ParcelID ,
    SUM(CASE WHEN PropertyAddress IS NULL THEN 1 ELSE 0 END) AS missing_PropertyAddress,
	SUM(CASE WHEN SaleDate IS NULL THEN 1 ELSE 0 END) AS missing_SaleDate,
	SUM(CASE WHEN LegalReference IS NULL THEN 1 ELSE 0 END) AS missing_LegalReference,
	SUM(CASE WHEN OwnerName IS NULL THEN 1 ELSE 0 END) AS missing_OwnerName,
	SUM(CASE WHEN OwnerAddress IS NULL THEN 1 ELSE 0 END) AS missing_OwnerAddress,
	SUM(CASE WHEN SoldAsVacant IS NULL THEN 1 ELSE 0 END) AS missing_SoldAsVacant,
	SUM(CASE WHEN SalePrice IS NULL THEN 1 ELSE 0 END) AS missing_SalePrice,
	SUM(CASE WHEN BuildingValue IS NULL THEN 1 ELSE 0 END) AS missing_BuildingValue,
	SUM(CASE WHEN LandValue IS NULL THEN 1 ELSE 0 END) AS missing_LandValue,
	SUM(CASE WHEN TotalValue IS NULL THEN 1 ELSE 0 END) AS missing_TotalValue,
	SUM(CASE WHEN YearBuilt IS NULL THEN 1 ELSE 0 END) AS missing_YearBuilt,
	SUM(CASE WHEN Acreage IS NULL THEN 1 ELSE 0 END) AS missing_Acreage
 
FROM NashvilleHousing

-- Standardize Date Format:

SELECT SaleDate2, CONVERT (date, SaleDate)
FROM NashvilleHousing

UPDATE NashvilleHousing
SET SaleDate = CONVERT(DATE, SaleDate)

-- If it doesn't Update properly try this:

ALTER TABLE NashvilleHousing
ADD SaleDate2 Date

UPDATE NashvilleHousing
SET SaleDate2 = CONVERT (date, SaleDate)

SELECT SaleDate2, CONVERT (date, SaleDate)
FROM NashvilleHousing

-- Populate Missing in Property Address - Using ParcelID

Select *
From NashvilleHousing
Where PropertyAddress is null

-- Use Self Join and ISNULL (use to replace the null with another value)

SELECT NH1.ParcelID, NH1.PropertyAddress, NH2.ParcelID, NH2.PropertyAddress, ISNULL(NH1.PropertyAddress,NH2.PropertyAddress)
From NashvilleHousing NH1
JOIN NashvilleHousing NH2
	on NH1.ParcelID = NH2.ParcelID
	AND NH1.[UniqueID ] <> NH2.[UniqueID ]
Where NH1.PropertyAddress is null


Update NH1

SET PropertyAddress = ISNULL(NH1.PropertyAddress,NH2.PropertyAddress)
From NashvilleHousing NH1
JOIN NashvilleHousing NH2
	on NH1.ParcelID = NH2.ParcelID
	AND NH1.[UniqueID ] <> NH2.[UniqueID ]
Where NH1.PropertyAddress is null


-- Breaking out Address into Individual Columns (Address, City, State)

SELECT PropertyAddress
From NashvilleHousing

SELECT

SUBSTRING(PropertyAddress, 1, CHARINDEX(',' , PropertyAddress)-1) AS P_Address,
SUBSTRING(PropertyAddress, CHARINDEX(',' , PropertyAddress) +1, LEN(PropertyAddress)) AS P_City

From NashvilleHousing

ALTER TABLE NashvilleHousing
ADD Property_Address Nvarchar(255)

UPDATE NashvilleHousing
SET Property_Address = SUBSTRING(PropertyAddress, 1, CHARINDEX(',' , PropertyAddress)-1)

ALTER TABLE NashvilleHousing
ADD Property_City Nvarchar(255)

UPDATE NashvilleHousing
SET Property_City = SUBSTRING(PropertyAddress, CHARINDEX(',' , PropertyAddress) +1, LEN(PropertyAddress))


-- Owner Address
Select OwnerAddress
From NashvilleHousing
Where OwnerAddress is null

SELECT NH1.ParcelID, NH1.OwnerAddress, NH2.ParcelID, NH2.OwnerAddress
From NashvilleHousing NH1
JOIN NashvilleHousing NH2
	on NH1.ParcelID = NH2.ParcelID
	AND NH1.[UniqueID ] <> NH2.[UniqueID ]
Where NH1.OwnerAddress is null

-- Split Owner Address
SELECT OwnerAddress
From NashvilleHousing

SELECT
PARSENAME(OwnerAddress, 1) -- Nothing will happen because it only works with period
From NashvilleHousing

SELECT

PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2) ,
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1) 

From NashvilleHousing

-- add column

ALTER TABLE NashvilleHousing
ADD Owner_Address Nvarchar(255)

UPDATE NashvilleHousing
SET Owner_Address = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)

ALTER TABLE NashvilleHousing
ADD Owner_City Nvarchar(255)

UPDATE NashvilleHousing
SET Owner_City = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)

ALTER TABLE NashvilleHousing
ADD Owner_State Nvarchar(255)

UPDATE NashvilleHousing
SET Owner_State = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)


-- Change Y and N to Yes and No in "Sold as Vacant" column:

SELECT DISTINCT (SoldAsVacant), COUNT (SoldAsVacant)
From NashvilleHousing
GROUP BY SoldAsVacant
-- ORDER BY 2

SELECT SoldAsVacant,

CASE WHEN  SoldAsVacant = 'Y' THEN 'Yes'
	 WHEN  SoldAsVacant = 'N' THEN 'No'
	 ELSE  SoldAsVacant
	 END

From NashvilleHousing

UPDATE NashvilleHousing

SET SoldAsVacant = 

CASE
	 WHEN  SoldAsVacant = 'Y' THEN 'Yes'
	 WHEN  SoldAsVacant = 'N' THEN 'No'
	 ELSE  SoldAsVacant
END

-- Remove Duplicates - Use CTE:

WITH RowNumCTE AS (
SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num
From NashvilleHousing
--ORDER BY ParcelID
)
SELECT *
From RowNumCTE
WHERE row_num > 1
ORDER BY PropertyAddress


WITH RowNumCTE AS (

SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num

From NashvilleHousing

)
DELETE
From RowNumCTE
WHERE row_num > 1

-- Delete Unused Columns:

ALTER TABLE NashvilleHousing
DROP COLUMN PropertyAddress, SaleDate, OwnerAddress, TaxDistrict


-- Get Summary Statistics for Numeric Values:

SELECT 

    AVG(SalePrice) AS avg_SalePrice,
    MIN(SalePrice) AS min_SalePrice,
    MAX(SalePrice) AS max_SalePrice,
    STDEV(SalePrice) AS stdev_SalePrice,
    AVG(Acreage) AS avg_Acreage,
    MIN(Acreage) AS min_Acreage,
    MAX(Acreage) AS max_Acreage,
    STDEV(Acreage) AS stdev_Acreage,
    AVG(LandValue) AS avg_LandValue,
    MIN(LandValue) AS min_LandValue,
    MAX(LandValue) AS max_LandValue,
    STDEV(LandValue) AS stdev_LandValue,
    AVG(BuildingValue) AS avg_BuildingValue,
    MIN(BuildingValue) AS min_BuildingValue,
    MAX(BuildingValue) AS max_BuildingValue,
    STDEV(BuildingValue) AS stdev_BuildingValue,
    AVG(TotalValue) AS avg_TotalValue,
    MIN(TotalValue) AS min_TotalValue,
    MAX(TotalValue) AS max_TotalValue,
    STDEV(TotalValue) AS stdev_TotalValue,
    AVG(Bedrooms) AS avg_Bedrooms,
    MIN(Bedrooms) AS min_Bedrooms,
    MAX(Bedrooms) AS max_Bedrooms,
    STDEV(Bedrooms) AS stdev_Bedrooms,
    AVG(FullBath) AS avg_FullBath,
    MIN(FullBath) AS min_FullBath,
    MAX(FullBath) AS max_FullBath,
    STDEV(FullBath) AS stdev_FullBath,
    AVG(HalfBath) AS avg_HalfBath,
    MIN(HalfBath) AS min_HalfBath,
    MAX(HalfBath) AS max_HalfBath,
    STDEV(HalfBath) AS stdev_HalfBath

FROM NashvilleHousing

-- Count Unique Values in Categorical columns:

SELECT LandUse, COUNT (*) AS CountLandUse
FROM NashvilleHousing
GROUP BY LandUse
ORDER BY CountLandUse DESC

SELECT Property_City, COUNT (*) AS CountPropertyCity
FROM NashvilleHousing
GROUP BY Property_City
ORDER BY CountPropertyCity DESC

SELECT Owner_City, COUNT (*) AS CountOwnerCity
FROM NashvilleHousing
GROUP BY Owner_City
ORDER BY CountOwnerCity DESC

SELECT Owner_State, COUNT (*) AS CountOwnerState
FROM NashvilleHousing
GROUP BY Owner_State
ORDER BY CountOwnerState DESC

-- Average Sales Price per City:

SELECT Property_City, AVG (SalePrice) AS avg_SalePrice_City
FROM NashvilleHousing
GROUP BY Property_City
ORDER BY avg_SalePrice_City DESC

SELECT LandUse, AVG (SalePrice) AS avg_SalePrice_LandUse
FROM NashvilleHousing
GROUP BY LandUse
ORDER BY avg_SalePrice_LandUse DESC

-- Distribution of property types by city:

SELECT Property_City, LandUse, COUNT (*) AS frequency
FROM NashvilleHousing
GROUP BY Property_City, LandUse
ORDER BY Property_City, frequency DESC

SELECT Property_City, LandUse, AVG (SalePrice) AS avg_SalePrice
FROM NashvilleHousing
GROUP BY Property_City, LandUse
ORDER BY Property_City, avg_SalePrice DESC



