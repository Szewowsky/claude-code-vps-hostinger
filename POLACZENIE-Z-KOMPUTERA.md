# Połączenie z komputera: terminal, aplikacja, edytor

Instrukcja do odcinka **„Jak Połączyć Się z Claude Code na VPS przez SSH (BEZ PROGRAMOWANIA)"**.

Masz już agenta na serwerze i chcesz się do niego dostać z komputera. Ta instrukcja robi jedną rzecz,
która działa w **dwóch z trzech** dróg wejścia: zamienia długą komendę w jedno słowo.

> **Nie masz jeszcze serwera?** Zacznij od [README](README.md) - tam jest cała pierwsza połowa:
> postawienie VPS-a, zabezpieczenie go i klucz SSH. Wróć tutaj, gdy `ssh` z kluczem już Ci działa.

---

## Dwa sposoby, wybierz swój

**A. Nie chcesz w to wchodzić** - skopiuj całą sekcję „Prompt do wklejenia" i wyślij ją swojemu
agentowi (Claude Code, Codex, cokolwiek używasz). Zrobi to za Ciebie i pokaże, co zmienił.

**B. Wolisz zrobić sam** - przejdź do „Krok po kroku". To pięć minut i cztery linijki tekstu.

Obie drogi kończą się w tym samym miejscu.

---

## A. Prompt do wklejenia

Podmień trzy rzeczy w nawiasach kwadratowych na swoje, resztę zostaw:

```text
Skonfiguruj mi skrót do logowania na mój serwer, żebym nie musiał wpisywać całej komendy SSH.

Moje dane:
- adres serwera: [ADRES-ALBO-IP]
- użytkownik: [NAZWA-UZYTKOWNIKA]
- port SSH: [PORT, zwykle 22]
- klucz prywatny: ~/.ssh/id_ed25519

Co masz zrobić:
1. Sprawdź, czy plik ~/.ssh/config istnieje. Jeśli nie, utwórz go i nadaj mu uprawnienia 600.
2. Dopisz na jego początku wpis "Host vps" z HostName, User, Port i IdentityFile z moich danych.
   Jeśli wpis "Host vps" już istnieje, NIE dubluj go - pokaż mi obecny i zapytaj, czy podmienić.
3. Sprawdź uprawnienia mojego klucza prywatnego. Jeśli są szersze niż 600, napraw je.
4. Przetestuj połączenie komendą: ssh -o BatchMode=yes vps "echo OK"
5. Pokaż mi zawartość wpisu, który dodałeś, i wynik testu.

Nie zmieniaj niczego innego w ~/.ssh/config. Nie ruszaj konfiguracji serwera.
```

Agent pokaże Ci, co zrobił. Możesz to przeczytać - to zwykły plik tekstowy, żadnej magii tam nie ma.

---

## B. Krok po kroku

### 1. Otwórz plik konfiguracyjny SSH

```bash
nano ~/.ssh/config
```

Jeśli plik nie istnieje, ta komenda utworzy pusty. To normalne.

### 2. Dopisz cztery linijki

```
Host vps
    HostName 11.22.33.44
    User twoj-uzytkownik
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

> Powyższe to **przykład** - podmień `HostName`, `User` i `Port` na swoje. Adres i użytkownika masz
> w panelu hostingu, port to `22`, chyba że sam go zmieniałeś.

Co znaczy każda linijka:

| Linijka | Co wpisać |
|---|---|
| `Host` | **nazwa skrótu**, dowolna. Wpiszesz ją potem zamiast całego adresu |
| `HostName` | adres albo IP Twojego serwera |
| `User` | użytkownik na serwerze (**nie root**, jeśli zrobiłeś hardening z README) |
| `Port` | port SSH. Zostaw `22`, jeśli go nie zmieniałeś |
| `IdentityFile` | ścieżka do Twojego klucza prywatnego na komputerze |

Zapisz: `Ctrl+O`, `Enter`, potem `Ctrl+X`.

### 3. Ustaw uprawnienia (jeśli plik był nowy)

```bash
chmod 600 ~/.ssh/config
```

### 4. Sprawdź

```bash
ssh vps
```

Powinieneś być na serwerze. Od teraz to jedno słowo zastępuje `ssh -p 2222 uzytkownik@adres`.

---

## Aplikacja desktopowa Claude Code

Tu alias się zwraca po raz drugi.

1. W aplikacji, przy polu do wpisywania promptu, rozwiń menu środowiska (jest tam `Local`, `Cloud`).
2. Wybierz **+ Add SSH connection**.
3. Wypełnij **jedno** pole:

| Pole | Wartość |
|---|---|
| Name | dowolna etykieta, np. `Moj VPS` |
| **SSH Host** | **`vps`** - sam skrót, który przed chwilą utworzyłeś |
| SSH Port | **zostaw puste** - weźmie się z `~/.ssh/config` |
| Identity File | **zostaw puste** - też weźmie się z configu |

4. Wybierz połączenie z listy.

**Przy pierwszym połączeniu aplikacja sama instaluje Claude Code na serwerze.** Nie musisz nic
stawiać na Linuksie ręcznie. Autoryzacja idzie przez aplikację na Twoim komputerze, więc serwer nie
potrzebuje przeglądarki.

Po lewej stronie zobaczysz drzewo plików. **To nie jest Twój dysk - to serwer.** Klikasz plik,
otwiera się, edytujesz, zapisujesz, i zostaje tam.

---

## Czego się nie spodziewać (żeby Cię nie zaskoczyło)

| Rzecz | Jak jest naprawdę |
|---|---|
| Wbudowany terminal w aplikacji | Działa **tylko w sesjach lokalnych**. W sesji SSH go nie ma - zostaje panel plików |
| Twoje skille i wtyczki | Sesja SSH czyta je z katalogu domowego **serwera**, nie z Twojego komputera. Tam ich po prostu nie ma |
| Codex | Czyta ten sam `~/.ssh/config`, **ale nie instaluje się sam** - musi już być na serwerze. Wzorce typu `Host *` są przez niego ignorowane, użyj konkretnej nazwy |
| Przeniesienie sesji do przeglądarki | Dla sesji SSH niedostępne |

---

## Sesja, która nie umiera

Zamkniesz okno terminala i praca agenta ginie razem z nim. Dlatego na serwerze uruchamiaj go
wewnątrz `tmux`:

```bash
ssh vps
tmux new -s praca
claude
```

Wychodzisz bez zabijania sesji: `Ctrl+B`, potem `D`.
Wracasz później: `ssh vps` i `tmux attach -t praca`.

To jest wirtualne biurko, które zostaje włączone, kiedy Ty stamtąd wychodzisz.

---

## Najczęstsze potknięcia

| Objaw | Przyczyna i fix |
|---|---|
| `Host denied` / `verification failed` w aplikacji | Komputer pamięta stary odcisk serwera po jego odtworzeniu. `ssh-keygen -R [adres-serwera]`, potem połącz ponownie |
| `WARNING: UNPROTECTED PRIVATE KEY FILE` | Klucz prywatny ma za szerokie uprawnienia. `chmod 600 ~/.ssh/id_ed25519` |
| Komendy dotykające `/root` nie działają | Jesteś zwykłym użytkownikiem i tak ma być. Opakuj w `sudo` |
| Agent nie może postawić czegoś na porcie 80 | Porty poniżej 1024 są zablokowane dla zwykłego użytkownika. Użyj 3000, 8080 i podobnych |
| Link autoryzacyjny urywa się w terminalu | Wciśnij `c`, żeby skopiować pełny adres |

---

## Bezpieczeństwo

Ta instrukcja zakłada, że serwer masz już zabezpieczony (zwykły użytkownik zamiast roota, logowanie
kluczem, firewall). Jeśli nie - zrób to **przed** wpuszczeniem tam agenta:
[github.com/Szewowsky/vps-security](https://github.com/Szewowsky/vps-security)
