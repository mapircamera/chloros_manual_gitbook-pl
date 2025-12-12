# Warstwy obrazu

Rozwijane menu Warstwy obrazu w przeglądarce obrazów Chloros umożliwia szybkie przełączanie się między różnymi wersjami tego samego obrazu — od oryginalnych ujęć po przetworzone wyniki odbicia i obliczone obrazy indeksowe.

## Czym są warstwy obrazu?

W Chloros **warstwy** odnoszą się do różnych wyników obrazu dostępnych dla jednego obrazu źródłowego. Podczas przetwarzania obrazów program Chloros tworzy wiele wersji:

* **Oryginalne obrazy** (pliki JPG i RAW z aparatu)
* Obrazy **z kalibracją odbicia** (jeśli kalibracja odbicia była włączona)
* **Obrazy docelowe** (jeśli obraz zawiera cele kalibracyjne)
* **Obrazy indeksowe** (NDVI, NDRE, GNDVI itp., jeśli skonfigurowano indeksy)

**Rozwijane menu wyboru warstw** w prawym górnym rogu przeglądarki obrazów pozwala na natychmiastowe przełączanie się między tymi wersjami bez opuszczania przeglądarki.

***

## Dostępne typy warstw

### JPG

* Oryginalny obraz podglądu JPG z aparatu
* Zawsze dostępny dla wszystkich obrazów
* Nieprzetworzony, taki jak uchwycony przez aparat
* Najszybszy do załadowania i wyświetlenia

**Kiedy wyświetlać:**

* Szybki podgląd oryginalnego zdjęcia
* Sprawdzanie kompozycji i kadrowania obrazu
* Sprawdzanie jakości zdjęcia przed przetworzeniem

### RAW (oryginalny)

* Oryginalne dane z czujnika RAW z aparatu
* Bez przetwarzania po usunięciu efektu debayeringu
* Wyższa głębia bitowa niż JPG (zazwyczaj 12-bitowe lub 14-bitowe dane z czujnika)

**Kiedy wyświetlać:**

* Sprawdzanie jakości oryginalnych danych z czujnika
* Sprawdzanie problemów z czujnikiem lub artefaktów
* Porównanie wyników przed i po przetworzeniu

### RAW (cel)

* Wyświetla się tylko w przypadku obrazów zidentyfikowanych jako zawierające cele kalibracyjne
* Pokazuje oryginalny obraz RAW z wykrytym celem
* Służy do weryfikacji, czy wykrycie celu zakończyło się powodzeniem

**Kiedy wyświetlać:**

* Potwierdzanie, że cele kalibracyjne zostały wykryte poprawnie
* Sprawdzanie jakości obrazu celu
* Rozwiązywanie problemów związanych z kalibracją

{% hint style=&quot;info&quot; %}
**Warstwa docelowa**: Ta warstwa pojawia się tylko w menu rozwijanym dla obrazów zawierających cele kalibracyjne. Zwykłe obrazy przechwycone nie będą miały tej opcji.
{% endhint %}

### RAW (odbicie)

* Skalibrowany obraz odbicia
* Korekcja winietowania (jeśli włączona w przetwarzaniu)
* Odbicie skalibrowane przy użyciu danych docelowych (jeśli włączone)
* Wielopasmowe TIFF ze wszystkimi kanałami kamery
* Wartości pikseli reprezentują procent odbicia (przy użyciu trybu procentowego)
* Gotowe do manipulacji za pomocą [Index/LUT Sandbox](index-lut-sandbox.md)

**Kiedy wyświetlać:**

* Sprawdzanie skalibrowanych wyników
* Weryfikacja jakości kalibracji
* Sprawdzanie wartości pikseli pod kątem dokładności naukowej
* Porównanie z oryginałem w celu sprawdzenia efektów kalibracji

{% hint style=&quot;success&quot; %}
**Zalecane**: Podczas sprawdzania wartości pikseli do pomiarów naukowych i analiz należy używać warstwy RAW (odbicie).
{% endhint %}

### RAW (NDVI Index)... i podobne

* Obliczony obraz indeksu wegetacji (w tym przykładzie NDVI)
* Nazwa indeksu zmienia się w zależności od tego, który indeks został skonfigurowany podczas przetwarzania
* Przykłady: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) itp.
* Jednopasmowy obraz w skali szarości przedstawiający wyniki obliczeń indeksu
* Dla każdego indeksu skonfigurowanego w ustawieniach projektu pojawia się jedna warstwa

**Możliwe nazwy indeksów:**

* RAW (indeks NDVI)
* RAW (indeks NDRE)
* RAW (indeks GNDVI)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* I wiele innych... (patrz [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md))

**Kiedy wyświetlać:**

* Sprawdzanie wyników obliczeń indeksu
* Sprawdzanie zakresów wartości indeksu
* Identyfikowanie obszarów zainteresowania
* Sprawdzanie obrazów indeksu przed użyciem w GIS lub analizie

***

## Korzystanie z selektora warstw

### Otwieranie menu rozwijanego

1. Otwórz obraz w trybie pełnoekranowym (kliknij dowolną miniaturę w przeglądarce obrazów).
2. Znajdź **menu rozwijane warstw** w prawym górnym rogu przeglądarki
3. Menu rozwijane pokazuje aktualnie wybraną warstwę (np. „JPG”)
4. Kliknij menu rozwijane, aby wyświetlić wszystkie dostępne warstwy

### Przełączanie warstw

1. Kliknij menu rozwijane warstw, aby otworzyć listę
2. Wyświetlane są wszystkie dostępne warstwy dla bieżącego obrazu
3. Kliknij dowolną nazwę warstwy, aby przełączyć się do tej wersji
4. Obraz zostanie natychmiast zaktualizowany, aby wyświetlić wybraną warstwę.

**Szybkie przełączanie:**

* Menu rozwijane zapamiętuje ostatni wybór.
* Podczas przechodzenia do następnego obrazu Chloros próbuje wyświetlić ten sam typ warstwy.
* Jeśli ta warstwa nie istnieje na następnym obrazie, domyślnie wyświetlana jest warstwa JPG.

### Dostępność warstw

Nie wszystkie warstwy są dostępne dla każdego obrazu:

**Zawsze dostępne:**

* ✅ JPG (każdy obraz ma podgląd JPG)

**Dostępne warunkowo:**

* ⚠️ RAW (oryginalny) — tylko jeśli obraz został zarejestrowany w trybie RAW lub RAW+JPG
* ⚠️ RAW (cel) — tylko jeśli obraz zawiera wykryte cele kalibracyjne
* ⚠️ RAW (odbicie) — tylko po przetworzeniu z włączoną kalibracją odbicia
* ⚠️ RAW (\[indeks] indeks) — tylko po przetworzeniu z skonfigurowanymi indeksami

***

## Trwałość warstwy

### Przechodzenie między obrazami

Podczas przechodzenia do innego obrazu (za pomocą klawiszy strzałek lub kliknięcia miniatur):

**Preferencje warstw są zachowywane:**

* Jeśli wyświetlasz „RAW (odbicie)”, następny obraz pokazuje „RAW (odbicie)” (jeśli jest dostępny)
* Jeśli wyświetlasz „RAW (NDVI indeks)”, następny obraz pokazuje „RAW (NDVI indeks)” (jeśli jest dostępny)
* Jeśli ta sama warstwa nie istnieje, domyślnie wyświetlany jest plik JPG

**Przykładowy przebieg pracy:**

1. Otwórz obraz 1, przejdź do RAW (NDVI Index)
2. Naciśnij →, aby wyświetlić obraz 2
3. Obraz 2 automatycznie wyświetla warstwę RAW (NDVI Index)
4. Kontynuuj nawigację — wszystkie obrazy wyświetlają warstwę NDVI.
5. Bardzo wydajne rozwiązanie do przeglądania wyników indeksowania wielu obrazów.

***

## Typowe przepływy pracy

### Przepływ pracy 1: Porównanie przed/po

**Cel**: Porównanie obrazu oryginalnego z obrazem skalibrowanym.

1. Otwórz przetworzony obraz w przeglądarce obrazów.
2. Wybierz **RAW (oryginalny)** z menu rozwijanego
3. Zwróć uwagę na winietowanie i nieskalibrowane wartości
4. Przejdź do **RAW (odbicie)** z menu rozwijanego
5. Porównaj — winietowanie zostało usunięte, a wartości skalibrowane

### Przebieg pracy 2: Przegląd indeksu

**Cel**: Szybki przegląd wyników NDVI w całym zestawie danych

1. Otwórz pierwszy przetworzony obraz.
2. Wybierz **RAW (NDVI Index)** z menu rozwijanego.
3. Użyj klawisza strzałki →, aby przejść do następnego obrazu.
4. Warstwa NDVI pozostaje automatycznie.
5. Przejrzyj wszystkie obrazy, sprawdzając wzorce NDVI.
6. Przejdź do **RAW (NDRE Index)**, aby porównać

### Przepływ pracy 3: Weryfikacja celu

**Cel**: Sprawdź, czy wszystkie obrazy docelowe zostały wykryte poprawnie

1. Przejdź do obrazu docelowego
2. Wybierz **RAW (Target)** z menu rozwijanego
3. Sprawdź, czy cele kalibracyjne są wyraźnie widoczne i wykryte
4. Przejdź do następnego obrazu docelowego.
5. Powtórz weryfikację dla wszystkich celów.

### Przebieg pracy 4: Kontrola wartości pikseli

**Cel**: Sprawdź wartości odbicia pod kątem dokładności naukowej.

1. Otwórz przetworzony obraz.
2. Wybierz warstwę **RAW (Odbicie)**.
3. Włącz tryb **Procent pikseli** (przycisk w prawym górnym pasku narzędzi).
4. Przesuń kursor nad obszary roślinności.
5. Sprawdź, czy wartości pikseli mieszczą się w oczekiwanych zakresach (30–70% dla NIR, 5–15% dla Red).
6. Sprawdź, czy wartości dla obszarów gleby i wody są odpowiednie.

***

## Rozumienie wartości pikseli według warstw

Różne warstwy pokazują różne zakresy wartości pikseli:

### Warstwa JPG

* **Zakres**: 0-255 (8 bitów)
* **Znaczenie**: Wyświetlane wartości, skorygowane pod kątem gamma
* **Zastosowanie**: Tylko do kontroli wzrokowej, nie do pomiarów naukowych

### RAW (oryginalne)

* **Zakres**: 0-65535 (16 bitów)
* **Znaczenie**: Surowe wartości cyfrowe czujnika
* **Zastosowanie**: Sprawdzanie wydajności czujnika, niekalibrowane

### RAW (odbicie)

* **Zakres**: 0–65 535 (16-bitowe TIFF) lub 0,0–1,0 (32-bitowe procenty)
* **Znaczenie**: Skalibrowane procentowe odbicie
* **Zastosowanie**: pomiary naukowe i analiza

**W przypadku 16-bitowego TIFF:** podziel przez 65 535, aby uzyskać procent odbicia **W przypadku 32-bitowego procentu:** wartości bezpośrednio reprezentują procent (0,5 = 50% odbicia)

### RAW (obrazy indeksowe)

* **Zakres**: różni się w zależności od indeksu (zazwyczaj od -1,0 do +1,0 dla indeksów znormalizowanych)
* **Znaczenie**: wynik obliczeń indeksu
* **Przykłady**:
  * NDVI: od -1 do +1 (roślinność zazwyczaj od 0,4 do 0,9)
  * NDRE: od -1 do +1 (wykrywanie stresu)
  * EVI: od 0 do 1 (wzbogacona roślinność)

***

## Wskazówki i najlepsze praktyki

### Efektywne przełączanie warstw

* **Znajomość skrótów klawiaturowych**: Chociaż nie ma skrótów klawiaturowych dla warstw, strzałki nawigacyjne (←/→) działają we wszystkich warstwach
* **Spójne przepływy pracy**: wybierz jedną warstwę (np. NDVI) i przejrzyj cały zestaw danych przed przełączeniem się na inną
* **Szybkie porównania**: przełączaj się między opcjami „Oryginał” i „Odbicie”, aby zweryfikować jakość przetwarzania

### Uwagi dotyczące wydajności

* **Najszybsze ładowanie plików JPG**: używaj do szybkiej nawigacji po wielu obrazach.
* **Warstwy RAW ładują się wolniej**: wyższa rozdzielczość i głębia bitowa.
* **Warstwy indeksowe**: podobna prędkość do warstw odbicia.
* **Pierwsze ładowanie jest najwolniejsze**: kolejne wyświetlenia tej samej warstwy są buforowane i szybsze.

### Weryfikacja jakości

* **Zawsze sprawdzaj RAW (oryginał)**: zanim zaufasz przetworzonym wynikom, sprawdź jakość danych źródłowych.
* **Porównaj warstwy**: użyj przełączania warstw, aby sprawdzić, czy przetwarzanie przebiegło poprawnie
* **Sprawdź zakresy indeksów**: użyj trybu procentu pikseli z warstwami indeksowymi, aby sprawdzić, czy wartości są rozsądne

***

## Rozwiązywanie problemów

### Warstwa niedostępna

**Problem**: oczekiwana warstwa nie pojawia się w menu rozwijanym

**Możliwe przyczyny:**

* Obraz nie został przetworzony (dostępne są tylko pliki JPG i RAW (oryginalne))
* Kalibracja odbicia została wyłączona podczas przetwarzania
* W ustawieniach projektu nie skonfigurowano określonego indeksu
* Obraz jest obrazem tylko dla celów (nie wygenerowano indeksów dla celów)

**Rozwiązania:**

1. Sprawdź, czy obraz został przetworzony (sprawdź folder wyjściowy pod kątem przetworzonych plików)
2. Sprawdź ustawienia projektu, aby potwierdzić, że indeksy zostały skonfigurowane
3. Przetwórz ponownie z włączonymi żądanymi indeksami.

### Wyświetlana jest niewłaściwa warstwa

**Problem**: Obraz otwiera się w nieoczekiwanej warstwie.

**Przyczyna**: Preferencje warstwy z poprzedniego obrazu zostały przeniesione, ale ta warstwa nie istnieje w bieżącym obrazie.

**Rozwiązanie**: Chloros automatycznie przechodzi do formatu JPG, gdy preferowana warstwa jest niedostępna — jest to normalne zachowanie.

### Nie widać celów kalibracji

**Problem**: Warstwa RAW (cel) nie pokazuje wykrytych celów.

**Możliwe przyczyny:**

* Cele nie zostały wykryte podczas przetwarzania.
* Obraz faktycznie nie zawiera celów.
* Ustawienia wykrywania celów są zbyt restrykcyjne.

**Rozwiązania:**

1. Sprawdź dziennik debugowania pod kątem komunikatów „Target found” (Wykryto cel).
2. Sprawdź, czy obraz faktycznie zawiera widoczne cele kalibracji.
3. Dostosuj ustawienia wykrywania celów w ustawieniach projektu.
4. Zobacz [Wybieranie obrazów docelowych](../processing-images-gui/choosing-target-images.md).

***

## Powiązane funkcje

### Narzędzia przeglądarki obrazów

Podczas przeglądania dowolnej warstwy można używać:

* **Elementy sterujące powiększeniem**: powiększ, aby sprawdzić szczegóły.
* **Przesuwanie**: kliknij i przeciągnij, aby przesuwać powiększony obraz.
* **Sprawdzanie wartości pikseli**: wyświetlanie wartości w miejscu kursora
* **Strzałki nawigacyjne**: przechodzenie między obrazami przy zachowaniu warstwy
* **Tryb procentowy pikseli**: przełączanie między wyświetlaniem DN a procentowym

Pełna dokumentacja przeglądarki obrazów znajduje się w sekcji [Otwieranie obrazu na pełnym ekranie](opening-an-image-full-screen.md).

### Sandbox indeksu/LUT

Do interaktywnego testowania indeksu i wizualizacji:

* **Obliczanie indeksu w czasie rzeczywistym**: testuj różne formuły indeksu
* **Mapowanie kolorów LUT**: stosuj gradienty kolorów do indeksów w skali szarości
* **Eksportuj wizualizacje**: zapisuj kolorowe obrazy indeksu

Szczegółowe informacje można znaleźć w sekcji [Sandbox indeksu/LUT](index-lut-sandbox.md).

***

## Kolejne kroki

Teraz, gdy rozumiesz już warstwy obrazu:

* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — kompletny przewodnik po przeglądarce obrazów
* [**Sandbox indeksów/LUT**](index-lut-sandbox.md) — interaktywna wizualizacja indeksów
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — dostępne indeksy referencyjne
* [**Zakończenie przetwarzania**](../processing-images-gui/finishing-the-processing.md) — zrozumienie przetworzonych wyników
