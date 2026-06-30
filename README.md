# Claude Code na VPS (Hostinger) - dostęp z telefonu i kompa

Postaw **Claude Code** (i opcjonalnie **Codex**) na własnym VPS, żeby działał 24/7, i łącz się z nim z telefonu albo z pulpitu. Koduj z dowolnego miejsca - z kanapy, pociągu, kawiarni.

> Instrukcja towarzysząca filmowi na YouTube. Wszystkie komendy są gotowe do skopiowania. Zamiast `twoj-serwer` wstaw swój adres VPS.

---

## Co będzie potrzebne

- **VPS** (Hostinger - polecany plan ok. KVM2, min. 4 GB RAM; działa też Hetzner, DigitalOcean itp.)
- **Termius** - darmowy klient SSH na telefon (iOS/Android)
- **Subskrypcja** Claude Max (do Claude Code) lub ChatGPT (do Codex)
- Komputer z terminalem (Mac/Linux) albo PowerShell (Windows)

---

## 1. Postaw VPS

**Najprościej (Hostinger):** przy zakładaniu VPS wybierz szablon aplikacji **"Claude Code"** - narzędzie jest już zainstalowane.

**Na czystym Ubuntu** (dowolny VPS) zainstalujesz Claude Code natywnym instalatorem - bez Node, bez npm:
```bash
curl -fsSL https://claude.ai/install.sh | bash
```
Sprawdź, że działa:
```bash
claude --version
```

---

## 2. Klucze SSH (zrób raz na komputerze)

Klucz SSH to para: **prywatny** zostaje na Twoim urządzeniu (sekret, nigdy go nie pokazuj) i **publiczny**, który dajesz serwerowi (jak zamek w drzwiach).

**Krok 1 - wygeneruj klucz** w Terminalu (Mac/Linux) lub PowerShell (Windows):
```bash
ssh-keygen -t ed25519
```
Naciśnij Enter trzy razy (domyślna lokalizacja, bez hasła).

**Krok 2 - pokaż i skopiuj swój publiczny klucz** (bezpieczny do pokazania):
```bash
cat ~/.ssh/id_ed25519.pub
```

**Krok 3 - wgraj klucz na serwer:**
- Najczyściej: przy tworzeniu VPS w panelu Hostinger wklej go w polu **"SSH Keys"**.
- Na działającym serwerze (póki masz jeszcze dostęp): `ssh-copy-id root@twoj-serwer`

> **🔑 ZŁOTA ZASADA:** wgraj klucze **wszystkich** urządzeń (komp + telefon) i sprawdź, że się logują, **ZANIM wyłączysz hasło** (krok 6). Urządzenie bez klucza zostanie za drzwiami na zawsze.

**Telefon (Termius):** w aplikacji wejdź w *Keychain → wygeneruj klucz → przytrzymaj → Export to host* (albo wklej jego publiczny klucz w Hostinger przy tworzeniu VPS).

---

## 3. Zaloguj Claude Code (subskrypcją)

```bash
claude
```
Pojawi się link do logowania. **Wskazówka:** w prompcie logowania wciśnij **`c`** - Claude Code sam skopiuje pełny link do schowka (długi URL łatwo się urywa przy ręcznym kopiowaniu w terminalu). Otwórz link w przeglądarce, zatwierdź konto, wróć do terminala.

---

## 4. tmux - sesja, która nie umiera

Bez tego sesja znika, gdy zamkniesz aplikację albo stracisz zasięg. tmux trzyma ją żywą:
```bash
sudo apt install -y tmux
tmux new -s claude     # nowa sesja
claude                 # odpalasz Claude Code w środku
# Ctrl-b, potem d      # "detach" - wychodzisz, sesja zostaje
```
Powrót do sesji (z dowolnego urządzenia):
```bash
tmux attach -t claude
```

> ⚠️ Sesje tmux są **per-użytkownik**. Sesja odpalona jako `root` jest niewidoczna dla innego użytkownika i odwrotnie. Loguj się tym samym użytkownikiem, który ją utworzył.

---

## 5. Dostęp z telefonu (Termius)

1. Zainstaluj **Termius**.
2. Dodaj nowy host: adres VPS, użytkownik, Twój klucz SSH.
3. Połącz się i wpisz `tmux attach -t claude` - jesteś dokładnie w tej samej sesji.

---

## 6. Dostęp z pulpitu (Claude Code - Add SSH connection)

Claude Code (i Codex) na komputerze ma wbudowane **"Add SSH connection"** - odpalasz narzędzie na VPS prosto z aplikacji:

| Pole | Wartość |
|---|---|
| Name | dowolne, np. `Moj VPS` |
| SSH Host | `root@twoj-serwer` (lub `user@twoj-serwer`) |
| SSH Port | puste (domyślnie 22) |
| Identity File | `~/.ssh/id_ed25519` |

> Jeśli zobaczysz **"Host denied / verification failed"** po postawieniu serwera od nowa - Twój komputer pamięta stary "odcisk" serwera. Usuń go: `ssh-keygen -R twoj-serwer` i połącz ponownie (albo połącz w nowym oknie).

---

## 7. Zabezpiecz serwer (podstawy)

Minimum, zanim wystawisz serwer na świat:

```bash
# fail2ban - blokuje ataki brute-force
sudo apt install -y fail2ban

# firewall - przepuść tylko SSH
sudo ufw allow 22 && sudo ufw enable

# wyłącz logowanie hasłem (TYLKO po teście klucza! - patrz ZŁOTA ZASADA)
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' \
  /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf && \
  sudo systemctl restart ssh
```

> Hostinger blokuje niestandardowe porty SSH - **zostaw port 22**.
>
> Pełny hardening (8 kroków: osobny użytkownik, fail2ban, UFW, wyłączenie roota, auto-update i więcej) → osobne repo z wizardem: **https://github.com/Szewowsky/vps-security**

---

## 8. Bonus: to samo z Codex

Identyczny schemat działa z OpenAI Codex. Różni się tylko logowanie:
```bash
# w ChatGPT: Settings → Security → włącz "Device code authorization"
codex login --device-auth
```
Otwórz pokazany link, wpisz kod - i Codex jest zalogowany na VPS. Resztę (tmux, Termius, desktop) robisz tak samo.

---

## Najczęstsze pułapki (i fixy)

| Problem | Przyczyna | Fix |
|---|---|---|
| `Invalid OAuth Request: Unknown scope: user:sessions:cl...` | URL logowania urwał się przy kopiowaniu | W prompcie logowania wciśnij **`c`** - skopiuje pełny link |
| `claude: command not found` u nowego użytkownika | Claude Code zainstalowany dla innego użytkownika | Zainstaluj ponownie jako ten użytkownik (instalator z kroku 1) |
| tmux "nie ma sesji" | Sesje są per-użytkownik | Zaloguj się tym samym użytkownikiem, który odpalił sesję |
| Zatrzaśnięcie po wyłączeniu hasła | Klucz nie był przetestowany przed `PasswordAuthentication no` | Trzymaj starą sesję otwartą, testuj klucz w nowej karcie, dopiero potem wyłączaj hasło |
| Klucz działa na telefonie, ale nie na kompie | Klucz jest per-urządzenie | Dodaj publiczny klucz kompa na serwer osobno |
| `Host denied / verification failed` | Komp pamięta stary odcisk serwera (po reinstalacji VPS) | `ssh-keygen -R twoj-serwer`, połącz ponownie |

---

## Compliance (ważne)

- Claude Code i Codex można uruchamiać na VPS **na subskrypcji**, jeśli to **Ty** sterujesz interaktywnie (a tak jest - piszesz polecenia ręcznie).
- **Klucz API** jest potrzebny dopiero pod pełną automatyzację bez nadzoru (agent jadący sam 24/7).
- Nie udostępniaj konta innym osobom i nie wystawiaj usługi publicznie.
- *Stan na czerwiec 2026 - regulaminy się zmieniają, sprawdź aktualne warunki Anthropic / OpenAI.*

---

## Licencja

MIT - rób z tym, co chcesz.
