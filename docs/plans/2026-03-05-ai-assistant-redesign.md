# AI Asistent Redesign — Dilibees 2.0 Dokumentace

**Datum:** 2026-03-05
**Status:** Schváleno, implementace

## Problém

AI asistent v dokumentaci dává strohé odpovědi a nerozumí systému Dilibees.

**Příčiny:**
1. Systémový prompt má jen 4 řádky — žádné doménové znalosti
2. Kontext = pouze `innerText` aktivního tabu (max 15k znaků)
3. Žádná historie konverzace
4. Žádný streaming — uživatel čeká na celou odpověď

## Schválený design (4 sekce)

### Sekce 1: Rich System Prompt (~800 slov)

Komplexní systémový prompt s plnou znalostí Dilibees:
- Co je Dilibees (agenturní HR systém, dělnické profese, SaaS B2B)
- Uživatelské role (Admin agentury, Klient, Pracovník, Dispečer)
- Tech stack (Supabase, PostgreSQL, RLS, Edge Functions, Flutter, Next.js)
- Design systém (barvy, typografie, komponenty)
- Klíčové pojmy (směna, zápůjčka, docházka, přidělení)
- Instrukce pro kvalitní odpovědi (česky, odrážky, cross-reference)

### Sekce 2: Full Documentation Extraction

- Při načtení stránky extrahuje VŠECHNY sekce (ne jen aktivní tab)
- Strukturovaný formát: `[MODE > Tab] obsah...`
- Uloženo v proměnné `AI_DOC_CACHE`
- Odesláno jako `cache_control: {type: "ephemeral"}` pro prompt caching
- 90% úspora po prvním volání (5min TTL)

### Sekce 3: Konverzace + Streaming

- 10 zpráv v paměti (`AI_CHAT_HISTORY[]`)
- SSE streaming (`stream: true`) — text se zobrazuje v reálném čase
- `AbortController` + tlačítko "Stop" pro přerušení generování
- Parser SSE eventů (`content_block_delta`, `message_stop`, `message_delta`)

### Sekce 4: UI vylepšení

- Chat historie s bublinami (uživatel vpravo, AI vlevo)
- Tlačítko "Vymazat chat" v headeru
- "Stop" tlačítko nahradí "Zeptat se" během generování
- Indikátor nákladů (cached vs uncached cena, kumulativní)
- Auto-scroll dolů při streamingu

## Soubory k úpravě

| Soubor | Sekce | Řádky |
|--------|-------|-------|
| `supabase/index.html` | CSS | 114-143 |
| `supabase/index.html` | HTML | 257-271 |
| `supabase/index.html` | JavaScript | 7887-8029 |

## Cenový model

- **Prompt caching**: System prompt + docs kontext = cached po 1. volání
- **Vstup (cached):** $0.375/MTok (vs $3/MTok uncached) = 90% úspora
- **Výstup:** $15/MTok
- Typický dotaz: ~$0.003 cached vs ~$0.02 uncached
