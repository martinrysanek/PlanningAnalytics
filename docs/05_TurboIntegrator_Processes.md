# TurboIntegrator Procesy - Planning Analytics

## Přehled

Tento dokument definuje všechny TurboIntegrator (TI) procesy pro import dat, údržbu dimenzí a datové transformace.

---

## Kategorie Procesů

### 1. Dimension Maintenance (Údržba Dimenzí)
- Vytváření a aktualizace dimenzí
- Správa hierarchií
- Správa atributů

### 2. Data Import (Import Dat)
- Import z SQL databáze
- Import skutečných dat (Actual)
- Import master dat

### 3. Data Transformation (Transformace Dat)
- Kalkulace a agregace
- Kopírování dat mezi verzemi
- Aplikace růstových faktorů

### 4. Maintenance (Údržba)
- Cleanup procesů
- Archivace dat
- Optimalizace kostek

---

## 1. DIMENSION MAINTENANCE PROCESSES

### 1.1 Process: `Dim.Time.Create`

**Účel:** Vytvoření časové dimenze s hierarchií měsíců, kvartálů a roků

**Typ:** Manual/Scheduled  
**Zdroj dat:** None (generováno programově)

**Parametry:**
- `pStartYear` (Numeric): Počáteční rok (např. 2025)
- `pEndYear` (Numeric): Koncový rok (např. 2030)

**Prolog:**
```tm1
# Vytvoření dimenze Time
DimensionCreate('Time');

# Vytvoření konsolidace Total Time
DimensionElementInsert('Time', '', 'Total_Time', 'C');

# Inicializace proměnných
vYear = pStartYear;
```

**Metadata:**
```tm1
# Iterace přes roky
WHILE(vYear <= pEndYear);
    
    # Vytvoření roku
    vYearElement = NUMBERTOSTRING(vYear);
    DimensionElementInsert('Time', '', vYearElement, 'C');
    DimensionElementComponentAdd('Time', 'Total_Time', vYearElement, 1);
    
    # Vytvoření kvartálů
    vQuarter = 1;
    WHILE(vQuarter <= 4);
        vQuarterElement = 'Q' | NUMBERTOSTRING(vQuarter) | ' ' | vYearElement;
        DimensionElementInsert('Time', '', vQuarterElement, 'C');
        DimensionElementComponentAdd('Time', vYearElement, vQuarterElement, 1);
        
        # Vytvoření měsíců
        vStartMonth = (vQuarter - 1) * 3 + 1;
        vMonth = vStartMonth;
        WHILE(vMonth < vStartMonth + 3);
            vMonthName = SUBST('JanFebMarAprMayJunJulAugSepOctNovDec', (vMonth - 1) * 3 + 1, 3);
            vMonthElement = vMonthName | ' ' | vYearElement;
            
            DimensionElementInsert('Time', '', vMonthElement, 'N');
            DimensionElementComponentAdd('Time', vQuarterElement, vMonthElement, 1);
            
            # Nastavení atributů
            AttrPutS(vMonthElement, 'Time', vMonthElement, 'Period_Code');
            AttrPutN(vYear, 'Time', vMonthElement, 'Year');
            AttrPutN(vQuarter, 'Time', vMonthElement, 'Quarter');
            AttrPutN(vMonth, 'Time', vMonthElement, 'Month');
            AttrPutS('Month', 'Time', vMonthElement, 'Period_Type');
            
            vMonth = vMonth + 1;
        END;
        
        vQuarter = vQuarter + 1;
    END;
    
    vYear = vYear + 1;
END;
```

**Epilog:**
```tm1
# Uložení dimenze
SaveDataAll;
```

---

### 1.2 Process: `Dim.Product.Update`

**Účel:** Aktualizace produktové dimenze z SQL databáze

**Typ:** Scheduled (denně)  
**Zdroj dat:** ODBC - SQL Database

**Parametry:**
- `pConnectionString` (String): ODBC connection string
- `pUpdateMode` (String): 'Full' nebo 'Incremental'

**Data Source:**
```sql
SELECT 
    Product_Code,
    Product_Name,
    Category,
    Subcategory,
    Active_Flag,
    Default_Price,
    Default_Cost,
    Supplier
FROM Products
WHERE Active_Flag = 'Y'
ORDER BY Category, Product_Code;
```

**Variables:**
```tm1
vProductCode = Product_Code;
vProductName = Product_Name;
vCategory = Category;
vSubcategory = Subcategory;
vActiveFlag = Active_Flag;
vDefaultPrice = Default_Price;
vDefaultCost = Default_Cost;
vSupplier = Supplier;
```

**Prolog:**
```tm1
# Vytvoření dimenze pokud neexistuje
IF(DimensionExists('Product') = 0);
    DimensionCreate('Product');
    DimensionElementInsert('Product', '', 'Total_Products', 'C');
ENDIF;

# Inicializace čítače
vRecordCount = 0;
```

**Metadata:**
```tm1
# Vytvoření kategorie pokud neexistuje
IF(DimensionElementExists('Product', vCategory) = 0);
    DimensionElementInsert('Product', '', vCategory, 'C');
    DimensionElementComponentAdd('Product', 'Total_Products', vCategory, 1);
ENDIF;

# Vytvoření subkategorie pokud neexistuje
IF(DimensionElementExists('Product', vSubcategory) = 0);
    DimensionElementInsert('Product', '', vSubcategory, 'C');
    DimensionElementComponentAdd('Product', vCategory, vSubcategory, 1);
ENDIF;

# Vytvoření nebo aktualizace produktu
IF(DimensionElementExists('Product', vProductCode) = 0);
    DimensionElementInsert('Product', '', vProductCode, 'N');
    DimensionElementComponentAdd('Product', vSubcategory, vProductCode, 1);
ENDIF;

# Nastavení atributů
AttrPutS(vProductCode, 'Product', vProductCode, 'Product_Code');
AttrPutS(vProductName, 'Product', vProductCode, 'Product_Name');
AttrPutS(vCategory, 'Product', vProductCode, 'Category');
AttrPutS(vActiveFlag, 'Product', vProductCode, 'Active_Flag');
AttrPutS(vSupplier, 'Product', vProductCode, 'Supplier');

# Aktualizace master dat
CellPutN(vDefaultPrice, 'Product_Master', vProductCode, 'Default_Price', 'Actual');
CellPutN(vDefaultCost, 'Product_Master', vProductCode, 'Default_Cost', 'Actual');

vRecordCount = vRecordCount + 1;
```

**Epilog:**
```tm1
# Log výsledků
ItemReject('Product dimension updated: ' | NUMBERTOSTRING(vRecordCount) | ' products');
SaveDataAll;
```

---

### 1.3 Process: `Dim.Channel.Update`

**Účel:** Aktualizace dimenze prodejních kanálů

**Typ:** Scheduled (týdně)  
**Zdroj dat:** ODBC - SQL Database

**Data Source:**
```sql
SELECT 
    Channel_Code,
    Channel_Name,
    Channel_Type,
    Default_Margin_Pct,
    Commission_Pct,
    Active_Flag
FROM Sales_Channels
WHERE Active_Flag = 'Y'
ORDER BY Channel_Type, Channel_Code;
```

**Metadata:**
```tm1
# Podobná logika jako Dim.Product.Update
# Vytvoření hierarchie podle Channel_Type
# Nastavení atributů a master dat
```

---

### 1.4 Process: `Dim.Division.Update`

**Účel:** Aktualizace dimenze divizí/obchodních jednotek

**Typ:** Manual (při změnách organizační struktury)  
**Zdroj dat:** ODBC - SQL Database

**Data Source:**
```sql
SELECT 
    Division_Code,
    Division_Name,
    Parent_Division,
    Manager,
    Cost_Center,
    Active_Flag
FROM Divisions
WHERE Active_Flag = 'Y'
ORDER BY Parent_Division, Division_Code;
```

---

## 2. DATA IMPORT PROCESSES

### 2.1 Process: `Import.Sales.Actual`

**Účel:** Import skutečných prodejních dat z ERP systému

**Typ:** Scheduled (denně v noci)  
**Zdroj dat:** ODBC - SQL Database

**Parametry:**
- `pStartDate` (String): Datum od (YYYY-MM-DD)
- `pEndDate` (String): Datum do (YYYY-MM-DD)
- `pConnectionString` (String): ODBC connection

**Data Source:**
```sql
SELECT 
    s.Transaction_Date,
    s.Product_Code,
    s.Channel_Code,
    s.Division_Code,
    SUM(s.Quantity) AS Total_Quantity,
    AVG(s.Unit_Price) AS Avg_Price,
    AVG(s.Unit_Cost) AS Avg_Cost,
    SUM(s.Sales_Amount) AS Total_Amount
FROM Sales_Transactions s
WHERE s.Transaction_Date BETWEEN ? AND ?
    AND s.Status = 'Completed'
GROUP BY 
    s.Transaction_Date,
    s.Product_Code,
    s.Channel_Code,
    s.Division_Code
ORDER BY s.Transaction_Date;
```

**Variables:**
```tm1
vTransactionDate = Transaction_Date;
vProductCode = Product_Code;
vChannelCode = Channel_Code;
vDivisionCode = Division_Code;
vQuantity = Total_Quantity;
vPrice = Avg_Price;
vCost = Avg_Cost;
vAmount = Total_Amount;
```

**Prolog:**
```tm1
# Inicializace
vRecordCount = 0;
vErrorCount = 0;
vStartTime = NOW;

# Vytvoření error log cube pokud neexistuje
IF(CubeExists('Import_Errors') = 0);
    CubeCreate('Import_Errors', 'Process', 'Error_Type', 'Time', 'Message');
ENDIF;
```

**Metadata:**
```tm1
# Konverze data na Time element
vYear = SUBST(vTransactionDate, 1, 4);
vMonth = SUBST(vTransactionDate, 6, 2);
vMonthName = SUBST('JanFebMarAprMayJunJulAugSepOctNovDec', 
                   (STRINGTONUMBER(vMonth) - 1) * 3 + 1, 3);
vTimeElement = vMonthName | ' ' | vYear;

# Validace existence elementů
vValid = 1;

IF(DimensionElementExists('Product', vProductCode) = 0);
    vValid = 0;
    vErrorMsg = 'Product not found: ' | vProductCode;
    CellPutS(vErrorMsg, 'Import_Errors', 'Import.Sales.Actual', 
             'Missing_Element', vTimeElement, vErrorMsg);
    vErrorCount = vErrorCount + 1;
ENDIF;

IF(DimensionElementExists('Channel', vChannelCode) = 0);
    vValid = 0;
    vErrorMsg = 'Channel not found: ' | vChannelCode;
    CellPutS(vErrorMsg, 'Import_Errors', 'Import.Sales.Actual', 
             'Missing_Element', vTimeElement, vErrorMsg);
    vErrorCount = vErrorCount + 1;
ENDIF;

IF(DimensionElementExists('Division', vDivisionCode) = 0);
    vValid = 0;
    vErrorMsg = 'Division not found: ' | vDivisionCode;
    CellPutS(vErrorMsg, 'Import_Errors', 'Import.Sales.Actual', 
             'Missing_Element', vTimeElement, vErrorMsg);
    vErrorCount = vErrorCount + 1;
ENDIF;
```

**Data:**
```tm1
# Import dat pouze pokud jsou všechny elementy validní
IF(vValid = 1);
    # Import Quantity
    CellPutN(vQuantity, 'Sales_PL', vProductCode, vChannelCode, vDivisionCode,
             vTimeElement, 'Actual', 'Product_Revenue', 'Quantity', 'CZK');
    
    # Import Price
    CellPutN(vPrice, 'Sales_PL', vProductCode, vChannelCode, vDivisionCode,
             vTimeElement, 'Actual', 'Product_Revenue', 'Price', 'CZK');
    
    # Import Cost
    CellPutN(vCost, 'Sales_PL', vProductCode, vChannelCode, vDivisionCode,
             vTimeElement, 'Actual', 'Product_COGS', 'Cost', 'CZK');
    
    # Amount se vypočítá pravidly
    
    vRecordCount = vRecordCount + 1;
ENDIF;
```

**Epilog:**
```tm1
# Výpočet doby běhu
vEndTime = NOW;
vDuration = vEndTime - vStartTime;

# Log výsledků
ItemReject('Import completed:');
ItemReject('  Records imported: ' | NUMBERTOSTRING(vRecordCount));
ItemReject('  Errors: ' | NUMBERTOSTRING(vErrorCount));
ItemReject('  Duration: ' | NUMBERTOSTRING(vDuration) | ' seconds');

# Uložení dat
SaveDataAll;

# Odeslání notifikace při chybách
IF(vErrorCount > 0);
    # Zde by byl kód pro odeslání emailu nebo notifikace
    ItemReject('WARNING: Import completed with errors!');
ENDIF;
```

---

### 2.2 Process: `Import.OPEX.Actual`

**Účel:** Import skutečných provozních nákladů

**Typ:** Scheduled (měsíčně)  
**Zdroj dat:** ODBC - SQL Database

**Data Source:**
```sql
SELECT 
    e.Posting_Date,
    e.Account_Code,
    e.Division_Code,
    e.Cost_Center,
    SUM(e.Amount) AS Total_Amount
FROM GL_Entries e
WHERE e.Posting_Date BETWEEN ? AND ?
    AND e.Account_Type = 'OPEX'
    AND e.Status = 'Posted'
GROUP BY 
    e.Posting_Date,
    e.Account_Code,
    e.Division_Code,
    e.Cost_Center
ORDER BY e.Posting_Date;
```

**Metadata/Data:**
```tm1
# Podobná logika jako Import.Sales.Actual
# Mapování GL účtů na Account dimenzi
# Import částek do správných buněk
```

---

### 2.3 Process: `Import.CAPEX.Actual`

**Účel:** Import skutečných investičních nákladů

**Typ:** Scheduled (měsíčně)  
**Zdroj dat:** ODBC - SQL Database

**Data Source:**
```sql
SELECT 
    i.Investment_Date,
    i.Asset_Category,
    i.Division_Code,
    i.Project_Code,
    SUM(i.Amount) AS Total_Amount,
    AVG(i.Useful_Life_Years) AS Avg_Life
FROM Investments i
WHERE i.Investment_Date BETWEEN ? AND ?
    AND i.Status = 'Approved'
GROUP BY 
    i.Investment_Date,
    i.Asset_Category,
    i.Division_Code,
    i.Project_Code
ORDER BY i.Investment_Date;
```

---

## 3. DATA TRANSFORMATION PROCESSES

### 3.1 Process: `Transform.CopyVersion`

**Účel:** Kopírování dat mezi verzemi (např. Actual → Budget jako základ)

**Typ:** Manual  
**Zdroj dat:** None (cube to cube)

**Parametry:**
- `pSourceVersion` (String): Zdrojová verze
- `pTargetVersion` (String): Cílová verze
- `pSourceYear` (String): Zdrojový rok
- `pTargetYear` (String): Cílový rok
- `pClearTarget` (Numeric): 1 = vymazat cíl před kopírováním

**Prolog:**
```tm1
# Validace parametrů
IF(DimensionElementExists('Version', pSourceVersion) = 0);
    ProcessQuit;
ENDIF;

IF(DimensionElementExists('Version', pTargetVersion) = 0);
    ProcessQuit;
ENDIF;

# Vymazání cílových dat pokud požadováno
IF(pClearTarget = 1);
    ViewZeroOut('Sales_PL', 'Copy_Target_View');
ENDIF;

vRecordCount = 0;
```

**Metadata:**
```tm1
# Vytvoření view pro iteraci
ViewCreate('Sales_PL', 'Copy_Source_View');

# Nastavení subsetů
SubsetCreate('Product', 'Copy_Products');
SubsetCreate('Channel', 'Copy_Channels');
SubsetCreate('Division', 'Copy_Divisions');
SubsetCreate('Time', 'Copy_Time');
SubsetCreate('Account', 'Copy_Accounts');
SubsetCreate('Measure', 'Copy_Measures');

# Přidání elementů do subsetů
# ... (kód pro vytvoření subsetů)

# Přiřazení subsetů k view
ViewSubsetAssign('Sales_PL', 'Copy_Source_View', 'Product', 'Copy_Products');
# ... (další dimenze)
```

**Data:**
```tm1
# Iterace přes view a kopírování dat
ViewExtractSkipCalcsSet('Sales_PL', 'Copy_Source_View', 1);
ViewExtractSkipZeroesSet('Sales_PL', 'Copy_Source_View', 1);

WHILE(ViewExtractNext('Sales_PL', 'Copy_Source_View') = 1);
    vProduct = CellGetS('Sales_PL', 'Copy_Source_View', 'Product');
    vChannel = CellGetS('Sales_PL', 'Copy_Source_View', 'Channel');
    vDivision = CellGetS('Sales_PL', 'Copy_Source_View', 'Division');
    vTime = CellGetS('Sales_PL', 'Copy_Source_View', 'Time');
    vAccount = CellGetS('Sales_PL', 'Copy_Source_View', 'Account');
    vMeasure = CellGetS('Sales_PL', 'Copy_Source_View', 'Measure');
    vCurrency = CellGetS('Sales_PL', 'Copy_Source_View', 'Currency');
    
    vValue = CellGetN('Sales_PL', vProduct, vChannel, vDivision, 
                      vTime, pSourceVersion, vAccount, vMeasure, vCurrency);
    
    # Mapování času mezi roky
    vTargetTime = SUBST(vTime, 1, 3) | ' ' | pTargetYear;
    
    # Zápis do cílové verze
    CellPutN(vValue, 'Sales_PL', vProduct, vChannel, vDivision,
             vTargetTime, pTargetVersion, vAccount, vMeasure, vCurrency);
    
    vRecordCount = vRecordCount + 1;
END;
```

**Epilog:**
```tm1
# Cleanup views a subsety
ViewDestroy('Sales_PL', 'Copy_Source_View');
SubsetDestroy('Product', 'Copy_Products');
# ... (další subsety)

# Log výsledků
ItemReject('Copy completed: ' | NUMBERTOSTRING(vRecordCount) | ' cells copied');
SaveDataAll;
```

---

### 3.2 Process: `Transform.ApplyGrowth`

**Účel:** Aplikace růstových faktorů na plánovaná data

**Typ:** Manual  
**Zdroj dat:** None

**Parametry:**
- `pVersion` (String): Verze pro aplikaci růstu
- `pYear` (String): Rok
- `pGrowthType` (String): 'Volume' nebo 'Price'
- `pGrowthRate` (Numeric): Růstový faktor v % (např. 5 = 5%)

**Prolog:**
```tm1
vGrowthFactor = 1 + (pGrowthRate / 100);
vRecordCount = 0;
```

**Metadata:**
```tm1
# Vytvoření view pro produkty a měsíce
# Aplikace růstu na Quantity nebo Price podle pGrowthType
```

**Data:**
```tm1
IF(pGrowthType @= 'Volume');
    vMeasure = 'Quantity';
ELSE;
    vMeasure = 'Price';
ENDIF;

# Iterace a aplikace růstu
# vNewValue = vOldValue * vGrowthFactor
```

---

### 3.3 Process: `Transform.CalculateForecast`

**Účel:** Výpočet Forecast verze (Actual YTD + Plan budoucí měsíce)

**Typ:** Scheduled (měsíčně)  
**Zdroj dat:** None

**Parametry:**
- `pCurrentMonth` (String): Aktuální měsíc (např. 'Jun 2025')

**Logic:**
```tm1
# Pro každý měsíc:
# - Pokud <= pCurrentMonth: kopíruj z Actual
# - Pokud > pCurrentMonth: kopíruj z Most_Likely
```

---

## 4. MAINTENANCE PROCESSES

### 4.1 Process: `Maint.ZeroOut.Cube`

**Účel:** Vymazání dat z kostky

**Typ:** Manual  
**Zdroj dat:** None

**Parametry:**
- `pCubeName` (String): Název kostky
- `pVersion` (String): Verze k vymazání
- `pYear` (String): Rok k vymazání

---

### 4.2 Process: `Maint.Archive.OldData`

**Účel:** Archivace starých dat

**Typ:** Scheduled (ročně)  
**Zdroj dat:** None

**Parametry:**
- `pArchiveYear` (String): Rok k archivaci
- `pArchivePath` (String): Cesta pro archiv

---

### 4.3 Process: `Maint.Optimize.Cubes`

**Účel:** Optimalizace kostek (restrukturalizace)

**Typ:** Scheduled (měsíčně v noci)  
**Zdroj dat:** None

**Logic:**
```tm1
# Pro každou kostku:
# - CubeProcessFeeders
# - CubeSaveData
# - Kontrola velikosti
```

---

## 5. UTILITY PROCESSES

### 5.1 Process: `Util.CreateViews`

**Účel:** Vytvoření standardních views pro uživatele

**Typ:** Manual (při setupu)

---

### 5.2 Process: `Util.SetupSecurity`

**Účel:** Nastavení bezpečnostních pravidel

**Typ:** Manual (při setupu)

---

### 5.3 Process: `Util.DataQualityCheck`

**Účel:** Kontrola kvality dat

**Typ:** Scheduled (denně)

**Checks:**
- Negativní revenue
- Nulové marže
- Chybějící data
- Outliers

---

## Process Execution Schedule

### Denní Procesy (2:00 AM):
1. `Import.Sales.Actual`
2. `Dim.Product.Update`
3. `Util.DataQualityCheck`

### Týdenní Procesy (Neděle 3:00 AM):
1. `Dim.Channel.Update`
2. `Maint.Optimize.Cubes`

### Měsíční Procesy (1. den měsíce 4:00 AM):
1. `Import.OPEX.Actual`
2. `Import.CAPEX.Actual`
3. `Transform.CalculateForecast`
4. `Maint.Archive.OldData` (pokud je konec roku)

### Manuální Procesy:
- `Dim.Time.Create` (při setupu)
- `Dim.Division.Update` (při změnách)
- `Transform.CopyVersion` (při plánování)
- `Transform.ApplyGrowth` (při plánování)
- `Maint.ZeroOut.Cube` (podle potřeby)

---

## Error Handling Strategy

### Všechny procesy by měly:
1. Validovat vstupní parametry
2. Logovat chyby do Import_Errors cube
3. Pokračovat při chybách (ne ProcessQuit)
4. Reportovat summary v Epilogu
5. Odesílat notifikace při kritických chybách

### Error Log Structure:
```
Import_Errors [Process, Error_Type, Time, Message]
```

---

## Performance Optimization

### Best Practices:
1. Používat ViewExtractSkipCalcs
2. Používat ViewExtractSkipZeroes
3. Minimalizovat DB funkce v loops
4. Používat bulk operations kde možné
5. Schedulovat náročné procesy v noci
6. Monitorovat execution times

---

## Testing Checklist

- [ ] Testovat každý proces samostatně
- [ ] Testovat s malým objemem dat
- [ ] Testovat s plným objemem dat
- [ ] Testovat error handling
- [ ] Testovat rollback scenarios
- [ ] Testovat performance
- [ ] Dokumentovat execution times
- [ ] Získat schválení od business users