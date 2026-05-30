# M-BUCKS Economy V2

Diese Erweiterung modernisiert das M-BUCKS-System in `index.html` und ergänzt ein optionales SQL-Backend für Supabase/Postgres.

## Was wurde im Frontend erneuert

- Neues Economy-Dashboard im M-BUCKS-Menü (`#mbucksCreditMenu`):
  - Wallet-Anzeige
  - Tagesverdienst
  - Auto-M-BUCKS Toggle
  - Ledger (letzte Buchungen)
- Zentrale Funktionen statt verteilter Direktzugriffe:
  - `grantMBucks(amount, source, options)`
  - `spendMBucks(amount, source, options)`
- Auto-Vergabe:
  - Intervall: alle `45s`
  - Reward: `+25 M-BUCKS`
  - Steuerbar per Toggle
- Persistenz im Savegame:
  - `mbucksEconomyState` wird mitgespeichert und geladen.

## Wichtige neue States/Funktionen

- State:
  - `mbucksEconomyState.autoEnabled`
  - `mbucksEconomyState.autoLastGrantAt`
  - `mbucksEconomyState.earnedToday`
  - `mbucksEconomyState.ledger`
- Core:
  - `normalizeMBucksEconomyState(...)`
  - `grantMBucks(...)`
  - `spendMBucks(...)`
  - `processMBucksAutoIncome()`
  - `renderMBucksEconomyPanel()`

## Bereits umgestellte Reward-Punkte

Beispielhaft sind zentrale Gameplay/Reward-Pfade bereits auf das neue System umgestellt:
- Vault-Belohnung
- Chain-Reaction-Belohnung
- Alien-Drops
- Hero-Gauntlet Rewards
- Admin-Bonus-Buttons

## SQL-Backend (optional, empfohlen)

Datei: `mbucks_economy_v2.sql`

Sie erstellt:
- `public.player_wallets`
- `public.mbucks_ledger`
- RPC-Funktionen:
  - `public.grant_mbucks(...)`
  - `public.spend_mbucks(...)`
- View:
  - `public.v_mbucks_wallet_summary`

### Einspielen

1. SQL Editor in Supabase öffnen.
2. Inhalt aus `mbucks_economy_v2.sql` ausführen.
3. RLS/Policies passend zu deinem Auth-Modell ergänzen (falls nötig).

## Beispiel: RPC aus dem Frontend

```js
await supabaseClient.rpc("grant_mbucks", {
  p_player_key: cloudPlayerKey,
  p_amount: 25,
  p_source: "Auto-Lobby",
  p_metadata: { reason: "interval" }
});
```

```js
await supabaseClient.rpc("spend_mbucks", {
  p_player_key: cloudPlayerKey,
  p_amount: 120,
  p_source: "Shop-Buy",
  p_metadata: { itemId: "bg_neon" }
});
```

## Integration-Strategie (empfohlen)

1. Kurzfristig: Weiter lokal mit `grantMBucks/spendMBucks` arbeiten.
2. Mittelfristig: Cloud-Accounts zusätzlich über RPC synchronisieren.
3. Langfristig: Alle `inventory[7] +=/-=` Stellen auf zentrale Calls migrieren.

## Hinweis

Die Economy V2 ist rückwärtskompatibel zum bestehenden Saveflow; direkte alte `inventory[7]`-Zugriffe funktionieren weiterhin, aber nur zentrale Calls schreiben sauber ins neue Ledger.
