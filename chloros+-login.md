# Chloros+ Logowanie

## Chloros i Chloros (przeglądarka) Logowanie

Menu <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> na pasku bocznym umożliwia zalogowanie się na konto Chloros+ i odblokowanie dodatkowych funkcji.

Po zalogowaniu się wyświetlą się szczegóły konta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Logowanie

Zaloguj się przy użyciu danych logowania Chloros+, aby włączyć przetwarzanie CLI.

**Składnia:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**Użytkownicy SDK**: Python SDK udostępnia również programową metodę `logout()` do czyszczenia danych uwierzytelniających z pamięci podręcznej. Szczegółowe informacje można znaleźć w [dokumentacji Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Przykład:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Znaki specjalne**: Użyj pojedynczych cudzysłowów wokół haseł zawierających znaki takie jak `$`, `!` lub spacje.
{% endhint %}

**Wynik:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Wygaśnięcie planu

Wygaśnięcie planu w GUI pokazuje, kiedy licencja straci ważność. W przypadku cyklicznych subskrypcji miesięcznych wygaśnięcie następuje z końcem miesiąca. W przypadku subskrypcji rocznych wygaśnięcie następuje po upływie roku od rozpoczęcia subskrypcji. Sprawdzanie licencji wymaga comiesięcznego połączenia z Internetem w celu weryfikacji, z 30-dniowym okresem karencji.

### Limit urządzeń

Każdy plan Chloros+ oferuje inną liczbę zarejestrowanych urządzeń. Każde urządzenie, na które logujesz się za pomocą konta Chloros+, będzie wliczane do liczby zarejestrowanych urządzeń. Możesz zmienić nazwę i usunąć urządzenie na stronie konta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros</th><th align="center">COPPER</th><th align="center">BRĄZ</th><th align="center">SREBRO</th><th align="center">ZŁOTO</th></tr></thead><tbody><tr><td align="right">Obsługiwane urządzenia</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
