# Logowanie do Chloros+

## Logowanie przez interfejs graficzny

Menu boczne użytkownika <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> pozwala zalogować się na konto Chloros+ i odblokować dodatkowe funkcje.

**Wystarczy zalogować się tylko raz na każdym komputerze.** Interfejs graficzny (GUI), interfejs wiersza poleceń (CLI) oraz zestaw SDK Python dla języka Python korzystają z tej samej sesji przechowywanej w pamięci podręcznej — zalogowanie się za pomocą interfejsu graficznego na komputerze stacjonarnym aktywuje również interfejs CLI i zestaw SDK na tym komputerze (i odwrotnie za pośrednictwem `chloros-cli login`).

Po zalogowaniu wyświetlą się szczegóły konta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Poziomy planów

| Plan | `plan_id` | Typ |
| --- | --- | --- |
| Iron | `0` | Bezpłatny |
| Copper | `1` | Płatny (Chloros

+) |
| Bronze | `2` | Płatny (Chloros

+) |
| Silver | `3` | Płatny (Chloros

+) |
| Złoto | `4` | Płatny (Chloros

+) |

Zobacz [plany i ceny](https://cloud.mapir.camera/pricing), aby dowiedzieć się, co obejmuje każdy płatny poziom.

### Dostęp do CLI

/SDK

wymaga płatnego poziomu

Dostęp do CLI

i Python

SDK

wymaga **dowolnego płatnego poziomu Chloros

+ (Copper lub wyższego)**. Jest to egzekwowane**po stronie serwera** — każde żądanie CLI

/SDK

musi zawierać zarówno aktywną sesję, jak i płatny plan:

| Status HTTP

| `error_code` | Znaczenie | Rozwiązanie |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Brak logowania na tym komputerze | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Zalogowano się, ale poziom planu jest zbyt niski (bezpłatny poziom Iron) | Przejdź na dowolny płatny plan Chloros

+ |

`chloros-cli status` pozostaje dostępny w ramach bezpłatnego planu, więc zawsze możesz sprawdzić swój aktualny plan i powód odmowy dostępu.

### Limity podłączonego sprzętu w poszczególnych planach

Każdy plan określa maksymalną liczbę kamer LATTICE i czujników światła DAQ, które mogą być podłączone na żywo w tym samym czasie:

| Plan | Kamery LATTICE | Czujniki światła DAQ |
| --- | --- | --- |
| Iron (bezpłatny / bez logowania) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Logowanie do CLI



Zaloguj się przy użyciu danych logowania do Chloros

+, aby włączyć przetwarzanie CLI

. W systemie Linux

(bez GUI) jest to jedyny sposób na aktywację licencji.

**Składnia:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Użytkownicy SDK**: Pakiet Python

SDK

udostępnia również metodę programową `logout()` służącą do czyszczenia danych uwierzytelniających z pamięci podręcznej. Szczegółowe informacje można znaleźć w [dokumentacji SDK

](reference/sdk-reference.md).
{% endhint %}

**Przykład:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Znaki specjalne**: Hasła zawierające znaki takie jak `$`, `!` lub spacje należy ująć w pojedyncze cudzysłowy.
{% endhint %}

**Wynik:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Przechowywanie danych uwierzytelniających

Zbuforowane dane uwierzytelniające i konfiguracja są przechowywane w folderze `.chloros` w katalogu domowym użytkownika na **wszystkich platformach**:

| Platforma | Ścieżka do pamięci podręcznej danych uwierzytelniających |
| --- | --- |
| **System Windows** | `%USERPROFILE%\.chloros\` |
| **System Linux** | `~/.chloros/` |

Zainstalowanie nowszej wersji na istniejącej zachowuje dane logowania w systemie Windows

(instalator) oraz w systemie Linux

(folder `.deb`, w tym `chloros-cli update`). Odinstalowanie programu Chloros

— lub usunięcie pakietu dla systemu Linux

— powoduje wyczyszczenie danych logowania z pamięci podręcznej, więc po nowej instalacji trzeba się zalogować ponownie.

### Wygaśnięcie planu i okres karencji w trybie offline

Data wygaśnięcia planu wyświetlana w interfejsie graficznym wskazuje, kiedy licencja straci ważność. W przypadku comiesięcznych subskrypcji wygaśnięcie następuje z końcem miesiąca; w przypadku subskrypcji rocznych — rok po rozpoczęciu subskrypcji.

Program Chloros

weryfikuje licencję online, ale praca w trybie offline jest obsługiwana w ramach okresu karencji:

* Pomyślne weryfikacje serwerowe są buforowane przez **5 minut**, więc podczas normalnego użytkowania wysyłanych jest bardzo niewiele żądań dotyczących licencji.
* Podpisana, przypisana do konkretnego urządzenia pamięć podręczna licencji obejmuje dłuższe okresy pracy w trybie offline: **30 dni w przypadku planów miesięcznych**oraz**do daty wygaśnięcia subskrypcji (maksymalnie 365 dni) w przypadku planów rocznych**.
* Po upływie okresu karencji plan przechodzi na bezpłatny poziom Iron do momentu, aż urządzenie choć raz połączy się z serwerem licencji; dostęp zostanie przywrócony po kolejnym pomyślnym sprawdzeniu.

### Limit urządzeń

Każdy plan Chloros

+ oferuje inną liczbę zarejestrowanych urządzeń. Każde urządzenie, na którym logujesz się za pomocą konta Chloros

+, wlicza się do liczby zarejestrowanych urządzeń. Możesz zmienić nazwę lub usunąć urządzenie na stronie swojego konta MAPIR

Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Obsługiwane urządzenia</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Dokładna liczba urządzeń dozwolonych na Twoim koncie jest widoczna na stronie konta MAPIR Cloud. Wylogowanie się z urządzenia skutecznie zwalnia jego miejsce, a urządzenie, które zostało już zarejestrowane, zawsze może się ponownie zalogować, nawet jeśli konto osiągnęło limit urządzeń.
