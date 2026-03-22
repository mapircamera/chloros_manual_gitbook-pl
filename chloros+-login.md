# Logowanie do Chloros+

## Logowanie do Chloros i Chloros (przeglądarka)

Menu boczne <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> w pasku bocznym pozwala zalogować się na konto Chloros+ i odblokować dodatkowe funkcje.

Po zalogowaniu wyświetlą się dane Twojego konta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## Logowanie do CLI

Zaloguj się przy użyciu danych logowania do Chloros+, aby umożliwić przetwarzanie w CLI. W przypadku Linux (bez GUI) jest to jedyny sposób na aktywację licencji.

**Składnia:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Użytkownicy SDK**: Python SDK udostępnia również programową metodę `logout()` do czyszczenia poświadczeń z pamięci podręcznej. Szczegółowe informacje można znaleźć w [dokumentacji Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Przykład:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Znaki specjalne**: Hasła zawierające znaki takie jak `$`, `!` lub spacje należy ująć w pojedyncze cudzysłowy.
{% endhint %}

**Wynik:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Przechowywanie poświadczeń

Buforowane poświadczenia są przechowywane w lokalizacji specyficznej dla platformy:

| Platforma | Ścieżka bufora poświadczeń |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Wygaśnięcie planu

Data wygaśnięcia planu w interfejsie graficznym pokazuje, kiedy Twoja licencja straci ważność. W przypadku cyklicznych subskrypcji miesięcznych wygaśnięcie następuje z końcem miesiąca. W przypadku subskrypcji rocznych jest to rok od momentu rozpoczęcia subskrypcji. Sprawdzanie licencji wymaga comiesięcznego połączenia z Internetem w celu weryfikacji, z 30-dniowym okresem karencji.

### Limit urządzeń

Każdy plan Chloros+ oferuje inną liczbę zarejestrowanych urządzeń. Każde urządzenie, na którym logujesz się za pomocą konta Chloros+, będzie wliczane do liczby zarejestrowanych urządzeń. Możesz zmienić nazwę i usunąć urządzenie na stronie swojego konta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">ZŁOTO</th></tr></thead><tbody><tr><td align="right">Obsługiwane urządzenia</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
