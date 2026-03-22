# Zakończenie przetwarzania

Po zakończeniu przetwarzania przez Chloros nadszedł czas na przejrzenie wyników, sprawdzenie jakości wyjściowej oraz przygotowanie przetworzonych obrazów do wykorzystania w procesie roboczym. Ta strona zawiera instrukcje dotyczące ostatnich kroków i dalszych działań.

## Wskazanie zakończenia przetwarzania

Po pomyślnym zakończeniu przetwarzania zobaczysz kilka wskaźników:

* ✅ **Pasek postępu**: osiąga 100%
* ✅ **Dziennik debugowania**: wyświetla komunikat „Przetwarzanie zakończone”
* ✅ **Przycisk Start**: staje się ponownie aktywny (gotowy do kolejnego przetwarzania)
* ✅ **Pliki wyjściowe**: Wszystkie przetworzone obrazy zapisane w podfolderze modelu aparatu***

## Lokalizowanie przetworzonych obrazów

### Otwieranie folderu wyjściowego

1. Kliknij ikonę **Menu głównego** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (w lewym górnym rogu)
2. Wybierz **„Otwórz folder projektu”**

3. Otworzy się eksplorator plików w katalogu projektu
4. Znajdź swój projekt według nazwy

***

## Przeglądanie przetworzonych obrazów

### Szybki podgląd w Eksploratorze plików

**Wbudowany podgląd Windows:**

1. Przejdź do podfolderu modelu kamery
2. Wybierz plik obrazu
3. Podgląd pojawi się w panelu podglądu Eksploratora Windows
4. Użyj klawiszy strzałek, aby przeglądać obrazy

### Podgląd w zewnętrznych przeglądarkach obrazów

**Zalecane przeglądarki:*** **QGIS** – bezpłatne oprogramowanie GIS (najlepsze do analizy wielospektralnej z georeferencjami)
* **IrfanView** – szybka, lekka przeglądarka obrazów (obsługuje TIFF)
* **Adobe Photoshop** — profesjonalna edycja (obsługa TIFF)
* **GIMP** — darmowa alternatywa dla programu Photoshop
* **Windows Photos** — podstawowe przeglądanie (może nie obsługiwać 16-bitowego formatu TIFF)

### Podgląd w przeglądarce obrazów Chloros

Użyj wbudowanej przeglądarki obrazów Chloros do zaawansowanej wizualizacji:

1. Kliknij miniaturę obrazu w przeglądarce plików
2. Obraz otworzy się w głównym obszarze podglądu
3. Kliknij zakładkę **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na pasku bocznym po lewej stronie
4. Użyj [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) do interaktywnej analizy

Szczegółowe instrukcje znajdziesz w [Przeglądarce obrazów](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Przeglądanie dziennika debugowania

### Sprawdź, czy są ostrzeżenia lub błędy

1. Otwórz zakładkę **Dziennik debugowania** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Przewiń komunikaty
3. Poszukaj żółtych ostrzeżeń lub czerwonych błędów
4. Przejrzyj wszystkie odnotowane problemy
5. Skontaktuj się z pomocą techniczną MAPIR w celu uzyskania pomocy

### Zapisywanie dziennika

Aby zachować zapis przetwarzania lub wysłać go do pomocy technicznej MAPIR:

1. Kliknij przycisk **„Kopiuj”**lub**„Pobierz”**

2. Zapisz jako plik tekstowy w folderze projektu
3. Dołącz do dokumentacji projektu
4. Wyślij do pomocy technicznej MAPIR, jeśli napotkasz problemy

***

## Typowe problemy z wynikami i rozwiązania

### Problem: Brakujące pliki wyjściowe

**Możliwe przyczyny:**

* Pliki nie spełniały kryteriów przetwarzania
* Obrazy przeznaczone wyłącznie dla docelowego urządzenia (wykluczone z eksportu)
* Zabrakło miejsca na dysku podczas eksportu
* Uszkodzenie plików podczas przetwarzania

**Rozwiązania:**

1. Sprawdź dziennik debugowania pod kątem komunikatów o pominięciu/błędach
2. Upewnij się, że było wystarczająco dużo miejsca na dysku
3. Policz pliki: Liczba powinna odpowiadać (liczba oryginalna - liczba docelowa) × (indeksy + 1)
4. Ponownie zaimportuj i przetwórz brakujące pliki

### Problem: Ciemne lub jasne krawędzie (wciąż widoczne winietowanie)

**Możliwe przyczyny:**

* Wyłączona korekcja winietowania
* Aparat/obiektyw nie figuruje w bazie danych profili Chloros
* Ekstremalne winietowanie przekraczające możliwości korekcji

**Rozwiązania:**

1. Sprawdź, czy korekcja winietowania została włączona w ustawieniach projektu
2. Sprawdź, czy model aparatu został poprawnie wykryty
3. Skontaktuj się z pomocą techniczną MAPIR, jeśli winietowanie nadal występuje

### Problem: Nieprawidłowe kolory lub wartości

**Możliwe przyczyny:**

* Nie wykryto celów kalibracyjnych
* Wybrano niewłaściwy model celu kalibracyjnego
* Wyłączona kalibracja odbicia
* Słaba jakość obrazów celów

**Rozwiązania:**

1. Sprawdź, czy kalibracja odbicia została włączona
2. Sprawdź komunikaty „Znaleziono cel” w dzienniku debugowania
3. Sprawdź jakość obrazów celów
4. Przetwórz ponownie z zaznaczonymi właściwymi celami

### Problem: Wartości NDVI wydają się nieprawidłowe

**Oczekiwane zakresy NDVI:*** **Woda, skały, gleba**: od -0,1 do 0,2
* **Rzadka/niezdrowa roślinność**: od 0,2 do 0,4
* **Umiarkowana roślinność**: od 0,4 do 0,6
* **Zdrowa, gęsta roślinność**: od 0,6 do 0,9**Jeśli wartości wykraczają poza te zakresy:**

1. Sprawdź, czy zastosowano kalibrację odbicia
2. Sprawdź, czy dołączono dziennik czujnika światła
3. Sprawdź, czy wykryto cele kalibracyjne
4. Upewnij się, że wykryto prawidłowy model kamery
5. Sprawdź czas i warunki wykonania zdjęć celów

***

## Korzystanie z przetworzonych zdjęć

### Do fotogrametrii / tworzenia ortomosaiki

**Zalecany przebieg pracy:**

1.**Zaimportuj skalibrowane obrazy odbicia** do oprogramowania fotogrametrycznego:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachowaj metadane EXIF**: Upewnij się, że dane GPS zostały zachowane do geotagowania
3. **Skalibrowane procedury**: Używaj obrazów odbicia dla naukowej dokładności
4. **Przetwarzaj mozaiki indeksowe**: Utwórz ortomosaiki NDVI z poszczególnych obrazów indeksowych
5. **Eksportuj georeferencjonowane GeoTIFF**: Do wykorzystania w aplikacjach GIS

### Do analizy GIS

**Zalecany przebieg pracy:**

1.**Załaduj do QGIS, ArcGIS lub podobnego oprogramowania**

2.**Użyj 16-bitowych obrazów odbicia TIFF** do analizy wielopasmowej
3. **Użyj obrazów indeksowych** (NDVI, NDRE) jako gotowych warstw roślinności
4. **Kalkulator rastrowy**: Połącz pasma do analizy niestandardowej
5. **Eksport**: Twórz mapy klasyfikacyjne, wykrywanie zmian, mapy stanu roślinności

### Do bezpośredniej analizy / raportowania

**Zalecany przebieg pracy:**

1.**Używaj obrazów indeksowych z kolorami LUT** do raportów wizualnych
2. **Wyodrębnij statystyki**: Średnia NDVI na pole/działkę
3. **Seria czasowa**: Porównaj wskaźniki z wielu sesji
4. **Generuj raporty**: Dołącz mapy, statystyki i wizualizacje***

## Archiwizacja i tworzenie kopii zapasowych

### Zalecana strategia tworzenia kopii zapasowych

**Co zapisać:*** ✅ **Oryginalne obrazy RAW/JPG** – Archiwizuj na oddzielnym dysku/w chmurze
* ✅ **Wyniki przetworzenia** – zachowaj skalibrowane obrazy i wskaźniki
* ✅ **Plik projektu** – zawiera wszystkie ustawienia potrzebne do ponownego przetworzenia w razie potrzeby
* ✅ **Dziennik debugowania** – dokumentuje szczegóły przetwarzania
* ✅ **Obrazy wzorców kalibracyjnych** – do weryfikacji i ponownego przetworzenia**Zalecenia dotyczące przechowywania:*** **Natychmiastowa kopia zapasowa**: Zewnętrzny dysk twardy
* **Archiwum długoterminowe**: Przechowywanie w chmurze (Google Drive, Dropbox itp.)
* **Dane krytyczne**: Zachowaj 2–3 kopie w różnych lokalizacjach***

## Kolejne przebiegi przetwarzania

### Ponowne wykorzystanie ustawień projektu

Jeśli w przyszłości będziesz przetwarzać podobne zbiory danych:

1. **Zapisz szablon projektu** (jeśli jeszcze tego nie zrobiłeś)
2. **Utwórz nowy projekt** przy użyciu zapisanego szablonu
3. **Zaimportuj nowe obrazy**

4.**Przetwórz**przy użyciu identycznych ustawień w celu zachowania spójności

### Przetwarzanie wsadowe wielu sesji

W przypadku wielu sesji/zbiorów danych:**Opcja 1: GUI – wiele projektów**

* Utwórz oddzielny projekt dla każdej sesji
* Używaj spójnych ustawień szablonu
* Przetwarzaj po jednym na raz

**Opcja 2: Chloros CLI (tylko Chloros+)**

* Zautomatyzuj przetwarzanie wsadowe
* Przetwarzaj wiele folderów za pomocą skryptów
* Zobacz [Dokumentacja CLI](../CLI.md)

**Opcja 3: Python SDK (tylko Chloros+)**

* Sterowanie programowe
* Integracja z potokami analitycznymi
* Zobacz [Dokumentacja API](../api-python-sdk.md)

***

## Rozwiązywanie problemów związanych z przetwarzaniem końcowym

### Ponowne przetwarzanie z innymi ustawieniami

Jeśli wyniki nie są zadowalające:

1. Zachowaj oryginalne obrazy (nigdy ich nie usuwaj)
2. Otwórz ten sam projekt w Chloros
3. Dostosuj ustawienia w panelu Ustawienia projektu
4. Przetwórz ponownie — wyniki zastąpią poprzednie

### Przetwarzanie podzbioru obrazów

Aby ponownie przetworzyć tylko określone obrazy:

1. Utwórz nowy projekt
2. Zaimportuj tylko te obrazy, które wymagają ponownego przetworzenia
3. Użyj tego samego szablonu ustawień
4. Przetwórz mniejszy zbiór danych

### Pomoc

W przypadku problemów:

* 📧 **E-mail**: info@mapir.camera (dołącz dziennik debugowania)
* 🌐 **Pomoc techniczna**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Często zadawane pytania](../faq.md)
* 📖 **Dokumentacja**: [Podręcznik Chloros](../)***

## Podsumowanie: Kompletny przebieg pracy

Właśnie zakończyłeś cały proces przetwarzania Chloros:

1. ✅ **Utworzono projekt** – zobacz [Projekty](../projects.md)
2. ✅ **Dodano pliki** — zobacz [Dodawanie plików](adding-files-to-a-project.md)
3. ✅ **Dostosowano ustawienia** — zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md)
4. ✅ **Oznaczono cele** — zobacz [Wybieranie obrazów docelowych](choosing-target-images.md)
5. ✅ **Rozpoczęto przetwarzanie** — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
6. ✅ **Monitorowany postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)
7. ✅ **Sprawdzone wyniki** — ta strona**Twoje skalibrowane, skorygowane pod kątem odbicia obrazy wielospektralne są gotowe do analizy!**

***

## Dodatkowe zasoby

### Funkcje zaawansowane

* [**Przeglądarka obrazów**](../image-viewer-gui/opening-an-image-full-screen.md) — interaktywna wizualizacja i analiza
* [**Sandbox indeksów/LUT**](../image-viewer-gui/index-lut-sandbox.md) — testowanie niestandardowych indeksów
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) – Kompletny zbiór indeksów

### Automatyzacja i integracja

* [**Dokumentacja CLI**](../CLI.md) – Przetwarzanie wsadowe z wiersza poleceń
* [**Python SDK**](../api-python-sdk.md) – Automatyzacja programowa
* [**Chloros+ Funkcje**](../#chloros) – Zaawansowane możliwości przetwarzania

### Pomoc techniczna i nauka

* [**FAQ**](../faq.md) – odpowiedzi na najczęściej zadawane pytania
* [**Targety kalibracyjne**](../calibration-targets.md) – zrozumienie kalibracji współczynnika odbicia
* [**Obsługiwane kamery**](../supported-cameras.md) – Kompatybilny sprzęt
