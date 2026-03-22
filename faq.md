---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Często zadawane pytania

<details>

<summary>Czy mogę przetwarzać obrazy z kamer innych marek niż MAPIR za pomocą Chloros?</summary>

Nie, Chloros obsługuje wyłącznie przetwarzanie obrazów z kamer MAPIR. Więcej informacji można znaleźć na liście [obsługiwanych modeli kamer](supported-cameras.md). Oferujemy przetwarzanie obrazów z innych kamer w chmurze MAPIR, pełną listę można znaleźć [tutaj](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Czy mogę skalibrować moje obrazy pod kątem współczynnika odbicia bez użycia tarczy kalibracyjnej?</summary>

Nie. Bez obrazu tarczy kalibracyjnej zarejestrowanego w tym samym czasie, co obrazy bez tarczy, nie będzie można powiązać wartości pikseli obrazu ze znanym procentem odbicia. Jeśli nie dołączysz również logu z czujnika światła MAPIR, spektrum światła otoczenia nie zostanie zmierzone, a wyniki odbicia nie będą dokładne.

</details>

<details>

<summary>Czy mogę edytować moje obrazy przed przetworzeniem w Chloros?</summary>

Nie. Program Chloros zakłada, że dane wejściowe nie zostały zmodyfikowane. Nie należy zmieniać nazw plików.

</details>

<details>

<summary>Czy mogę ustawić moje aparaty MAPIR i Survey3 na automatyczną ekspozycję i przetwarzać zdjęcia w Chloros?</summary>

Nie. Zbiory danych obrazów Survey3 muszą mieć stałą/zablokowaną ekspozycję, więc nie ma automatycznej prędkości migawki ani automatycznego ISO. Wszystkie obrazy z tego samego modelu kamery muszą mieć identyczną prędkość migawki i ISO (ekspozycję).

</details>

<details>

<summary>Czy Chloros może przetwarzać lub analizować obrazy ortomosaiczne?</summary>

Nie. Obsługiwane są tylko pojedyncze zdjęcia z aparatu MAPIR, a nie zdjęcia połączone, takie jak mapa ortomosaiczna.

</details>

<details>

<summary>Jak mogę przyspieszyć etap wykrywania celów w Chloros?</summary>

W tabeli przeglądarki plików wstępny wybór obrazów docelowych w prawej kolumnie spowoduje, że Chloros będzie szukać celów kalibracyjnych tylko w tych obrazach, co znacznie przyspieszy przetwarzanie.

</details>

<details>

<summary>Jeśli zamierzam przesłać moje obrazy do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">chmury MAPIR,</a> czy powinienem przetworzyć je w Chloros przed przesłaniem?</summary>

Jeśli planujesz przesłać pliki na naszą platformę przetwarzania online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nie edytuj obrazów przed przesłaniem. Chmura wykona wszystkie te same operacje i wiele więcej.

</details>

<details>

<summary>Czy MAPIR kiedykolwiek będzie obsługiwać funkcję X? Bardzo chciałbym, aby MAPIR oferowało funkcję X.</summary>

Zawsze jesteśmy zainteresowani opiniami na temat naszych produktów. Jeśli zauważysz problem z naszymi produktami lub masz sugestię, jak możemy je ulepszyć, skontaktuj się z nami [CONTACT US](https://www.mapir.camera/community/contact), aby podzielić się swoimi przemyśleniami. Większość naszych działań badawczo-rozwojowych opiera się na wsłuchiwaniu się w najważniejsze potrzeby naszych klientów.

</details>

<details>

<summary>Czy Chloros jest dostępny dla Linux?</summary>

Tak! Chloros 1.1.0 obsługuje Linux amd64 (x86_64) i arm64 (NVIDIA Jetson JetPack 6) poprzez pakiety `.deb`. CLI oraz Python SDK są w pełni obsługiwane na Linux. Nie ma GUI dla Linux — cała interakcja odbywa się za pośrednictwem [CLI](CLI.md) lub [Python SDK](api-python-sdk.md). Szczegółowe informacje można znaleźć w [Przegląd Linux](linux/linux-overview.md).

</details>

<details>

<summary>Czy mogę uruchomić Chloros na platformie NVIDIA Jetson?</summary>

Tak! Chloros 1.1.0 obsługuje platformy NVIDIA Jetson, w tym Jetson Nano, Orin Nano, Orin NX i AGX Orin z systemem JetPack 6. Chloros automatycznie wykrywa model Jetson i optymalizuje strategię przetwarzania. Zobacz [Przewodnik po NVIDIA Jetson](linux/nvidia-jetson-guide.md), aby uzyskać instrukcje dotyczące konfiguracji i wdrożenia.

</details>

<details>

<summary>Czy Chloros automatycznie optymalizuje się pod kątem mojego sprzętu?</summary>

Tak! Chloros 1.1.0 zawiera funkcję [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), która automatycznie wykrywa procesor, kartę graficzną, pamięć RAM oraz (w przypadku Jetson) czujniki temperatury. Następnie wybiera optymalną strategię przetwarzania — od `GPU_PARALLEL` w systemach z dużą pamięcią, przez `GPU_SINGLE` na urządzeniach o ograniczonych zasobach, aż po `CPU_PARALLEL` w systemach bez procesora graficznego NVIDIA. Nie jest wymagana żadna ręczna konfiguracja.

</details>

<details>

<summary>Czym jest 4-wątkowa potok przetwarzania?</summary>

Chloros 1.1.0 wykorzystuje 4-wątkową architekturę potokową dla użytkowników Chloros+: Wątek 1 (Wykrywanie) ładuje obrazy i wykrywa cele kalibracji, Wątek 2 (Kalibracja) oblicza kalibrację współczynnika odbicia, Wątek 3 (Przetwarzanie) wykonuje przyspieszane przez procesor graficzny usuwanie efektu bayera i obliczanie indeksu, a Wątek 4 (Eksport) zapisuje pliki wyjściowe. W celu uzyskania maksymalnej przepustowości wiele obrazów może znajdować się jednocześnie w różnych wątkach. Szczegółowe informacje można znaleźć w sekcji [Potok przetwarzania](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Jak uruchomić diagnostykę mojej instalacji Chloros?</summary>

Użyj polecenia `selftest`, aby uruchomić 7 testów diagnostycznych systemu, w tym sprawdzenie wersji, dostępność portów, uruchomienie backendu, łączność API, informacje o systemie, modele odszumiania oraz dostępność CUDA:

```bash
chloros-cli selftest
```

Jest to szczególnie przydatne w systemach Linux/Jetson do weryfikacji konfiguracji GPU i CUDA.

</details>
