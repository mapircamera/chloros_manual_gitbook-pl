# Warstwy obrazu

Menu rozwijane „Warstwy obrazu” w przeglądarce obrazów Chloros umożliwia szybkie przełączanie się między różnymi wersjami tego samego obrazu – od oryginalnych ujęć po przetworzone wyniki odbicia oraz obliczone obrazy indeksowe.

## Czym są warstwy obrazu?

W programie Chloros **warstwy** odnoszą się do różnych wyników obrazu dostępnych dla jednego obrazu źródłowego. Podczas przetwarzania obrazów program Chloros tworzy wiele wersji:

* **Oryginalne obrazy** (pliki JPG i RAW z aparatu)
* Wyniki **skalibrowane pod kątem odbicia** (jeśli kalibracja odbicia była włączona)
* **Obrazy docelowe** (jeśli obraz zawiera cele kalibracyjne)
* **Obrazy indeksowe** (NDVI, NDRE, GNDVI itp., jeśli skonfigurowano indeksy)**Rozwijane menu wyboru warstw** w prawym górnym rogu przeglądarki obrazów pozwala na natychmiastowe przełączanie się między tymi wersjami bez opuszczania przeglądarki.***

## Dostępne typy warstw

### JPG

* Oryginalny obraz podglądu JPG z aparatu
* Zawsze dostępny dla wszystkich obrazów
* Nieprzetworzony, taki jak uchwycony przez aparat
* Najszybszy do załadowania i wyświetlenia

**Kiedy wyświetlać:**

* Szybki podgląd oryginalnego zdjęcia
* Sprawdzanie kompozycji i kadrowania zdjęcia
* Weryfikacja jakości zdjęcia przed przetworzeniem

### RAW (oryginalny)

* Oryginalne dane z matrycy RAW z aparatu
* Bez odszumiania i bez zastosowania przetwarzania końcowego
* Wyższa głębia bitowa niż w przypadku JPG (zazwyczaj dane z matrycy 12-bitowe lub 14-bitowe)

**Kiedy przeglądać:**

* Sprawdzanie jakości oryginalnych danych z matrycy
* Sprawdzanie pod kątem problemów z matrycą lub artefaktów
* Porównanie wyników przed i po przetworzeniu

### RAW (Cel)

* Wyświetla się tylko w przypadku zdjęć zidentyfikowanych jako zawierające cele kalibracyjne
* Pokazuje oryginalny obraz RAW z wykrytym celem
* Służy do weryfikacji, czy wykrycie celu zakończyło się sukcesem

**Kiedy wyświetlać:**

* Potwierdzanie, że cele kalibracyjne zostały wykryte poprawnie
* Sprawdzanie jakości obrazu celu
* Rozwiązywanie problemów z kalibracją

{% hint style="info" %}
**Warstwa celu**: Ta warstwa pojawia się w menu rozwijanym tylko w przypadku obrazów zawierających cele kalibracyjne. Zwykłe zdjęcia nie będą miały tej opcji.
{% endhint %}

### RAW (Odbicie)

* Skalibrowany obraz wyjściowy odbicia
* Skorygowany pod kątem winietowania (jeśli włączono podczas przetwarzania)
* Odbicie skalibrowane przy użyciu danych celów (jeśli włączono)
* Wielopasmowy TIFF ze wszystkimi kanałami kamery
* Wartości pikseli reprezentują procent odbicia (przy użyciu trybu procentowego)
* Gotowy do obróbki za pomocą [Index/LUT Sandbox](index-lut-sandbox.md)

**Kiedy przeglądać:**

* Sprawdzanie skalibrowanych wyników
* Weryfikacja jakości kalibracji
* Sprawdzanie wartości pikseli pod kątem dokładności naukowej
* Porównanie z oryginałem w celu oceny efektów kalibracji

{% hint style="success" %}
**Zalecane**: Użyj warstwy RAW (Odbicie) podczas sprawdzania wartości pikseli do pomiarów naukowych i analizy.
{% endhint %}

### RAW (Indeks NDVI)... i podobne

* Obliczony obraz indeksu wegetacyjnego (NDVI w tym przykładzie)
* Nazwa wskaźnika zmienia się w zależności od tego, który wskaźnik został skonfigurowany podczas przetwarzania
* Przykłady: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) itp.
* Obraz w skali szarości w jednym paśmie przedstawiający wyniki obliczeń indeksu
* Dla każdego indeksu skonfigurowanego w ustawieniach projektu pojawia się jedna warstwa

**Możliwe nazwy indeksów:**

* RAW (indeks NDVI)
* RAW (indeks NDRE)
* RAW (indeks GNDVI)
* RAW (indeks OSAVI)
* RAW (indeks EVI)
* RAW (indeks SAVI)
* I wiele innych... (zobacz [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md))

**Kiedy wyświetlać:**

* Analiza wyników obliczeń indeksu
* Sprawdzanie zakresów wartości indeksu
* Identyfikacja obszarów zainteresowania
* Weryfikacja obrazów indeksowych przed użyciem w systemie GIS lub analizie

***

## Korzystanie z selektora warstw

### Otwieranie menu rozwijanego

1. Otwórz obraz w trybie pełnoekranowym (kliknij dowolną miniaturę w przeglądarce obrazów)
2. Znajdź **menu rozwijane warstw** w prawym górnym rogu przeglądarki
3. Menu rozwijane pokazuje aktualnie wybraną warstwę (np. „JPG”)
4. Kliknij menu rozwijane, aby wyświetlić wszystkie dostępne warstwy

### Przełączanie warstw

1. Kliknij menu rozwijane warstw, aby otworzyć listę
2. Wyświetlą się wszystkie warstwy dostępne dla bieżącego obrazu
3. Kliknij nazwę dowolnej warstwy, aby przełączyć się na tę wersję
4. Obraz natychmiast się zaktualizuje, aby wyświetlić wybraną warstwę

**Szybkie przełączanie:**

* Menu rozwijane zapamiętuje ostatni wybór
* Podczas przechodzenia do następnego obrazu, Chloros próbuje wyświetlić ten sam typ warstwy
* Jeśli ta warstwa nie istnieje na następnym obrazie, domyślnie wyświetlana jest warstwa JPG

### Dostępność warstw

Nie wszystkie warstwy są dostępne dla każdego obrazu:

**Zawsze dostępne:*** ✅ JPG (każdy obraz ma podgląd JPG)

**Dostępne warunkowo:**

* ⚠️ RAW (oryginalny) – tylko jeśli obraz został zarejestrowany w trybie RAW lub RAW+JPG
* ⚠️ RAW (cel) – tylko jeśli obraz zawiera wykryte cele kalibracyjne
* ⚠️ RAW (Odbicie) – Tylko po przetworzeniu z włączoną kalibracją odbicia
* ⚠️ RAW (\[Indeks] Indeks) – Tylko po przetworzeniu z skonfigurowanymi indeksami

***

## Trwałość warstw

### Przechodzenie między obrazami

Podczas przechodzenia do innego obrazu (za pomocą klawiszy strzałek lub kliknięcia miniatur):**Preferencje warstw są zachowywane:**

* Jeśli wyświetlane jest „RAW (Odbicie)”, następny obraz pokazuje „RAW (Odbicie)” (jeśli jest dostępny)
* Jeśli wyświetlane jest „RAW (NDVI Indeks)”, następny obraz pokazuje „RAW (NDVI Indeks)” (jeśli jest dostępny)
* Jeśli ta sama warstwa nie istnieje, domyślnie wyświetlany jest format JPG

**Przykładowy przebieg pracy:**

1. Otwórz obraz 1, przełącz na RAW (NDVI Index)
2. Naciśnij →, aby wyświetlić obraz 2
3. Obraz 2 automatycznie wyświetla warstwę RAW (NDVI Index)
4. Kontynuuj przeglądanie – wszystkie obrazy wyświetlają warstwę NDVI
5. Bardzo wydajne rozwiązanie do przeglądania wyników indeksowania wielu obrazów

***

## Typowe procedury

### Procedura 1: Porównanie przed i po

**Cel**: Porównanie obrazu oryginalnego z obrazem skalibrowanym

1. Otwórz przetworzony obraz w przeglądarce obrazów
2. Wybierz **RAW (Oryginalny)** z menu rozwijanego
3. Zwróć uwagę na winietowanie i wartości nieskalibrowane
4. Przełącz na **RAW (Odbicie)** z menu rozwijanego
5. Porównaj – winietowanie usunięte, wartości skalibrowane

### Procedura 2: Przegląd indeksu

**Cel**: Szybki przegląd wyników NDVI w całym zbiorze danych

1. Otwórz pierwszy przetworzony obraz
2. Wybierz **RAW (NDVI Index)** z menu rozwijanego
3. Użyj klawisza strzałki →, aby przejść do następnego obrazu
4. Warstwa NDVI pozostaje aktywna automatycznie
5. Przejrzyj wszystkie obrazy, sprawdzając wzorce NDVI
6. Przejdź do **RAW (NDRE Index)** w celu porównania

### Przebieg pracy 3: Weryfikacja celów

**Cel**: Sprawdź, czy wszystkie obrazy celów zostały poprawnie wykryte

1. Przejdź do obrazu celu
2. Wybierz **RAW (Target)** z menu rozwijanego
3. Sprawdź, czy cele kalibracyjne są wyraźnie widoczne i wykryte
4. Przejdź do następnego obrazu docelowego
5. Powtórz weryfikację dla wszystkich celów

### Przebieg pracy 4: Kontrola wartości pikseli

**Cel**: Sprawdź wartości odbicia pod kątem dokładności naukowej

1. Otwórz przetworzony obraz
2. Wybierz warstwę **RAW (Odbicie)**

3. Włącz tryb**Procent pikseli** (przycisk w prawym górnym pasku narzędzi)
4. Przesuń kursor nad obszary roślinności
5. Sprawdź, czy wartości pikseli mieszczą się w oczekiwanych zakresach (30–70% dla NIR, 5–15% dla Red)
6. Sprawdź, czy obszary gleby i wody mają odpowiednie wartości

***

## Zrozumienie wartości pikseli według warstw

Różne warstwy pokazują różne zakresy wartości pikseli:

### Warstwa JPG

* **Zakres**: 0–255 (8-bitowy)
* **Znaczenie**: Wartości wyświetlane, skorygowane pod kątem gamma
* **Zastosowanie**: Wyłącznie do kontroli wizualnej, nie do pomiarów naukowych

### RAW (oryginalny)

* **Zakres**: 0–65535 (16-bitowy)
* **Znaczenie**: Surowe dane cyfrowe z czujnika
* **Zastosowanie**: Sprawdzanie działania czujnika, bez kalibracji

### RAW (Odbicie)

* **Zakres**: 0–65 535 (16-bitowy TIFF) lub 0,0–1,0 (32-bitowy procent)
* **Znaczenie**: Skalibrowany procent odbicia
* **Zastosowanie**: Pomiary naukowe i analiza**Dla 16-bitowego TIFF:**Należy podzielić przez 65 535, aby uzyskać procent odbicia**Dla 32-bitowego procentu:** Wartości bezpośrednio odpowiadają procentom (0,5 = 50% odbicia)

### RAW (Obrazy indeksowe)

* **Zakres**: Różni się w zależności od indeksu (zazwyczaj od -1,0 do +1,0 dla indeksów znormalizowanych)
* **Znaczenie**: Wynik obliczeń indeksu
* **Przykłady**:
  * NDVI: od -1 do +1 (roślinność zazwyczaj od 0,4 do 0,9)
  * NDRE: od -1 do +1 (wykrywanie stresu)
  * EVI: od 0 do 1 (wzmocniona roślinność)

***

## Wskazówki i najlepsze praktyki

### Efektywne przełączanie warstw

* **Skróty klawiaturowe**: Chociaż nie ma skrótów klawiaturowych dla warstw, strzałki nawigacyjne (←/→) działają we wszystkich warstwach
* **Spójne procedury**: Wybierz jedną warstwę (np. NDVI) i przejrzyj cały zbiór danych przed przełączeniem się na inną
* **Szybkie porównania**: Przełączaj się między warstwą „Original” a „Reflectance”, aby zweryfikować jakość przetwarzania

### Kwestie związane z wydajnością

* **Pliki JPG ładują się najszybciej**: Używaj ich do szybkiej nawigacji po wielu obrazach
* **Warstwy RAW ładują się wolniej**: Wyższa rozdzielczość i głębia bitowa
* **Warstwy indeksowe**: Prędkość podobna do warstw „Reflectance”
* **Pierwsze ładowanie jest najwolniejsze**: Kolejne wyświetlenia tej samej warstwy są buforowane i szybsze

### Weryfikacja jakości

* **Zawsze sprawdzaj warstwę RAW (oryginalną)**: Sprawdź jakość danych źródłowych, zanim zaufasz przetworzonym wynikom
* **Porównaj warstwy**: Użyj przełączania warstw, aby sprawdzić, czy przetwarzanie przebiegło poprawnie
* **Sprawdź zakresy indeksu**: Użyj trybu „Pixel Percent” z warstwami indeksu, aby sprawdzić, czy wartości są rozsądne***

## Rozwiązywanie problemów

### Warstwa niedostępna

**Problem**: Oczekiwana warstwa nie pojawia się w menu rozwijanym**Możliwe przyczyny:**

* Obraz nie został przetworzony (dostępne są tylko formaty JPG i RAW (oryginalny))
* Kalibracja odbicia została wyłączona podczas przetwarzania
* Konkretny indeks nie został skonfigurowany w ustawieniach projektu
* Obraz jest obrazem wyłącznie docelowym (dla obiektów docelowych nie generuje się indeksów)

**Rozwiązania:**

1. Sprawdź, czy obraz został przetworzony (sprawdź folder wyjściowy pod kątem przetworzonych plików)
2. Sprawdź ustawienia projektu, aby potwierdzić, że indeksy zostały skonfigurowane
3. Przetwórz ponownie z włączonymi żądanymi indeksami

### Wyświetlana jest niewłaściwa warstwa

**Problem**: Obraz otwiera się w nieoczekiwanej warstwie**Przyczyna**: Preferencje warstwy z poprzedniego obrazu zostały przeniesione, ale ta warstwa nie istnieje na bieżącym obrazie**Rozwiązanie**: Chloros automatycznie przechodzi na format JPG, gdy preferowana warstwa jest niedostępna — jest to normalne zachowanie

### Brak widocznych celów kalibracyjnych

**Problem**: Warstwa RAW (cel) nie pokazuje wykrytych celów**Możliwe przyczyny:**

* Cele nie zostały wykryte podczas przetwarzania
* Obraz faktycznie nie zawiera celów
* Ustawienia wykrywania celów są zbyt restrykcyjne

**Rozwiązania:**

1. Sprawdź dziennik debugowania pod kątem komunikatów „Target found”
2. Sprawdź, czy obraz faktycznie zawiera widoczne cele kalibracyjne
3. Dostosuj ustawienia wykrywania celów w ustawieniach projektu
4. Zobacz [Wybieranie obrazów docelowych](../processing-images-gui/choosing-target-images.md)

***

## Powiązane funkcje

### Narzędzia przeglądarki obrazów

Podczas przeglądania dowolnej warstwy możesz używać:

* **Elementy sterujące powiększeniem**: Powiększ, aby sprawdzić szczegóły
* **Przesuwanie**: kliknij i przeciągnij, aby poruszać się po powiększonym obrazie
* **Sprawdzanie wartości pikseli**: wyświetl wartości w miejscu kursora
* **Strzałki nawigacyjne**: przechodź między obrazami, zachowując warstwę
* **Tryb procentowy pikseli**: przełączaj między wyświetlaniem wartości DN a wartości procentowej

Zobacz [Otwieranie obrazu na pełnym ekranie](opening-an-image-full-screen.md), aby uzyskać pełną dokumentację przeglądarki obrazów.

### Środowisko testowe indeksu/LUT

Do interaktywnego testowania i wizualizacji indeksu:

* **Obliczanie indeksu w czasie rzeczywistym**: Testuj różne formuły indeksu
* **Mapowanie kolorów LUT**: Zastosuj gradienty kolorów do indeksów w skali szarości
* **Eksportuj wizualizacje**: Zapisz kolorowe obrazy indeksu

Zobacz [Środowisko testowe indeksu/LUT](index-lut-sandbox.md).

***

## Kolejne kroki

Teraz, gdy rozumiesz już warstwy obrazu:

* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) – Kompletny przewodnik po przeglądarce obrazów
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktywna wizualizacja indeksu
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — lista dostępnych indeksów
* [**Zakończenie przetwarzania**](../processing-images-gui/finishing-the-processing.md) — zrozumienie przetworzonych wyników
