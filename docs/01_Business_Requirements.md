# Business Requirements - Finanční Plánování Elektronika

## Přehled Projektu

Aplikace pro finanční plánování a forecast pro společnost zabývající se prodejem elektroniky v IBM Planning Analytics.

## Cíl Projektu

Vytvoření kompletní aplikace pro:
- Finanční plánování a forecasting
- Profit & Loss analýzu
- Modelování různých scénářů
- Sledování skutečných vs. plánovaných výsledků

## Business Požadavky

### 1. Produktové Portfolio

**Kategorie produktů:**
- Mobilní telefony
- Počítače
- Tablety
- Notebooky
- Příslušenství

**Metriky pro každý produkt:**
- Objem prodeje (množství)
- Průměrná prodejní cena
- Pořizovací cena (nákladová cena)

### 2. Prodejní Kanály

**Typy kanálů:**
1. Přímý prodej na prodejně
2. Prodej prostřednictvím partnerů
3. Online prodej na Internetu

**Charakteristika:**
- Každý kanál má vlastní průměrnou prodejní marži
- Různé produkty mohou mít různou marži v různých kanálech

### 3. Organizační Struktura

- Divize / obchodní jednotky
- Každá divize může prodávat různé produkty přes různé kanály

### 4. Finanční Dimenze

**Nákladové kategorie:**
- **OPEX (Provozní náklady):**
  - Mzdy a platy
  - Nájemné
  - Marketing a reklama
  - IT náklady
  - Ostatní provozní náklady
  
- **CAPEX (Investiční náklady):**
  - Investice do vybavení
  - IT infrastruktura
  - Renovace prodejen

### 5. Časová Dimenze

**Granularita:**
- Základní úroveň: Měsíce
- Konsolidace: Kvartály
- Konsolidace: Roky

**Časové období:**
- 2025 - 2030 (6 let)
- Celkem 72 měsíců

### 6. Scénáře Plánování

**Verze dat:**
1. **Actual** - Skutečná data z ERP systému
2. **Best Case** - Optimistický scénář
3. **Most Likely** - Nejpravděpodobnější scénář
4. **Worst Case** - Pesimistický scénář

### 7. Výstupní Reporty

**Hlavní výstup: Profit & Loss Statement**

Struktura P&L:
```
Tržby (Revenue)
  = Objem prodeje × Prodejní cena
  
Náklady na prodané zboží (COGS)
  = Objem prodeje × Pořizovací cena
  
Hrubá marže (Gross Margin)
  = Tržby - COGS
  
Provozní náklady (OPEX)
  
EBITDA
  = Hrubá marže - OPEX
  
Odpisy (Depreciation)
  
EBIT
  = EBITDA - Odpisy
  
Investiční náklady (CAPEX)
  (sledováno samostatně)
```

### 8. Integrace Dat

**Zdroj dat:**
- SQL databáze (ERP systém společnosti)

**Metoda připojení:**
- ODBC/JDBC připojení

**Typ dat k importu:**
- Skutečné objemy prodeje
- Aktuální ceny (prodejní i pořizovací)
- Skutečné náklady (OPEX, CAPEX)
- Marže prodejních kanálů

### 9. Funkční Požadavky

**Plánování:**
- Možnost zadávat plánované hodnoty na úrovni měsíců
- Automatické agregace do kvartálů a roků
- Možnost kopírování dat mezi scénáři
- Možnost aplikace růstových faktorů

**Kalkulace:**
- Automatický výpočet tržeb z objemu a ceny
- Automatický výpočet COGS
- Výpočet marží a ziskovosti
- Konsolidace napříč dimenzemi

**Reporting:**
- P&L report s drill-down možnostmi
- Porovnání scénářů
- Variance analýza (Actual vs. Plan)
- Trend analýza

### 10. Nefunkční Požadavky

**Výkon:**
- Rychlá odezva při zadávání dat (< 2 sekundy)
- Efektivní agregace velkých objemů dat

**Bezpečnost:**
- Řízení přístupu podle rolí
- Oddělení práv pro čtení a zápis
- Audit trail změn

**Údržba:**
- Snadná aktualizace dimenzí
- Flexibilní přidávání nových produktů/kanálů
- Verzování pravidel a procesů

## Klíčoví Uživatelé

1. **Finanční kontroling** - hlavní uživatelé, zadávání plánů
2. **Management** - čtení reportů, analýzy
3. **IT administrátoři** - správa systému, import dat

## Úspěšnostní Kritéria

1. Kompletní P&L analýza dostupná do 5 minut od importu dat
2. Možnost vytvořit nový scénář do 10 minut
3. Všechny kalkulace automatizované bez manuálních zásahů
4. Flexibilní přidávání nových produktů bez změny struktury
5. Snadné porovnání scénářů a variance analýza