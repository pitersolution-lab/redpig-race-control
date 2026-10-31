# RACE CONTROL 24H — RED PIG

Gotowa aplikacja webowa zespołu Mercedes W140 **RED PIG**, Silesia Ring Pętla Wschodnia+ (3,140 km). React + Vite + Supabase (Postgres, Auth, Realtime), hosting Vercel.

## Funkcje

- **RACE CONTROL** — jedna zakładka wprowadzania danych: stint, analogowy wskaźnik paliwa, PIT, tankowanie, checklisty, pomiary/rotacja/wymiana opon i zdarzenia.
- **Plan LIVE** — automatyczne przeliczanie *pozostałej* części 24 h na podstawie zapisanych stintów, czasu PIT, strat czasu i ustawień.
- **Stinty** — historia i średnie tempo każdego kierowcy.
- **Opony** — pomiary, identyfikatory opon, operacje i historia.
- **PIT** — czas postojów, paliwo, kontrole.
- **Bilans** — czas, dystans, spalanie i prognozy.
- **Zdarzenia** — chronologiczny dziennik.
- **Ustawienia** — strategia, kierowcy, zaproszenia i eksport JSON.
- Konta e-mail/hasło, zespół z kodem zaproszenia, role owner/editor/viewer, RLS, Postgres Realtime.

## 1. Utwórz Supabase

1. Zarejestruj się na https://supabase.com i utwórz nowy projekt.
2. Otwórz **SQL Editor → New query**, wklej zawartość `supabase/schema.sql` i wykonaj (**Run**). Skrypt tworzy tabele, polityki RLS, bezpieczne funkcje zakładania/dołączania do zespołu i włącza Realtime.
3. W **Authentication → Providers → Email** włącz logowanie e-mail/hasło. Zdecyduj, czy użytkownicy mają potwierdzać e-mail (zalecane).
4. W **Project Settings → API** skopiuj **Project URL** i **publishable/anon key**. Używaj tylko klucza publicznego. **NIGDY nie wstawiaj `service_role` ani secret key do aplikacji!**
5. W **Authentication → URL Configuration** dodaj adres docelowy Vercel jako **Site URL** i **Redirect URL** (np. `https://twoja-aplikacja.vercel.app/**`). Do testów lokalnych dodaj `http://localhost:5173/**`.

## 2. Uruchom lokalnie

Wymagany Node.js 20+.

```bash
npm install
cp .env.example .env
```

W `.env` ustaw:

```env
VITE_SUPABASE_URL=https://TWOJ_PROJEKT.supabase.co
VITE_SUPABASE_ANON_KEY=TWÓJ_KLUCZ_PUBLICZNY
```

Uruchom:

```bash
npm run dev
```

Otwórz http://localhost:5173. Zarejestruj konto, zaloguj się, utwórz zespół i przekaż kod zaproszenia współpracownikom.

## 3. Publikacja na Vercel

1. Utwórz repozytorium GitHub i wgraj pliki projektu (bez `.env` i `node_modules`).
2. Zaloguj się na https://vercel.com → **Add New → Project** → importuj repozytorium.
3. Framework: **Vite**, Build Command: `npm run build`, Output Directory: `dist`.
4. W **Environment Variables** dodaj `VITE_SUPABASE_URL` i `VITE_SUPABASE_ANON_KEY` z Supabase.
5. Kliknij **Deploy**. Vercel poda publiczny adres `https://...vercel.app`.
6. Uzupełnij w Supabase **Authentication → URL Configuration** adres aplikacji i dopuszczone adresy przekierowań. Po zmianie zmiennych środowiskowych wykonaj redeploy.
7. Opcjonalnie podłącz własną domenę w **Vercel → Project → Settings → Domains**.

## 4. Zespół i uprawnienia

- Osoba tworząca zespół ma rolę `owner`.
- Użytkownik z kodem zaproszenia dołącza jako `editor`. **Kod jest sekretem dającym możliwość edycji danych.**
- Role `viewer` i `editor` można nadać w Supabase SQL Editor jako administrator projektu:

```sql
-- Zamień identyfikatory na rzeczywiste UUID z tabeli race_members.
update public.race_members
set role = 'viewer'
where team_id = 'UUID_ZESPOLU' and user_id = 'UUID_UZYTKOWNIKA';
```

W tej wersji interfejs nie zawiera administracji rolami ani resetowania kodu zaproszenia. Do obsługi kont i ról służy panel Supabase. Nie publikuj kodu zaproszenia na publicznych stronach.

## 5. Ważne ograniczenia przed wyścigiem

- **Wymaga internetu do zapisu**. Przy utracie połączenia aplikacja wyraźnie blokuje zapis. Nie ma kolejki offline ani gwarancji zapisu po zamknięciu strony. Zapewnij hotspot zapasowy.
- **Czas nie płynie sam**: plan jest liczony z zapisanych stintów, PIT i zdarzeń. Brak live timing i telemetrii.
- **Paliwo**: 1/1 = 95 l, 1/2 = 50 l, 1/4 ≈ 25 l, 3/4 ≈ 72,5 l. Korekty `+/-` są interpolacją; nie są pomiarem litrów. Do strategii tankowania używany jest *ustawiony* wskaźnik spalania, domyślnie 19 l/100 km, a nie wahania analogowego wskaźnika.
- Po tankowaniu następny odczyt `PRZED` jest ustawiany do najbliższego znacznika. Operator **musi skontrolować i poprawić go według rzeczywistej wskazówki** przed zapisaniem kolejnego stintu.
- Prognozy są orientacyjne; uwzględniają średnie tempo i zadany czas PIT, nie uwzględniają ruchu, neutralizacji w przyszłości ani indywidualnego zmęczenia kierowcy.
- Zapisy stintów, PIT i zdarzeń są append-only. Pomyłki należy korygować w bazie z uprawnieniami administratora po wykonaniu kopii danych; aplikacja nie ma funkcji edycji/wycofania wpisów.
- Przed wydarzeniem wykonaj próbę równoczesnego zapisu na kilku urządzeniach i sprawdź polityki Supabase, realtime, logowanie, tankowanie, harmonogram, brak internetu i kopie zapasowe.
- Zaplanuj retencję i backupy bazy danych odpowiednie do znaczenia danych.

## Kontrola jakości

```bash
npm install
npm run build
```

Projekt jest dostarczony jako kod źródłowy do samodzielnej publikacji; nie został wdrożony na konto użytkownika.
