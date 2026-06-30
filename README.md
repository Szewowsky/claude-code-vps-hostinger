# Claude Code na VPS (Hostinger) - dostęp z telefonu i kompa

Postaw **Claude Code** (i opcjonalnie **Codex**) na własnym VPS, żeby działał 24/7, i łącz się z nim z telefonu albo z pulpitu. Koduj z dowolnego miejsca - z kanapy, pociągu, kawiarni.

> Instrukcja towarzysząca filmowi na YouTube. Każda komenda jest w osobnym bloku - kliknij ikonę kopiowania i wklej do terminala. Zamiast `twoj-serwer` wstaw adres swojego VPS.

---

## Co będzie potrzebne

- **VPS** (Hostinger - polecany plan ok. KVM2, min. 4 GB RAM; działa też Hetzner, DigitalOcean itp.)
- **Termius** - darmowy klient SSH na telefon (iOS/Android)
- **Subskrypcja** Claude Max (do Claude Code) lub ChatGPT (do Codex)
- Komputer z terminalem (Mac/Linux) albo PowerShell (Windows)

---

## 1. Postaw VPS

**Najprościej (Hostinger):** przy zakładaniu VPS wybierz szablon aplikacji **"Claude Code"** - narzędzie jest już zainstalowane. Jeśli możesz, od razu wklej swój publiczny klucz SSH w polu **"SSH Keys"** (jak go zdobyć - krok 2).

Na czystym Ubuntu zainstaluj Claude Code natywnym instalatorem (bez Node, bez npm):

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Sprawdź, że działa:

```bash
claude --version
```

Zaloguj się subskrypcją:

```bash
claude
```

Pojawi się link do logowania. W prompcie wciśnij klawisz **`c`** - Claude Code skopiuje pełny link do schowka (długi URL łatwo się urywa przy ręcznym zaznaczaniu w terminalu). Otwórz link w przeglądarce, zatwierdź konto, wróć do terminala.

---

## 2. Zabezpiecz VPS (ważne!)

> **🔑 KOLEJNOŚĆ MA ZNACZENIE:** fail2ban i firewall włącz od razu (niczego nie blokują). Ale **klucze SSH ustaw przed wyłączeniem hasła** - inaczej zostaniesz za drzwiami. Hasło wyłączamy na samym końcu (krok 2f).

### 2a. fail2ban (blokuje ataki brute-force)

Zainstaluj:

```bash
sudo apt install -y fail2ban
```

Sprawdź, że działa (szukaj `active (running)`; wyjdź klawiszem `q`):

```bash
sudo systemctl status fail2ban
```

### 2b. Firewall - przepuść tylko SSH

Włącz firewall i otwórz port SSH:

```bash
sudo ufw allow 22 && sudo ufw enable
```

Sprawdź (szukaj `Status: active` oraz `22 ALLOW`):

```bash
sudo ufw status
```

> Hostinger blokuje niestandardowe porty SSH - **zostaw port 22**.

### 2c. Klucz SSH z komputera

Wygeneruj klucz (Terminal na Mac/Linux, PowerShell na Windows):

```bash
ssh-keygen -t ed25519
```

Naciśnij `Enter` trzy razy (domyślna lokalizacja, bez hasła). Powstają dwa pliki: **prywatny** (`id_ed25519` - sekret, nigdy nie pokazuj) i **publiczny** (`id_ed25519.pub` - ten dajesz serwerowi).

Pokaż i skopiuj publiczny klucz (bezpieczny na ekranie):

```bash
cat ~/.ssh/id_ed25519.pub
```

Wgraj go na serwer jedną komendą (póki masz dostęp):

```bash
ssh-copy-id root@twoj-serwer
```

### 2d. Klucz SSH z telefonu (Termius)

W Termiusie: *Keychain → wygeneruj klucz → przytrzymaj → Export to host* (albo wklej jego publiczny klucz w Hostinger przy tworzeniu VPS).

### 2e. Przetestuj klucze

Zaloguj się z komputera i z telefonu **kluczem** (bez hasła). Dopiero gdy oba wchodzą - przejdź dalej.

### 2f. Wyłącz logowanie hasłem (DOPIERO po teście kluczy z 2e!)

```bash
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf && sudo systemctl restart ssh
```

Sprawdź **z komputera** (nowe okno terminala - NIE z serwera!), że hasło naprawdę nie działa. Ta komenda wymusza próbę logowania **hasłem** (pomija klucz):

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no root@twoj-serwer
```

Powinno **odbić**: `Permission denied (publickey)` - czyli logowanie hasłem jest wyłączone, wchodzi już tylko klucz. ✅

---

## 3. Dostęp z telefonu (Termius)

1. Zainstaluj **Termius**.
2. Dodaj nowy host: adres VPS, użytkownik, Twój klucz SSH.
3. Połącz się.

---

## 4. tmux - sesja, która nie umiera

Bez tego sesja znika, gdy zamkniesz aplikację albo stracisz zasięg. tmux trzyma ją żywą.

Zainstaluj:

```bash
sudo apt install -y tmux
```

Nowa sesja:

```bash
tmux new -s claude
```

Odpal Claude Code w środku:

```bash
claude
```

Wyjdź z sesji (zostaje żywa): naciśnij `Ctrl-b`, a potem `d`.

Wróć do sesji - z dowolnego urządzenia:

```bash
tmux attach -t claude
```

> ⚠️ Sesje tmux są **per-użytkownik**. Sesja odpalona jako `root` jest niewidoczna dla innego użytkownika i odwrotnie. Loguj się tym samym użytkownikiem, który ją utworzył.

---

## 5. Bonus: dostęp z pulpitu (Claude Code - Add SSH connection)

Claude Code (i Codex) na komputerze ma wbudowane **"Add SSH connection"** - odpalasz narzędzie na VPS prosto z aplikacji:

| Pole | Wartość |
|---|---|
| Name | dowolne, np. `Moj VPS` |
| SSH Host | `root@twoj-serwer` |
| SSH Port | puste (domyślnie 22) |
| Identity File | `~/.ssh/id_ed25519` |

> Jeśli zobaczysz **"Host denied / verification failed"** po postawieniu serwera od nowa - Twój komputer pamięta stary "odcisk" serwera. Usuń go i połącz ponownie:
> ```bash
> ssh-keygen -R twoj-serwer
> ```

---

## 6. Bonus: to samo z Codex

Identyczny schemat działa z OpenAI Codex. Różni się tylko logowanie. W ChatGPT włącz *Settings → Security → "Device code authorization"*, potem:

```bash
codex login --device-auth
```

Otwórz pokazany link, wpisz kod - i Codex jest zalogowany na VPS. Resztę (tmux, Termius, pulpit) robisz tak samo.

---

## Najczęstsze pułapki (i fixy)

| Problem | Przyczyna | Fix |
|---|---|---|
| `Invalid OAuth Request: Unknown scope: user:sessions:cl...` | URL logowania urwał się przy kopiowaniu | W prompcie logowania wciśnij **`c`** - skopiuje pełny link |
| `claude: command not found` u nowego użytkownika | Claude Code zainstalowany dla innego użytkownika | Zainstaluj ponownie jako ten użytkownik (instalator z kroku 1) |
| tmux "nie ma sesji" | Sesje są per-użytkownik | Zaloguj się tym samym użytkownikiem, który odpalił sesję |
| Zatrzaśnięcie po wyłączeniu hasła | Klucz nie był przetestowany przed wyłączeniem | Trzymaj starą sesję otwartą, testuj klucz w nowej karcie, dopiero potem wyłączaj hasło |
| Klucz działa na telefonie, ale nie na kompie | Klucz jest per-urządzenie | Dodaj publiczny klucz kompa na serwer osobno |
| `Host denied / verification failed` | Komp pamięta stary odcisk serwera (po reinstalacji VPS) | `ssh-keygen -R twoj-serwer`, połącz ponownie |

---

## Compliance (ważne)

- Claude Code i Codex można uruchamiać na VPS **na subskrypcji**, jeśli to **Ty** sterujesz interaktywnie (a tak jest - piszesz polecenia ręcznie).
- **Klucz API** jest potrzebny dopiero pod pełną automatyzację bez nadzoru (agent jadący sam 24/7).
- Nie udostępniaj konta innym osobom i nie wystawiaj usługi publicznie.
- *Stan na czerwiec 2026 - regulaminy się zmieniają, sprawdź aktualne warunki Anthropic / OpenAI.*

---

## Pełny hardening serwera

To repo pokazuje podstawy. Pełne zabezpieczenie (osobny użytkownik, wyłączenie roota, auto-update, monitoring i więcej - 8 kroków z wizardem):

👉 **https://github.com/Szewowsky/vps-security**

---

## Licencja

MIT - rób z tym, co chcesz.
