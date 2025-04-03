# Data Cleaning in SQL: Nashville Housing Dataset
## Project Description:
Cleaned and transformed raw housing data using SQL for better analysis and reporting by standardizing date formats, filling missing address fields with self-joins, splitting address components using SUBSTRING and PARSENAME, replacing categorical values, removing 100+ duplicate records with CTE and ROW_NUMBER(), and dropping irrelevant columns to improve data quality and readiness for analysis

## Key Features
- ### Converted SaleDate to standard date format

      Select SaleDateConverted,
      CONVERT(date,saleDate)
      FROM [Nashville Housing ].dbo.NashvilleHousing

      Update NashvilleHousing
      SET SaleDate=CONVERT(date,saleDate)

      ALTER TABLE NashvilleHousing 
      Add SaleDateConverted Date;

      Update NashvilleHousing 
      SET SaleDateConverted=CONVERT(date,saleDate)

- ### Filled missing PropertyAddress using self-joins

I Join the table with itself and use isNull funtion to replace any Null in a.ProperyAddress with b.ProperyAddress Value

       Select 
       a.ParcelID
      ,a.PropertyAddress
      ,b.ParcelID
      ,b.PropertyAddress
      ,ISNULL(a.PropertyAddress,b.PropertyAddress)

       FROM [Nashville Housing ].dbo.NashvilleHousing  a
       Join [Nashville Housing ].dbo.NashvilleHousing  b
       on a.ParcelID=b.ParcelID
       and a.[UniqueID ]<>b.[UniqueID ]
       where a.PropertyAddress IS null

       Update a
       Set 
       a.PropertyAddress=ISNULL(a.PropertyAddress,b.    PropertyAddress)
      --Or you can write a string
      --a.PropertyAddress=ISNULL(a.PropertyAddress,'NO Address')

      FROM [Nashville Housing ].dbo.NashvilleHousing  a
      Join [Nashville Housing ].dbo.NashvilleHousing  b
      on a.ParcelID=b.ParcelID
      and a.[UniqueID ]<>b.[UniqueID ]
      where a.PropertyAddress IS null

- ### Split address fields (PropertyAddress, OwnerAddress) into separate columns (Street, City, State)

      Select PropertyAddress
      From [Nashville Housing ].dbo.NashvilleHousing
      --Where PropertyAdress in null
      --Order by ParcelID

       Select 
       SUBSTRING(PropertyAddress,1,CHARINDEX(',',PropertyAddress) -1) As Address
      --CHARINDEX is a number, and here its mean end with the    dilimeter which is the comma 
      --To remove the comma add -1

      SUBSTRING(PropertyAddress,CHARINDEX(',',PropertyAddress+1, Len(PropertAddress))As Address
      --Strat with the first charecter after the comma till the end
      FROM [Nashville Housing ].dbo.NashvilleHousing  

      ALTER Table NashvilleHousing
      ADD PropertySplitAddress Nvarchar(255);
      UPDATE NashvilleHousing
      SET PropertySplitAddress = SUBSTRING(PropertyAddress,1,CHARINDEX(',',PropertyAddress)-1)

      ALTER Table NashvilleHousing
      ADD PropertySplitCity Nvarchar(255);
      UPDATE NashvilleHousing
      SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))

      Select *
      From [Nashville Housing ].dbo.NashvilleHousing

 #### Another Option is PARSENAME FUNCTION that will Dedect the Dilimeter or the comma
       Select OwnerAddress
       From [Nashville Housing ].dbo.NashvilleHousing

       Select
       PARSENAME(Replace(OwnerAddress, ',' , '.'),3)
      ,PARSENAME(Replace(OwnerAddress, ',' , '.'),2)
      ,PARSENAME(Replace(OwnerAddress, ',' , '.'),1)

      From [Nashville Housing ].dbo.NashvilleHousing

      Alter Table NashvilleHousing
      ADD OwnerSplitAddress Nvarchar(225)
      Update NashvilleHousing
      Set OwnerSplitAddress = PARSENAME(Replace(OwnerAddress, ',' , '.'),3)

      Alter Table NashvilleHousing
      ADD OwnerSplitCity Nvarchar(225)
      Update NashvilleHousing
      Set OwnerSplitCity = PARSENAME(Replace(OwnerAddress, ',' , '.'),2)


      Alter Table NashvilleHousing
      ADD OwnerSplitState Nvarchar(225)
      Update NashvilleHousing
      Set OwnerSplitState = PARSENAME(Replace(OwnerAddress, ',' , '.'),1)


     Select *
     From [Nashville Housing ].dbo.NashvilleHousing

- ### Replaced Y/N values in SoldAsVacant with Yes/No
      Select
      Distinct(SoldAsVacant), COUNT(SoldAsVacant)
      From [Nashville Housing ].dbo.NashvilleHousing
      Group by SoldAsVacant
      Order by 2

      Select SoldAsVacant,
      Case when SoldAsVacant ='Y' Then 'Yes'
      when SoldAsVacant='N' then 'No'
      Else SoldAsVacant
      End
      From [Nashville Housing ].dbo.NashvilleHousing

      Update NashvilleHousing
      Set SoldAsVacant=Case when SoldAsVacant ='Y' Then 'Yes'
      when SoldAsVacant='N' then 'No'
      Else SoldAsVacant
      End

- ### Removed 104 duplicate rows using ROW_NUMBER() and CTE

       WITH RowNumCTE As (
       Select *, 
       ROW_NUMBER() Over (
	   Partition by
	               ParcelID
				  ,PropertyAddress
				  ,SalePrice
				  ,SaleDate,
				  LegalReference
				  Order by 
				          UniqueID
						  ) row_num
       From [Nashville Housing ].dbo.NashvilleHousing
      --order by ParcelID
      )

      Select *
      From RowNumCTE
      where row_num>1
      --Order by PropertyAddress
      --Peplace Select with Delete and deactive Order by to remove the 104 duplicate rows

- ### Dropped irrelevant columns to streamline the dataset

      Select *
      From [Nashville Housing ].dbo.NashvilleHousing

      Alter Table [Nashville Housing ].dbo.NashvilleHousing
      Drop COLUMN OwnerAddress, PropertyAddress, TaxDistrict

      Alter Table [Nashville Housing ].dbo.NashvilleHousing
      Drop COLUMN SaleDate

## Tools Used
- SQL Server
- SQL functions: CONVERT, ISNULL, SUBSTRING, PARSENAME, ROW_NUMBER(), CTE

