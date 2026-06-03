# JB Fitness Pro

PWA Fitness-App mit lokalem Login, mehreren Profilen, Kalorien-/Proteintracking (automatische Zielberechnung nach Mifflin-St-Jeor), Lebensmittel-Datenbank (1700+ Einträge), Gewichtsverlauf mit Wochen-/Monatsansicht, Backup-Export/-Import, Offline-Modus (Service Worker) und optionaler Cloud-Synchronisation über Supabase (funktioniert identisch auf iPhone und Android).

## Dateien
- `index.html` – die komplette App
- `service-worker.js` – Offline-Cache (network-first fürs HTML, damit Updates ankommen)
- `manifest.json`, `icon-192.png`, `icon-512.png` – PWA-Installations-Assets

## Cloud-Sync mit Supabase einrichten (optional)
Ohne diese Schritte läuft die App rein lokal auf dem Gerät – das ist völlig in Ordnung.
Für Synchronisation zwischen mehreren Geräten (iOS & Android):

1. Auf https://supabase.com ein kostenloses Projekt anlegen.
2. **Project Settings → API**: „Project URL" und „anon public" Key kopieren.
3. In `index.html` oben im Script-Block bei `const supabaseConfig = {...}` `url` und `anonKey` eintragen.
4. **Authentication → Providers → Email** ist standardmäßig aktiv. Für sofortige Anmeldung ohne Bestätigungs-Mail optional **Authentication → Providers → Email → "Confirm email" deaktivieren** (sonst muss die E-Mail erst bestätigt werden). Später lässt sich hier auch „Sign in with Apple" oder Google ergänzen.
5. Im **SQL Editor** dieses Skript ausführen (Tabelle + Sicherheitsregeln, sodass jeder nur seine eigenen Daten sieht):

```sql
create table if not exists public.backups (
  user_id uuid primary key references auth.users on delete cascade,
  data jsonb,
  updated timestamptz default now()
);

alter table public.backups enable row level security;

create policy "own backup - select" on public.backups
  for select using (auth.uid() = user_id);
create policy "own backup - insert" on public.backups
  for insert with check (auth.uid() = user_id);
create policy "own backup - update" on public.backups
  for update using (auth.uid() = user_id);
```

Danach erscheint in **Profil → ☁️ Cloud-Sync** die Anmeldung. Nach dem Login werden Änderungen automatisch hochgeladen; auf einem neuen Gerät lädt „Herunterladen" deine Daten.

## Hinweise
- Der lokale Profil-Login (Profilauswahl + optionale PIN) speichert Daten im localStorage des Geräts. Die Cloud-Anmeldung (Supabase) ist davon getrennt und dient der Synchronisation über mehrere Geräte.
- Anmelden auf einem neuen Gerät ersetzt die lokalen Daten durch das Cloud-Backup.
- Das Supabase-SDK lädt nur online. Offline funktioniert die App über den Service Worker und localStorage normal weiter; die Synchronisation läuft, sobald wieder Verbindung besteht.
