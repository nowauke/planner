# ADHD Planner — Plan działania (Symfony + Docker + React PWA)

Ten dokument opisuje plan budowy aplikacji do planowania dla osób z ADHD / “słomianym zapałem”.
Na tym etapie skupiamy się wyłącznie na działaniu aplikacji (bez monetyzacji).

---

## Cele projektu

- Full-stack: **Symfony (API)** + **React (PWA)**
- Nauka: **Docker + docker-compose**, API-first, auth, powiadomienia, offline/PWA
- MVP, które **realnie da się używać na co dzień**
- Podejście iteracyjne: małe kroki, częsty “działa na prodzie”

---

## Założenia (na start)

- Aplikacja webowa jako **PWA** (instalowalna na telefonie)
- Front: React + TypeScript
- Backend: Symfony (API)
- Baza: PostgreSQL
- Powiadomienia: startowo **in-app + e-mail** (push w kolejnej iteracji)
- Auth: **session + HttpOnly cookie** (rekomendowane na start; mniej bólu niż JWT)

---

## MVP — definicja “działa”

MVP jest gotowe, jeśli:
- użytkownik może się zarejestrować / zalogować
- może dodać zadanie w 2–3 sekundy (Quick Add / Inbox)
- ma widok “Dziś” i listę “Inbox”
- może ustawić termin i przypomnienie
- aplikacja działa wygodnie na telefonie (PWA, responsywność)
- podstawowe dane są bezpieczne (autoryzacja + izolacja użytkowników)

---

## Roadmap (etapy)

### Etap 0 — Repo + narzędzia (0.5–1 dzień)
**Cel:** projekt wstaje lokalnie jednym poleceniem.

Checklist:
- [ ] Struktura repo:
  - `/backend` (Symfony)
  - `/frontend` (React)
  - `/infra` (nginx, docker configs)
  - `docker-compose.yml`
- [ ] Makefile / skrypty (opcjonalnie, ale polecane):
  - `make up`, `make down`, `make backend`, `make frontend`, `make logs`
- [ ] Ustalona konwencja branchy i commitów (np. conventional commits)

---

### Etap 1 — Docker + “Hello World” (1–2 dni)
**Cel:** docker-compose, nginx, php-fpm, postgres, node. Symfony i React działają.

Backend:
- [ ] Symfony uruchamia się w kontenerze (php-fpm)
- [ ] Nginx reverse proxy pod backend
- [ ] Połączenie z PostgreSQL (ENV + healthcheck)
- [ ] Migrations działają z kontenera

Frontend:
- [ ] React + TS (Vite) w kontenerze dev (hot reload)
- [ ] Proxy API (dev) lub CORS skonfigurowany

Done:
- [ ] `GET /api/health` zwraca `{status: "ok"}`
- [ ] Front wyświetla “OK” po fetchu z backendu

---

### Etap 2 — Model danych + CRUD zadań (2–4 dni)
**Cel:** podstawowe zadania, przypięte do użytkownika, działające listy.

Model (propozycja):
- **User**
- **Task**
  - id
  - user_id (właściciel)
  - title (string)
  - note (text, nullable)
  - due_at (datetime, nullable)
  - status (enum: `open`, `done`, `archived`)
  - priority (int 1–3, nullable)
  - created_at, updated_at
- (opcjonalnie później) Tag, Project, Habit

Backend:
- [ ] Encja Task + migracje
- [ ] Endpointy:
  - `POST /api/tasks` (create)
  - `GET /api/tasks?filter=...` (list)
  - `PATCH /api/tasks/{id}` (update)
  - `DELETE /api/tasks/{id}` (delete/soft-delete)
- [ ] Walidacja (Symfony Validator)
- [ ] Autoryzacja: user widzi tylko swoje taski

Frontend:
- [ ] Widoki:
  - Inbox (wszystkie otwarte bez terminu lub z dowolnym)
  - Today (due dziś + “overdue” opcjonalnie)
- [ ] Dodawanie i oznaczanie jako done
- [ ] Podstawowe UX:
  - szybkie dodanie (input na górze)
  - minimalna liczba klików

Done:
- [ ] Da się używać: dodaj → zobacz → oznacz jako done

---

### Etap 3 — Auth (1–3 dni)
**Cel:** pełne logowanie/rejestracja i ochrona endpointów.

Backend:
- [ ] Rejestracja
- [ ] Logowanie (session, HttpOnly cookie)
- [ ] Wylogowanie
- [ ] Rate limit / proste zabezpieczenia (minimum)

Frontend:
- [ ] Ekrany login/register
- [ ] Persist sesji
- [ ] Guard routes (niezalogowany -> login)

Done:
- [ ] Po odświeżeniu strony użytkownik pozostaje zalogowany

---

### Etap 4 — ADHD-UX: Quick Capture + “Next action” (2–5 dni)
**Cel:** aplikacja zaczyna wyróżniać się “pod ADHD”.

Funkcje:
- [ ] **Quick Capture**: dodanie zadania jednym polem (np. “Dentysta jutro 15:00” → na razie bez NLP, tylko szybki input)
- [ ] **Inbox** jako “zrzut myśli”
- [ ] Widok “Next” / “Teraz”:
  - pokazuj 1–3 najważniejsze rzeczy, nie 50
- [ ] Minimalistyczny UI (anti-overwhelm)

Done:
- [ ] Użytkownik może szybko wrzucić 10 myśli i potem je ogarnąć

---

### Etap 5 — Przypomnienia (wersja 1) (2–6 dni)
**Cel:** realne przypominanie, ale bez push (na razie).

Model:
- **Reminder**
  - id
  - task_id
  - remind_at (datetime)
  - channel (enum: `in_app`, `email`)
  - status (enum: `scheduled`, `sent`, `canceled`)
  - created_at

Backend:
- [ ] Messenger + worker
- [ ] Komenda/cron do odpalania dispatch (albo transport, który obsłuży opóźnienia)
- [ ] Endpointy dla reminderów:
  - `POST /api/tasks/{id}/reminders`
  - `DELETE /api/reminders/{id}`
- [ ] In-app: “Notifications” endpoint (lista)

Frontend:
- [ ] UI do ustawienia przypomnienia (np. dropdown: 10 min / 1h / jutro rano / custom)
- [ ] “Notifications” ekran (in-app)
- [ ] Badge/licznik (opcjonalnie)

Done:
- [ ] Przypomnienie pojawia się w appce + (opcjonalnie) przychodzi mail

---

### Etap 6 — PWA (1–3 dni)
**Cel:** instalowalna web-app, szybka i “mobilna”.

- [ ] Manifest + ikony
- [ ] Service worker (cache shell)
- [ ] Offline read (ostatnio pobrane zadania)
- [ ] Add to Home Screen flow (instrukcja w UI)

Done:
- [ ] Da się zainstalować na telefonie i uruchamia się jak appka

---

### Etap 7 — Jakość i stabilność (ciągle, minimum 2–4 dni)
**Cel:** mniej bugów, lepsza pewność zmian.

Backend:
- [ ] Testy jednostkowe dla logiki (minimum)
- [ ] Testy integracyjne dla endpointów (happy path)
- [ ] CI (lint + testy)

Frontend:
- [ ] ESLint + prettier
- [ ] Proste testy komponentów (opcjonalnie)

Observability:
- [ ] Logi sensowne w dockerze
- [ ] (opcjonalnie) Sentry w przyszłości

---

## Backlog “po MVP” (kolejne iteracje)

### Push notifications (PWA)
- [ ] Web Push (VAPID)
- [ ] Wybór kanałów: push vs email vs in-app
- [ ] Ustawienia quiet hours

### Habits (nawyki) — w wersji ADHD-friendly
- [ ] Zamiast “streak 30/30” → elastyczne “5/7 dni”
- [ ] “Zasada minimum” (np. 1 minuta to też sukces)
- [ ] Mikro-cele i świętowanie postępu

### Time blindness tools
- [ ] Timeboxing (Start 5 min)
- [ ] Pomodoro (krótsze, adaptacyjne)
- [ ] Wizualne odliczanie

### Kalendarz / plan dnia
- [ ] Widok tygodnia
- [ ] Blokowanie czasu (time blocks)
- [ ] Eksport / import ICS (opcjonalnie)

### React Native (po dopięciu web)
- [ ] Wspólne modele i API client
- [ ] Auth flow mobile
- [ ] Powiadomienia natywne

---

## Konwencje i standardy

### API
- JSON
- Spójne błędy walidacji
- Pagination na listach (gdy zajdzie potrzeba)
- Filterowanie po statusie, due_at

### Bezpieczeństwo (minimum)
- Każdy rekord powiązany z userem
- Autoryzacja w warstwie backendu (nie tylko po stronie frontu)
- HttpOnly cookies dla sesji
- CSRF (jeśli wymagane przez flow)

### UX pod ADHD
- Mało bodźców
- Minimum opcji na ekranie
- “Jedna rzecz teraz” (Next)
- Szybkie dodawanie i proste domyślne ustawienia

---

## Proponowane “Definition of Done” dla każdego taska

- [ ] Działa lokalnie w Dockerze
- [ ] Ma endpointy / UI + walidacje
- [ ] Ma podstawowe testy (jeśli dotyczy logiki)
- [ ] Nie psuje istniejących flow
- [ ] Krótki opis w README / changelog (jeśli większa zmiana)

---

## Notatki techniczne (do uzupełnienia w repo)

Miejsca na wpisanie konkretnych komend po dodaniu Docker/Makefile:

- `make up`
- `make backend`
- `make frontend`
- `make migrate`
- `make worker`

---

## Pierwsze kroki (najbliższe 5 zadań)

1. [ ] Utwórz monorepo: `/backend`, `/frontend`, `docker-compose.yml`
2. [ ] Postaw kontenery: nginx + php-fpm + postgres + node
3. [ ] Zrób `GET /api/health`
4. [ ] Zrób React view, który odpytuje `/api/health`
5. [ ] Dodaj encję Task + listę “Inbox” w UI

---

## Kryterium sukcesu projektu

Po 4–6 tygodniach:
- korzystasz z aplikacji codziennie do swoich zadań
- masz pełny stack w dockerze
- możesz pokazać projekt jako portfolio (GitHub + demo)
