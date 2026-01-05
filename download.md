---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Pobierz

Pobierz najnowszą wersję Chloros, aby rozpocząć przetwarzanie obrazów wielospektralnych.

### Wymagania systemowe

| Wymagania          | Minimalne                         | Zalecane                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **System operacyjny** | Windows 10 (64-bitowy)             | Windows 11 (64-bitowy)             |
| **Procesor**        | Intel Core i5 lub równoważny     | Intel Core i7 lub lepszy         |
| **Pamięć (RAM)**     | 8 GB                             | 16 GB lub więcej                    |
| **Karta graficzna**    | Zgodna z DirectX 11           | Procesor graficzny NVIDIA z 4 GB+ pamięci VRAM       |
| **Pamięć masowa**          | 6 GB wolnego miejsca                  | Dysk SSD z 10 GB+ wolnego miejsca       |
| **Wyświetlacz**          | 1920 x 1080                       | 2560 x 1440 lub wyższa             |
| **Internet**         | Wymagany do aktywacji licencji | Wymagany do aktywacji licencji |

{% hint style=&quot;info&quot; %}
**Przyspieszenie GPU**: Użytkownicy Chloros+ z procesorami graficznymi NVIDIA (4 GB+ VRAM) mogą korzystać z przyspieszenia CUDA, aby znacznie przyspieszyć przetwarzanie. Użytkownicy Chloros+ zyskują również możliwość przetwarzania wielowątkowego, zapewniającego maksymalną prędkość.
{% endhint %}

***

## Pobierz Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Pobierz Chloros tutaj</a>

### Najnowsza stabilna wersja

**Chloros Instalator dla Windows*** **Wersja**: 1.0.4
* **Data wydania**: 5 stycznia 2026 r.
* **Rozmiar pliku (pobieranie)**: 1,8 GB
* **Rozmiar pliku (po zainstalowaniu)**: 5,7 GB
* **Typ pliku**: .exe (instalator Windows)

#### **Kroki instalacji:**

1. Pobierz plik `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Kliknij dwukrotnie instalator, aby rozpocząć instalację
3. Postępuj zgodnie z instrukcjami kreatora instalacji
4. Wybierz katalog instalacyjny (domyślnie: `C:\Program Files\[USER]\Chloros\`)
5. Zakończ instalację i uruchom Chloros, Chloros (przeglądarka) lub Chloros CLI
6. Zaloguj się na swoje konto [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (lub kontynuuj korzystanie z wersji bezpłatnej).

{% hint style=&quot;success&quot; %}
Instalator automatycznie dodaje `chloros-cli` do ścieżki PATH systemu, aby umożliwić dostęp z wiersza poleceń.
{% endhint %}

***

## Dodatkowe zasoby

### Python SDK

Dla programistów i procesów automatyzacji należy zainstalować Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacja**: [API: Python SDK](api-python-sdk.md)**Wymagania**: Chloros Desktop musi być zainstalowany, wymagane jest logowanie do licencji Chloros+.***

## Co zawiera pakiet

Instalacja Chloros zawiera:

* ✅ **Chloros** — w pełni funkcjonalny interfejs graficzny
* ✅ **Chloros (przeglądarka)** — interfejs internetowy dla systemów o niższej specyfikacji
* ✅ **Chloros CLI** – interfejs wiersza poleceń (wymaga licencji Chloros+)
* ✅ **Chloros SDK** - Python API (wymaga licencji Chloros+)
* ✅ **Profile kamer** - Wstępnie skonfigurowane szablony kamer MAPIR***

## Aktualizacja do Chloros+

Odblokuj zaawansowane funkcje dzięki subskrypcji Chloros+:

* 🚀 **Przetwarzanie wielowątkowe** — przetwarzaj obrazy równolegle
* ⚡ **Przyspieszenie GPU (CUDA)** — wykorzystaj moc procesora graficznego NVIDIA
* 💻 **Dostęp do CLI** — automatyzacja za pomocą narzędzi wiersza poleceń
* 🐍 **Python SDK** — programowy dostęp do API
* 📱 **Wiele urządzeń** — możliwość korzystania na 2–10+ urządzeniach (w zależności od planu)
* 🧮 **Formuły niestandardowe** — tworzenie niestandardowych indeksów wielospektralnych

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobacz plany i ceny Chloros</a></p>***

## Pomoc dotycząca instalacji

### Rozwiązywanie problemów

**Instalacja kończy się niepowodzeniem i wyświetla komunikat o błędzie:**

* Upewnij się, że masz uprawnienia administratora
* Tymczasowo wyłącz oprogramowanie antywirusowe
* Sprawdź, czy spełniasz minimalne wymagania systemowe

**Aplikacja nie uruchamia się:**

* Wypróbuj wersję Chloros (przeglądarka)
* Sprawdź, czy zainstalowano Windows 10/11 (64-bit)
* Zaktualizuj sterowniki graficzne
* Sprawdź Windows Podgląd zdarzeń, aby uzyskać szczegółowe informacje o błędzie
* Skontaktuj się z pomocą techniczną, podając logi błędów

**Problemy z aktywacją licencji:**

* Upewnij się, że połączenie internetowe jest aktywne
* Sprawdź dane uwierzytelniające w [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Sprawdź, czy zapora sieciowa nie blokuje Chloros
* Szczegółowe instrukcje można znaleźć w sekcji [Chloros+ Login](chloros+-login.md)

### Pomoc techniczna

Potrzebujesz pomocy przy instalacji lub konfiguracji?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacja**: [Pierwsze kroki](./)
* ❓ **FAQ**: [Często zadawane pytania](faq.md)***

## Lista zmian

<details>

<summary>Wersja 1.0.4</summary>

#### **Data wydania**: 5 stycznia 2026 r.**Nowe funkcje*** **Przełącznik obrazu/metadanych**: Dodano przełącznik w przeglądarce plików, aby wyświetlać metadane wybranego obrazu w tabeli zamiast w siatce obrazów
* **Suwak powiększania siatki obrazów**: Nowy suwak interfejsu użytkownika do regulacji rozmiaru miniatur (obsługuje również kombinację klawiszy CTRL + kółko myszy).
* **Przyciski eksportu siatki obrazów**: przyciski w górnym rzędzie służące do przełączania miniatur z formatu JPG na przetworzone eksporty (cele, odbicie, indeks, LUT).
* **Zakładka Mapa**: nowa interaktywna mapa 2D pokazująca znaczniki lokalizacji GPS obrazów.
  * Obsługuje mapy Google Maps i kafelki map ESRI (automatycznie wybiera najlepszą usługę kafelków w oparciu o dostępność poziomu powiększenia).
  * Podgląd miniatury po najechaniu myszką na znaczniki mapy.

**Poprawki błędów*** Ulepszona obsługa instalacji Chloros na komputerach z językiem innym niż angielski.

</details>

<details>

<summary>Wersja 1.0.3</summary>

#### **Data wydania**: 20 grudnia 2025 r.**Nowe funkcje*** Pierwsze uruchomienie

**Ulepszenia*** Pierwsze uruchomienie

**Poprawki błędów*** Pierwsze uruchomienie

**Znane problemy*** Pierwsze uruchomienie

</details>***

## Umowa licencyjna**Oprogramowanie zastrzeżone** — Copyright (c) 2025 MAPIR Inc.

Niedozwolone jest nieautoryzowane użytkowanie, dystrybucja lub modyfikacja.

**Wersja bezpłatna**: dostępna do użytku osobistego i komercyjnego z ograniczeniami funkcji.**Chloros+**: licencja oparta na subskrypcji, umożliwiająca korzystanie z zaawansowanych funkcji i wdrożeń komercyjnych.
