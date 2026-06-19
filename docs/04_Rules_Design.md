# Kalkulační Pravidla (Rules) - Planning Analytics

## Přehled

Tento dokument definuje všechna kalkulační pravidla pro kostku Sales_PL. Pravidla jsou organizována podle účtů (Account dimension) a typů kalkulací.

---

## Struktura Rules Souboru

### Soubor: `Sales_PL.rux`

```
# ============================================
# Sales_PL Cube Rules
# Purpose: P&L Calculations and Aggregations
# Author: Financial Planning Team
# Last Modified: 2025-01-01
# ============================================

SKIPCHECK;
FEEDSTRINGS;
```

---

## 1. Revenue Calculations (Výpočty Tržeb)

### 1.1 Product Revenue Amount

**Logika:** Revenue Amount = Quantity × Price

```rules
# Revenue Amount Calculation
['Product_Revenue', 'Amount'] = N:
    ['Product_Revenue', 'Quantity'] * 
    ['Product_Revenue', 'Price'];

# Feeder
['Product_Revenue', 'Quantity'] => ['Product_Revenue', 'Amount'];
['Product_Revenue', 'Price'] => ['Product_Revenue', 'Amount'];
```

### 1.2 Gross Revenue Consolidation

**Logika:** Gross_Revenue = Product_Revenue + Other_Revenue

```rules
# Gross Revenue Consolidation
['Gross_Revenue', 'Amount'] = N:
    ['Product_Revenue', 'Amount'] + 
    ['Other_Revenue', 'Amount'];

# Feeder
['Product_Revenue', 'Amount'] => ['Gross_Revenue', 'Amount'];
['Other_Revenue', 'Amount'] => ['Gross_Revenue', 'Amount'];
```

### 1.3 Net Revenue Calculation

**Logika:** Revenue = Gross_Revenue - Returns - Discounts

```rules
# Net Revenue Calculation
['Revenue', 'Amount'] = N:
    ['Gross_Revenue', 'Amount'] - 
    ['Returns', 'Amount'] - 
    ['Discounts', 'Amount'];

# Feeder
['Gross_Revenue', 'Amount'] => ['Revenue', 'Amount'];
['Returns', 'Amount'] => ['Revenue', 'Amount'];
['Discounts', 'Amount'] => ['Revenue', 'Amount'];
```

---

## 2. COGS Calculations (Výpočty Nákladů na Prodané Zboží)

### 2.1 Product COGS Amount

**Logika:** COGS Amount = Quantity × Cost

```rules
# Product COGS Calculation
['Product_COGS', 'Amount'] = N:
    ['Product_Revenue', 'Quantity'] * 
    ['Product_COGS', 'Cost'];

# Feeder
['Product_Revenue', 'Quantity'] => ['Product_COGS', 'Amount'];
['Product_COGS', 'Cost'] => ['Product_COGS', 'Amount'];
```

### 2.2 Total COGS Consolidation

**Logika:** COGS = Product_COGS + Freight_Costs + Warehousing_Costs

```rules
# Total COGS Consolidation
['COGS', 'Amount'] = N:
    ['Product_COGS', 'Amount'] + 
    ['Freight_Costs', 'Amount'] + 
    ['Warehousing_Costs', 'Amount'];

# Feeder
['Product_COGS', 'Amount'] => ['COGS', 'Amount'];
['Freight_Costs', 'Amount'] => ['COGS', 'Amount'];
['Warehousing_Costs', 'Amount'] => ['COGS', 'Amount'];
```

---

## 3. Margin Calculations (Výpočty Marží)

### 3.1 Gross Margin Amount

**Logika:** Gross_Margin = Revenue - COGS

```rules
# Gross Margin Calculation
['Gross_Margin', 'Amount'] = N:
    ['Revenue', 'Amount'] - 
    ['COGS', 'Amount'];

# Feeder
['Revenue', 'Amount'] => ['Gross_Margin', 'Amount'];
['COGS', 'Amount'] => ['Gross_Margin', 'Amount'];
```

### 3.2 Gross Margin Percentage

**Logika:** Margin % = (Gross_Margin / Revenue) × 100

```rules
# Gross Margin Percentage
['Gross_Margin', 'Margin_Pct'] = N:
    IF(['Revenue', 'Amount'] <> 0,
        (['Gross_Margin', 'Amount'] \ ['Revenue', 'Amount']) * 100,
        0
    );

# Feeder
['Gross_Margin', 'Amount'] => ['Gross_Margin', 'Margin_Pct'];
['Revenue', 'Amount'] => ['Gross_Margin', 'Margin_Pct'];
```

---

## 4. Operating Expenses Consolidation (Konsolidace Provozních Nákladů)

### 4.1 Personnel Costs

```rules
# Personnel Costs Consolidation
['Personnel_Costs', 'Amount'] = N:
    ['Salaries', 'Amount'] + 
    ['Benefits', 'Amount'] + 
    ['Training', 'Amount'];

# Feeder
['Salaries', 'Amount'] => ['Personnel_Costs', 'Amount'];
['Benefits', 'Amount'] => ['Personnel_Costs', 'Amount'];
['Training', 'Amount'] => ['Personnel_Costs', 'Amount'];
```

### 4.2 Facility Costs

```rules
# Facility Costs Consolidation
['Facility_Costs', 'Amount'] = N:
    ['Rent', 'Amount'] + 
    ['Utilities', 'Amount'] + 
    ['Maintenance', 'Amount'];

# Feeder
['Rent', 'Amount'] => ['Facility_Costs', 'Amount'];
['Utilities', 'Amount'] => ['Facility_Costs', 'Amount'];
['Maintenance', 'Amount'] => ['Facility_Costs', 'Amount'];
```

### 4.3 Marketing Costs

```rules
# Marketing Costs Consolidation
['Marketing_Costs', 'Amount'] = N:
    ['Advertising', 'Amount'] + 
    ['Promotions', 'Amount'] + 
    ['Digital_Marketing', 'Amount'];

# Feeder
['Advertising', 'Amount'] => ['Marketing_Costs', 'Amount'];
['Promotions', 'Amount'] => ['Marketing_Costs', 'Amount'];
['Digital_Marketing', 'Amount'] => ['Marketing_Costs', 'Amount'];
```

### 4.4 IT Costs

```rules
# IT Costs Consolidation
['IT_Costs', 'Amount'] = N:
    ['Software_Licenses', 'Amount'] + 
    ['IT_Services', 'Amount'] + 
    ['Hardware_Maintenance', 'Amount'];

# Feeder
['Software_Licenses', 'Amount'] => ['IT_Costs', 'Amount'];
['IT_Services', 'Amount'] => ['IT_Costs', 'Amount'];
['Hardware_Maintenance', 'Amount'] => ['IT_Costs', 'Amount'];
```

### 4.5 Other OPEX

```rules
# Other OPEX Consolidation
['Other_OPEX', 'Amount'] = N:
    ['Professional_Services', 'Amount'] + 
    ['Insurance', 'Amount'] + 
    ['Other_Expenses', 'Amount'];

# Feeder
['Professional_Services', 'Amount'] => ['Other_OPEX', 'Amount'];
['Insurance', 'Amount'] => ['Other_OPEX', 'Amount'];
['Other_Expenses', 'Amount'] => ['Other_OPEX', 'Amount'];
```

### 4.6 Total Operating Expenses

```rules
# Total Operating Expenses
['Operating_Expenses', 'Amount'] = N:
    ['Personnel_Costs', 'Amount'] + 
    ['Facility_Costs', 'Amount'] + 
    ['Marketing_Costs', 'Amount'] + 
    ['IT_Costs', 'Amount'] + 
    ['Other_OPEX', 'Amount'];

# Feeder
['Personnel_Costs', 'Amount'] => ['Operating_Expenses', 'Amount'];
['Facility_Costs', 'Amount'] => ['Operating_Expenses', 'Amount'];
['Marketing_Costs', 'Amount'] => ['Operating_Expenses', 'Amount'];
['IT_Costs', 'Amount'] => ['Operating_Expenses', 'Amount'];
['Other_OPEX', 'Amount'] => ['Operating_Expenses', 'Amount'];
```

---

## 5. EBITDA, EBIT, and Net Income (Ziskové Ukazatele)

### 5.1 EBITDA Calculation

**Logika:** EBITDA = Gross_Margin - Operating_Expenses

```rules
# EBITDA Calculation
['EBITDA', 'Amount'] = N:
    ['Gross_Margin', 'Amount'] - 
    ['Operating_Expenses', 'Amount'];

# Feeder
['Gross_Margin', 'Amount'] => ['EBITDA', 'Amount'];
['Operating_Expenses', 'Amount'] => ['EBITDA', 'Amount'];
```

### 5.2 EBITDA Margin Percentage

```rules
# EBITDA Margin Percentage
['EBITDA', 'Margin_Pct'] = N:
    IF(['Revenue', 'Amount'] <> 0,
        (['EBITDA', 'Amount'] \ ['Revenue', 'Amount']) * 100,
        0
    );

# Feeder
['EBITDA', 'Amount'] => ['EBITDA', 'Margin_Pct'];
['Revenue', 'Amount'] => ['EBITDA', 'Margin_Pct'];
```

### 5.3 EBIT Calculation

**Logika:** EBIT = EBITDA - Depreciation

```rules
# EBIT Calculation
['EBIT', 'Amount'] = N:
    ['EBITDA', 'Amount'] - 
    ['Depreciation', 'Amount'];

# Feeder
['EBITDA', 'Amount'] => ['EBIT', 'Amount'];
['Depreciation', 'Amount'] => ['EBIT', 'Amount'];
```

### 5.4 Net Income Calculation

**Logika:** Net_Income = EBIT (zjednodušeno, bez daní a úroků)

```rules
# Net Income Calculation
['Net_Income', 'Amount'] = N:
    ['EBIT', 'Amount'];

# Feeder
['EBIT', 'Amount'] => ['Net_Income', 'Amount'];
```

---

## 6. CAPEX Consolidation (Konsolidace Investičních Nákladů)

### 6.1 Total CAPEX

```rules
# Total CAPEX Consolidation
['CAPEX', 'Amount'] = N:
    ['IT_Infrastructure', 'Amount'] + 
    ['Store_Equipment', 'Amount'] + 
    ['Warehouse_Equipment', 'Amount'] + 
    ['Other_CAPEX', 'Amount'];

# Feeder
['IT_Infrastructure', 'Amount'] => ['CAPEX', 'Amount'];
['Store_Equipment', 'Amount'] => ['CAPEX', 'Amount'];
['Warehouse_Equipment', 'Amount'] => ['CAPEX', 'Amount'];
['Other_CAPEX', 'Amount'] => ['CAPEX', 'Amount'];
```

---

## 7. Time Aggregations (Časové Agregace)

### 7.1 Quarter Aggregation

**Logika:** Kvartální hodnoty = suma měsíčních hodnot

```rules
# Quarter Aggregation for Amount
# Automaticky řešeno hierarchií Time dimenze
# Žádné explicitní pravidlo není potřeba pro standardní agregaci

# Pro průměrné hodnoty (Price, Cost, Margin_Pct):
['Product_Revenue', 'Price'] = N:
    IF(ELISANC('Time', !Time, 'Quarter') = 1,
        # Vážený průměr podle množství
        DB('Sales_PL', !Product, !Channel, !Division, 
           ELPAR('Time', !Time, 1), !Version, 'Product_Revenue', 'Amount') \
        DB('Sales_PL', !Product, !Channel, !Division, 
           ELPAR('Time', !Time, 1), !Version, 'Product_Revenue', 'Quantity'),
        CONTINUE
    );
```

### 7.2 Year Aggregation

**Logika:** Roční hodnoty = suma kvartálních hodnot

```rules
# Year Aggregation
# Automaticky řešeno hierarchií Time dimenze
# Podobná logika jako u kvartálů pro průměrné hodnoty
```

---

## 8. Version-Specific Rules (Pravidla Specifická pro Verze)

### 8.1 Forecast Calculation

**Logika:** Forecast = Actual (YTD) + Plan (budoucí měsíce)

```rules
# Forecast Version Logic
[] = N:
    IF(!Version @= 'Forecast',
        # Pokud je období v minulosti, použij Actual
        IF(ATTRS('Time', !Time, 'Period_End_Date') <= TODAY,
            DB('Sales_PL', !Product, !Channel, !Division, !Time, 
               'Actual', !Account, !Measure, !Currency),
            # Jinak použij Most_Likely
            DB('Sales_PL', !Product, !Channel, !Division, !Time, 
               'Most_Likely', !Account, !Measure, !Currency)
        ),
        CONTINUE
    );

# Feeder
['Actual'] => ['Forecast'];
['Most_Likely'] => ['Forecast'];
```

---

## 9. Currency Conversion Rules (Pravidla pro Konverzi Měn)

### 9.1 Multi-Currency Support

```rules
# Currency Conversion
[] = N:
    IF(!Currency @<> 'CZK',
        # Konverze z CZK do cílové měny
        DB('Sales_PL', !Product, !Channel, !Division, !Time, 
           !Version, !Account, !Measure, 'CZK') *
        DB('FX_Rates', 'CZK', !Currency, !Time, 'Average'),
        CONTINUE
    );

# Feeder
['CZK'] => DB('', '', '', '', '', '', '', '', !Currency);
```

---

## 10. Channel Margin Rules (Pravidla pro Marže Kanálů)

### 10.1 Channel-Specific Pricing

**Logika:** Prodejní cena se liší podle kanálu

```rules
# Channel Margin Application
['Product_Revenue', 'Price'] = N:
    IF(DB('Sales_PL', !Product, !Channel, !Division, !Time, 
          !Version, 'Product_Revenue', 'Price', !Currency) = 0,
        # Pokud není zadána cena, použij default + channel margin
        DB('Product_Master', !Product, 'Default_Price', !Version) *
        (1 + DB('Channel_Master', !Channel, 'Default_Margin_Pct', !Version) \ 100),
        CONTINUE
    );

# Feeder
DB('Product_Master', !Product, 'Default_Price', !Version) => 
    ['Product_Revenue', 'Price'];
DB('Channel_Master', !Channel, 'Default_Margin_Pct', !Version) => 
    ['Product_Revenue', 'Price'];
```

---

## 11. Growth Rate Applications (Aplikace Růstových Faktorů)

### 11.1 Volume Growth

```rules
# Volume Growth Application
['Product_Revenue', 'Quantity'] = N:
    IF(!Version @= 'Budget',
        # Aplikuj růstový faktor z Assumptions
        DB('Sales_PL', !Product, !Channel, !Division, 
           ATTRS('Time', !Time, 'Prior_Year_Period'), 
           'Actual', 'Product_Revenue', 'Quantity', !Currency) *
        (1 + DB('Assumptions', 'Growth_Rates', 'Volume_Growth_Pct', 
                !Time, !Version) \ 100),
        CONTINUE
    );
```

### 11.2 Price Inflation

```rules
# Price Inflation Application
['Product_Revenue', 'Price'] = N:
    IF(!Version @= 'Budget',
        # Aplikuj cenovou inflaci
        DB('Sales_PL', !Product, !Channel, !Division, 
           ATTRS('Time', !Time, 'Prior_Year_Period'), 
           'Actual', 'Product_Revenue', 'Price', !Currency) *
        (1 + DB('Assumptions', 'Growth_Rates', 'Price_Inflation_Pct', 
                !Time, !Version) \ 100),
        CONTINUE
    );
```

---

## 12. Data Validation Rules (Validační Pravidla)

### 12.1 Negative Revenue Check

```rules
# Validation: Revenue cannot be negative
[] = N:
    IF(!Account @= 'Revenue',
        IF(CONTINUE < 0,
            # Log error to Data_Quality cube
            CELLPUTN(1, 'Data_Quality', 'Sales_PL', 
                     'Negative_Revenue', !Time, 'Error'),
            0
        ),
        CONTINUE
    );
```

### 12.2 Margin Threshold Check

```rules
# Validation: Gross Margin should be positive
[] = N:
    IF(!Account @= 'Gross_Margin',
        IF(CONTINUE < 0,
            # Log warning
            CELLPUTN(1, 'Data_Quality', 'Sales_PL', 
                     'Negative_Margin', !Time, 'Warning'),
            0
        ),
        CONTINUE
    );
```

---

## 13. Performance Optimization (Optimalizace Výkonu)

### 13.1 SKIPCHECK Usage

```rules
SKIPCHECK;
# Přeskočí kontrolu cirkulárních referencí
# Použít pouze pokud jsme si jisti, že nejsou přítomny
```

### 13.2 FEEDSTRINGS Usage

```rules
FEEDSTRINGS;
# Umožňuje feedování string hodnot
# Nutné pro některé pokročilé scénáře
```

### 13.3 Conditional Feeders

```rules
# Conditional Feeder Example
['Product_Revenue', 'Quantity'] => 
    IF(DB('Sales_PL', !Product, !Channel, !Division, !Time, 
          !Version, 'Product_Revenue', 'Quantity', !Currency) > 0,
        ['Product_Revenue', 'Amount']
    );
```

---

## 14. Rule Maintenance Guidelines (Pokyny pro Údržbu Pravidel)

### Best Practices:

1. **Komentáře:**
   - Každé pravidlo musí mít komentář vysvětlující logiku
   - Uvést datum poslední změny
   - Uvést autora změny

2. **Testování:**
   - Testovat každé pravidlo samostatně
   - Testovat interakce mezi pravidly
   - Testovat s různými objemy dat

3. **Verzování:**
   - Používat version control (Git)
   - Dokumentovat všechny změny
   - Udržovat changelog

4. **Performance:**
   - Pravidelně kontrolovat rule statistics
   - Optimalizovat pomalá pravidla
   - Minimalizovat DB funkce

5. **Dokumentace:**
   - Udržovat aktuální dokumentaci
   - Dokumentovat business logiku
   - Dokumentovat technické detaily

---

## 15. Rule Testing Scenarios (Testovací Scénáře)

### Test Case 1: Basic Revenue Calculation
```
Input:
- Quantity: 100
- Price: 1000 CZK

Expected Output:
- Amount: 100,000 CZK
```

### Test Case 2: Margin Calculation
```
Input:
- Revenue: 100,000 CZK
- COGS: 60,000 CZK

Expected Output:
- Gross Margin: 40,000 CZK
- Margin %: 40%
```

### Test Case 3: Time Aggregation
```
Input:
- Jan: 10,000 CZK
- Feb: 15,000 CZK
- Mar: 12,000 CZK

Expected Output:
- Q1: 37,000 CZK
```

### Test Case 4: Multi-Version Logic
```
Input:
- Actual (Jan-Jun): 600,000 CZK
- Most_Likely (Jul-Dec): 800,000 CZK

Expected Output:
- Forecast (Full Year): 1,400,000 CZK
```

---

## 16. Error Handling (Zpracování Chyb)

### Division by Zero

```rules
# Safe Division
['Gross_Margin', 'Margin_Pct'] = N:
    IF(['Revenue', 'Amount'] <> 0,
        (['Gross_Margin', 'Amount'] \ ['Revenue', 'Amount']) * 100,
        0  # Return 0 instead of error
    );
```

### Missing Data

```rules
# Handle Missing Data
[] = N:
    IF(CONTINUE = 0,
        # Try to get default value
        DB('Product_Master', !Product, 'Default_' | !Measure, !Version),
        CONTINUE
    );
```

---

## Implementační Checklist

- [ ] Vytvořit Sales_PL.rux soubor
- [ ] Implementovat revenue calculations
- [ ] Implementovat COGS calculations
- [ ] Implementovat margin calculations
- [ ] Implementovat OPEX consolidations
- [ ] Implementovat P&L calculations
- [ ] Implementovat time aggregations
- [ ] Implementovat version-specific logic
- [ ] Implementovat currency conversion
- [ ] Implementovat validation rules
- [ ] Otestovat všechna pravidla
- [ ] Optimalizovat feeders
- [ ] Dokumentovat všechny změny
- [ ] Provést performance testing
- [ ] Získat schválení od business users