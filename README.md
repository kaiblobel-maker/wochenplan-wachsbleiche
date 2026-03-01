# Wochenplan Team Wachsbleiche

Kollaborative Wochenplanung für Vermögensberater-Teams. Built with Next.js 14, Supabase, TailwindCSS.

---

## 📁 Repo-Struktur

```
wochenplan-team-wachsbleiche/
├── .github/
│   └── workflows/ci.yml
├── public/
│   └── manifest.json
├── src/
│   ├── app/
│   │   ├── (app)/                    # Auth-protected routes
│   │   │   ├── layout.tsx            # App shell mit Sidebar
│   │   │   ├── dashboard/page.tsx    # Redirect je nach Rolle
│   │   │   ├── me/page.tsx           # Eigene Wocheneingabe
│   │   │   ├── team/page.tsx         # Admin: Teamübersicht
│   │   │   └── users/
│   │   │       ├── page.tsx
│   │   │       └── UserList.tsx      # Client-Component
│   │   ├── api/
│   │   │   └── create-user/route.ts  # Admin: User erstellen
│   │   ├── login/page.tsx
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   └── MobileHeader.tsx
│   │   ├── MetricsForm.tsx
│   │   ├── WeekSelector.tsx
│   │   └── StatCard.tsx
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts
│   │   │   ├── server.ts
│   │   │   └── middleware.ts
│   │   ├── utils/
│   │   │   ├── week.ts
│   │   │   └── cn.ts
│   │   └── validations.ts
│   ├── types/index.ts
│   └── middleware.ts
├── supabase/
│   └── migrations/
│       ├── 001_initial.sql
│       └── 002_seed_admin.sql
├── .env.example
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

---

## 🚀 Schritt-für-Schritt Setup

### 1. Supabase Projekt erstellen

1. Gehe zu [supabase.com](https://supabase.com) → "New Project"
2. Name: `wochenplan-wachsbleiche`
3. Datenbank-Passwort notieren
4. Region: `eu-central-1` (Frankfurt) empfohlen
5. Warte bis Projekt bereit ist (~2 min)

### 2. SQL ausführen

Im Supabase Dashboard → **SQL Editor** → "New query":

1. Inhalt von `supabase/migrations/001_initial.sql` einfügen → **Run**
2. Prüfen: Links unter "Table Editor" sollten die 4 Tabellen sichtbar sein

### 3. Supabase Auth Einstellungen

Im Dashboard → **Authentication** → **Providers**:
- Email: ✅ aktiviert
- "Confirm email" → je nach Bedarf (intern: kannst du **deaktivieren** für einfacheren Setup)

→ **Authentication** → **URL Configuration**:
- Site URL: `https://dein-project.vercel.app`
- Redirect URLs: `https://dein-project.vercel.app/**`

### 4. Vercel verbinden

```bash
# Lokal: Repo pushen
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:DEIN_USERNAME/wochenplan.git
git push -u origin main
```

1. [vercel.com](https://vercel.com) → "New Project" → GitHub Repo importieren
2. Framework: **Next.js** (auto-detected)
3. Root Directory: `/` (kein Unterordner)

### 5. Env Vars setzen

In Vercel → Project Settings → **Environment Variables**:

| Variable | Wo zu finden |
|----------|-------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase → Settings → API → Project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase → Settings → API → anon/public key |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase → Settings → API → service_role key ⚠️ geheim! |

Dann: **Deploy** klicken.

### 6. Erste Anmeldung + Admin setzen

#### 6a. Account erstellen
1. Gehe zu deiner Vercel-URL → `/login`
2. Erstelle deinen Account direkt in Supabase: **Authentication** → **Users** → "Add User"
   - Email + sicheres Passwort
   - ✅ "Auto Confirm User"

#### 6b. Admin-Rolle setzen
In Supabase → SQL Editor:

```sql
-- 002_seed_admin.sql anpassen:
update public.profiles
set role = 'admin'
where id = (
  select id from auth.users where email = 'DEINE@EMAIL.DE'
);
```

→ **Run**

#### 6c. Einloggen
Gehe zu `/login` und melde dich an. Du wirst automatisch zum Team-Dashboard weitergeleitet.

---

## 👥 Weitere Benutzer anlegen

Als Admin: `/users` → "Neuer Benutzer" → Email, Name, Passwort, Rolle eingeben.

---

## 📱 PWA / Mobile

Die App ist als PWA vorbereitet. Auf dem iPhone:
- Safari → Teilen → "Zum Home-Bildschirm"

→ App öffnet sich ohne Browser-UI.

---

## ✅ Checkliste: Was getestet werden muss

### Auth & Rollen
- [ ] Login funktioniert (Email/Passwort)
- [ ] Berater sieht nur `/me`, kein `/team` oder `/users`
- [ ] Admin sieht `/team`, `/users`
- [ ] Redirect nach Login korrekt (Admin → /team, Berater → /me)
- [ ] Logout funktioniert

### Dateneingabe
- [ ] `/me`: KW-Selektor zeigt aktuelle KW als Default
- [ ] Plan + Metrics werden automatisch angelegt (kein manuelles Erstellen nötig)
- [ ] Alle Felder speicherbar (Analysen, Erstberatung, Nachberatung, Service, Umsatz, Verdienst)
- [ ] Negative Zahlen werden abgelehnt
- [ ] Karriere-Note Dropdown funktioniert
- [ ] Speichern-Button: wird grau wenn kein dirty state
- [ ] Toast-Feedback bei Speichern

### Teamübersicht
- [ ] Admin: alle Berater in Tabelle sichtbar
- [ ] Sortierung nach Umsatz, Verdienst, Analysen
- [ ] Team-Summe (letzte Zeile) korrekt
- [ ] KW-Wechsel aktualisiert Daten
- [ ] Berater ohne Daten zeigen 0er-Werte (nicht fehlen)

### Benutzerverwaltung
- [ ] Neuen Berater erstellen → kann sich einloggen
- [ ] Rolle Admin ↔ Berater änderbar
- [ ] Level-Felder (aktuell/Ziel) editierbar

### RLS (Security)
- [ ] Berater kann andere Berater-Daten NICHT sehen (direkte Supabase-Abfrage testen)
- [ ] Admin kann alle Daten sehen

### Mobile
- [ ] Hamburger-Menü öffnet auf Mobile
- [ ] Eingabemaske auf iPhone bedienbar

---

## 🔧 Lokale Entwicklung

```bash
# Abhängigkeiten installieren
npm install

# .env.local erstellen
cp .env.example .env.local
# → Werte aus Supabase eintragen

# Dev-Server starten
npm run dev

# Type-Check
npm run type-check

# Lint
npm run lint
```

---

## 📝 Erweiterungsideen

- **Monats-Aggregation**: Daten über mehrere KWs summieren
- **Export**: CSV/Excel-Export der Team-Daten
- **Ziele**: Pro Berater Wochenziele definieren + Ampellogik
- **Notizen**: Team-Notizen pro KW (Admin only, `team_notes`-Tabelle bereits vorhanden)
- **Charts**: Umsatz-Trend über Wochen visualisieren
