# Program Copernicus - obliczanie NDVI

**Autor:** Gracjan Krzympiec / s102376

## Cel zadania

Obliczenie wskaźnika roślinności NDVI (*Normalized Difference Vegetation Index*) dla wybranego fragmentu terenu na podstawie zobrazowania satelitarnego Sentinel-2 oraz nałożenie wyniku na mapę bazową w programie QGIS.

## Dane

| Element | Wartość |
|---|---|
| Źródło danych | Copernicus (Sentinel-2) |
| Data zobrazowania | 2026-10-03 |
| Użyte kanały | B04 (czerwony), B08 (bliska podczerwień) |
| Obszar | Opole i okolice |
| Mapa bazowa | OpenStreetMap (kafle XYZ) |
| Układ współrzędnych | EPSG:4326 |
| Oprogramowanie | QGIS |

## Wzór

NDVI oblicza się z kanału bliskiej podczerwieni (NIR) i czerwonego (RED):

```
NDVI = (NIR - RED) / (NIR + RED)
```

Dla Sentinel-2: NIR = B08, RED = B04. Wartości mieszczą się w przedziale od -1 do 1.

## Przebieg pracy

1. Pobranie zobrazowania Sentinel-2 dla interesującego obszaru ze strony Copernicus.
2. Wczytanie kanałów B04 i B08 do QGIS (jako warstwy `BO4` i `BO8`).
3. Obliczenie NDVI: **Raster → Kalkulator rastra**, wyrażenie:

   ```
   ("BO8@1" - "BO4@1") / ("BO8@1" + "BO4@1")
   ```

   ![Kalkulator rastra](kalkulator.png)

4. Ustawienie symbologii warstwy `NDVI`: **Pasmo pojedyncze – pseudokolor**, paleta *green–yellow–red* (zielony = niskie NDVI, czerwony = wysokie).
5. Dodanie mapy bazowej OpenStreetMap i nałożenie warstwy NDVI na wybrany fragment mapy.

## Wynik

![Mapa NDVI](ndvi_mapa.png)

Zakres wartości na obliczonej warstwie: od **-0,92** do **0,45**.

## Interpretacja

| Kolor | NDVI | Co oznacza |
|---|---|---|
| Zielony | wysokie | gęsta, zdrowa roślinność (lasy, parki, łąki) |
| Żółty / pomarańczowy | średnie | roślinność rzadka, pola uprawne |
| Czerwony | niskie | gleba odsłonięta, zabudowa, drogi |
| Wartości ujemne | poniżej 0 | woda |

## Wnioski

- Najwyższe wartości NDVI występują w kompleksach leśnych i terenach zieleni, np. w parkach.
- Obszary zabudowane (centrum miasta) oraz drogi mają niskie NDVI.
- Dominacja odcieni czerwieni i pomarańczu wynika z terminu zobrazowania (październik): wiele pól jest już po zbiorach i pozostaje bez roślinności.
- Wskaźnik NDVI dobrze rozróżnia tereny zielone od zabudowanych i odsłoniętej gleby, więc nadaje się do monitorowania stanu roślinności.

## Zawartość paczki

```
├── README.md
├── ndvi_mapa.png
└── kalkulator.png
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Analiza wysokorozdzielczych danych CIR oraz obliczenie wskaźnika NDVI w programie QGIS

## 1. Cel zadania
Celem zadania było pobranie wysokorozdzielczej ortofotomapy w barwach umownych z wykorzystaniem bliskiej podczerwieni (**CIR – Color Infrared**) dla wybranego obszaru (Opole – okolice Ratusza), wyznaczenie znormalizowanego różnicowego wskaźnika wegetacji (**NDVI**), a następnie przeprowadzenie analizy statystycznej (statystyki strefowej) dla wybranych poligonów testowych reprezentujących różne formy pokrycia terenu.

---

## 2. Wykorzystane dane i narzędzia
* **Oprogramowanie:** QGIS
* **Źródło danych:** Krajowy Geoportal ([geoportal.gov.pl](https://www.geoportal.gov.pl/))
* **Wykorzystany arkusz ortofotomapy:** `M-33-48-D-d-3-4` (aktualność: 13.06.2025 r., kompozycja barwna: CIR)
* **Układ współrzędnych:** `EPSG:2180` (ETRF2000-PL / CS92)

### Charakterystyka kompozycji CIR
W wykorzystanej ortofotomapie CIR kanały spektralne przypisane są w następujący sposób:
* **Kanał 1:** Bliska podczerwień ($NIR$)
* **Kanał 2:** Pasmo czerwone ($Red$)
* **Kanał 3:** Pasmo zielone ($Green$)

Zdrowa roślinność charakteryzuje się silnym odbiciem promieniowania w zakresie bliskiej podczerwieni ($NIR$) oraz silną absorpcją w paśmie czerwonym ($Red$), dzięki czemu na kompozycji CIR odwzorowuje się w intensywnych odcieniach czerwieni.

---

## 3. Przebieg wykonania zadania (Krok po kroku)

### Krok 1: Pozyskanie danych z Geoportalu
1. W serwisie `mapy.geoportal.gov.pl` uruchomiono sekcję **Dane do pobrania** $\rightarrow$ **Ortofotomapa** $\rightarrow$ **Ortofotomapa wg rozdzielczości**.
2. Zlokalizowano obszar centrum Opola (Ratusz) i pobrano arkusz `M-33-48-D-d-3-4` z 2025 roku w kompozycji **CIR** (format GeoTIFF).
3. Pobrany raster wczytano do programu QGIS pod nazwą roboczą `NDVI_HIGH_RES`.

### Krok 2: Obliczenie wskaźnika NDVI
Wskaźnik wegetacji NDVI obliczono zgodnie ze wzorem:

$$NDVI = \frac{NIR - R}{NIR + R}$$

1. Uruchomiono narzędzie **Raster** $\rightarrow$ **Kalkulator rastrów...** (Raster Calculator).
2. Aby uniknąć utraty jakości (przeskalowania wyniku do rozdzielczości warstw satelitarnych Sentinel-2 obecnych w projekcie), zaznaczono na liście pasmo `NDVI_HIGH_RES@1` i użyto opcji **Zasięg z bieżącej warstwy** (Current layer extent), zachowując oryginalną, wysoką rozdzielczość przestrzenną ortofotomapy oraz układ `EPSG:2180`.
3. Wprowadzono formułę obliczeniową uwzględniającą kolejność kanałów w obrazie CIR:
   ```text
   ( "NDVI_HIGH_RES@1" - "NDVI_HIGH_RES@2" ) / ( "NDVI_HIGH_RES@1" + "NDVI_HIGH_RES@2" )
   ```
4. Wygenerowano jednokanałowy raster wynikowy o wartościach mieszczących się w przedziale od $-1$ do $1$.

### Krok 3: Utworzenie warstwy wektorowej (poligonów testowych)
1. Wybrano opcję **Warstwa** $\rightarrow$ **Twórz warstwę** $\rightarrow$ **Nowa warstwa tymczasowa...**
2. Utworzono warstwę poligonową w układzie `EPSG:2180`.
3. Za pomocą narzędzia **Dodaj poligon** zwektoryzowano kilka reprezentatywnych obszarów testowych o różnym pokryciu terenu (np. korony drzew / zieleń miejska, dachy budynków / nawierzchnia rynku, woda).

Poniższy zrzut ekranu przedstawia obliczony wskaźnik `NDVI_HIGH_RES` (w palecie barw od czerwonego do zielonego, zakres wartości od -0,93 do 0,84) nałożony na podkład OpenStreetMap (układ EPSG:2180) wraz z utworzoną warstwą wektorową `Dzialka` reprezentującą badany obszar:

![Mapa NDVI W HD Z DZIAŁKAMI](NDVI_HD1.png)

### Krok 4: Obliczenie statystyk strefowych (Zonal Statistics)
1. W panelu **Algorytmy** (Processing Toolbox) uruchomiono narzędzie **Statystyki strefowe** (Zonal Statistics).
2. Jako warstwę wektorową wskazano utworzone poligony testowe, natomiast jako warstwę rastrową – wyliczony wcześniej wysokorozdzielczy wskaźnik NDVI.
3. Wybrano do obliczenia podstawowe miary statystyczne: **średnia** (`mean`), **mediana** (`median`), **minimum** (`min`) oraz **maksimum** (`max`).
4. Wyniki odczytano w tabeli atrybutów nowo powstałej warstwy wektorowej.
