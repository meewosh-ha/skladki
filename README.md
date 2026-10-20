# skladki: rejestr składek klasy 1d

Publiczna, jednoplikowa strona HTML (`index.html`) z listą wpłat, zaległościami, wydatkami i saldem klasy 1d. Dane są wpisane wprost w `<script>` i aktualizowane ręcznie.

- Produkcja: https://meewosh-ha.github.io/skladki/ (GitHub Pages, gałąź `main`, katalog `/`)
- Bezpośredni link do wydatków: https://meewosh-ha.github.io/skladki/?view=wydatki
- Repo: https://github.com/meewosh-ha/skladki (publiczne; nie wpisuj tu żadnych sekretów)

## Od czego apka zależy (stan na 2026-10-20)

- Node, Python, menedżer pakietów, build: nic z tego. Nie ma `package.json`, kroku budowania ani zależności.
- Zmienne środowiskowe i sekrety: brak (`.env.example` nie jest potrzebny).
- Zewnętrzne zasoby w czasie działania: tylko czcionki Google Fonts (Fraunces, Inter, IBM Plex Mono) ładowane z `fonts.googleapis.com`. Bez sieci strona działa na czcionkach zastępczych.
- launchd, port, hub constellations, `~/Cowork/hub/bin/deploy`, Notion, dioda w `/status.json`: nie dotyczy. Apka nie ma żadnego zadania `pl.bolechowski.*` ani wpisu w `NAMES` w `hub/server.py`.
- Dane lokalne: brak. Jedynym źródłem danych jest `index.html` w repo.
- `.netlify/` w katalogu roboczym to pozostałość po porzuconej próbie hostingu na Netlify; nieużywana i ignorowana przez git.
- Zapasowa kopia na Synology (Web Station, port 8950, portal "skladki", wystawiona przez Tailscale Funnel `https://storage.tail7837c.ts.net/`) odpowiadała HTTP 200 w dniu sprawdzenia, ale to osobna, ręcznie wgrywana kopia, a nagłówki sugerują, że jest starsza niż produkcja. Adresy `*.ts.net` bywają blokowane przez filtry DNS, dlatego rodzicom wysyłamy link do GitHub Pages. Strony Synology nie sprawdzałem (nie logowałem się na NAS).

## Nowy komputer, krok po kroku

1. Zainstaluj git i GitHub CLI (Homebrew musi już być zainstalowany):

```bash
brew install git gh
```

2. Zaloguj się do GitHuba na konto `meewosh-ha`. Ręcznie: wybierz GitHub.com → HTTPS → przeglądarka, i zaloguj się na to konto:

```bash
gh auth login
```

3. Ustaw tożsamość git (tylko dla tego repo, po sklonowaniu w kroku 4):

```bash
git clone https://github.com/meewosh-ha/skladki.git ~/Cowork/skladki
cd ~/Cowork/skladki
git config user.name "meewosh-ha"
git config user.email "264000484+meewosh-ha@users.noreply.github.com"
```

4. (Opcjonalnie) Podgląd lokalny. Wszystkie wersje macOS mają `python3`, a jeśli nie, `xcode-select --install`:

```bash
cd ~/Cowork/skladki && python3 -m http.server 8130
```

Otwórz http://localhost:8130/index.html?v=1 (parametr `?v=` omija cache przeglądarki).

5. Jeśli GitHub Pages jest wyłączone albo repo odtwarzasz od zera: GitHub → repo `skladki` → Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder `/ (root)` → Save. Albo z terminala:

```bash
gh api -X POST repos/meewosh-ha/skladki/pages -f "source[branch]=main" -f "source[path]=/"
```

6. Sprawdzenie. Strona ma się otwierać i pokazywać aktualną datę w nagłówku:

```bash
gh api repos/meewosh-ha/skladki/pages --jq '.html_url, .status'
curl -s https://meewosh-ha.github.io/skladki/ | grep -o 'updated: "[^"]*"'
```

## Aktualizacja danych

1. Edytuj `index.html`: dopisz wpłaty do `PAYMENTS` lub wydatki do `EXPENSES`.
2. Ustaw `CLASS_INFO.updated` na aktualny czas (Warszawa, format `"YYYY-MM-DDTHH:MM"`).
3. Commit i push na `main`; Pages przebuduje się w ok. minutę.
4. Sprawdź, że push dotarł, porównując hashe (nie ufaj `raw.githubusercontent.com` ani samej stronie, bo przez kilka minut potrafią serwować starą wersję):

```bash
git hash-object index.html
curl -s "https://api.github.com/repos/meewosh-ha/skladki/contents/index.html?ref=main" | python3 -c "import json,sys; print(json.load(sys.stdin)['sha'])"
```

Oba hashe muszą być takie same.

## Model danych (skrót)

- `STUDENTS`: pary `{ id, name }` w kolejności wyświetlania (alfabetycznie po nazwisku). `id` NIE wynika z pozycji. Nowego ucznia wstaw alfabetycznie, ale daj mu nowe, niewykorzystane `id` (kolejne po najwyższym), nigdy nie zmieniaj istniejących `id`, bo odwołuje się do nich `PAYMENTS`.
- `CATEGORIES`: `dueNow` (próg statusu "Opłacone", 150 zł w I semestrze; na starcie II semestru zmień na 300) oraz stałe `semester1Amount` (150 zł; dzieli wpłatę ucznia na pulę I i II semestru). Nie myl tych dwóch pól. Kategoria z `active: true` dostaje własną pigułkę.
- `PAYMENTS`: `{ studentId, categoryId, amount, date }`; `EXPENSES`: `{ description, amount, date, semester: 1|2 }`.
