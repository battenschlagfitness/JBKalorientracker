# JB Fitness Pro

PWA Fitness-App mit lokalem Login, mehreren Profilen, Kalorien-/Proteintracking (automatische Zielberechnung nach Mifflin-St-Jeor), Lebensmittel-Datenbank (1700+ Einträge), Gewichtsverlauf mit Wochen-/Monatsansicht, Backup-Export/-Import, Offline-Modus (Service Worker) und optionaler Cloud-Synchronisation über Firebase.

## Dateien
- `index.html` – die komplette App
- `service-worker.js` – Offline-Cache (network-first fürs HTML, damit Updates ankommen)
- `manifest.json`, `icon-192.png`, `icon-512.png` – PWA-Installations-Assets

## Cloud-Sync mit Firebase einrichten (optional)
Ohne diese Schritte läuft die App rein lokal auf dem Gerät – das ist völlig in Ordnung.
Für geräteübergreifende Synchronisation:

1. Auf https://console.firebase.google.com ein kostenloses Projekt anlegen.
2. **Build → Authentication → Sign-in method → E-Mail/Passwort** aktivieren.
3. **Build → Firestore Database → Datenbank erstellen** (Produktionsmodus).
4. **Projektübersicht → Web-App hinzufügen (</>)**, App registrieren, die `firebaseConfig`-Werte kopieren.
5. In `index.html` oben im Script-Block bei `const firebaseConfig = {...}` apiKey, authDomain, projectId und appId eintragen.
6. Firestore-Sicherheitsregeln so setzen, dass jeder nur sein eigenes Backup liest/schreibt:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /backups/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

Danach erscheint in **Profil → Cloud-Sync** die Anmeldung. Nach dem Login werden Änderungen automatisch hochgeladen; auf einem neuen Gerät lädt „Herunterladen" deine Daten.

## Hinweis
Der lokale Profil-Login (Profilauswahl + optionale PIN) speichert Daten im Browser/localStorage des Geräts. Die Cloud-Anmeldung (Firebase) ist davon getrennt und dient der Synchronisation über mehrere Geräte.
