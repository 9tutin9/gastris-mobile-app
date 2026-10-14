# GastrIS Číšník — Android app

Nativní Android wrapper (Capacitor 6) pro číšnickou PWA GastrISu. Vypadá a chová se
jako plnohodnotná appka (ikona na ploše, fullscreen, žádná adresní řádka), ne jako
stránka v prohlížeči. Párovací QR z kasy se skenuje přes Google **ML Kit** (stejná
čtečka jako Albert/DM/Tesco appky).

## Jak to funguje

```
┌─────────────────────────────────────────┐
│ GastrIS APK (Capacitor, cz.gastris.app)  │
│ ┌───────────────────────────────────────┐│
│ │ www/index.html (párovací launcher)    ││
│ │   📷 sken QR z kasy (ML Kit)          ││
│ │   → GastrisServer.setServer(url,claim)││
│ │ WebView → https://UZEL/?wt=…&mode=…   ││
│ │   ↑ existující GastrIS číšnická PWA   ││
│ └───────────────────────────────────────┘│
│ Plugins: @capacitor-mlkit/barcode-scanning│
└─────────────────────────────────────────┘
```

**Párování (jednou):** na kase otevři *Nastavení → Zařízení → Vytvořit QR*, vyber
číšníka a režim (číšník / kuchyně). Na telefonu v appce dej *Naskenovat QR z kasy*.
Appka si uloží adresu uzlu a přihlásí zařízení (jednorázový `wt` token z QR).

**Běžný provoz:** appka startuje rovnou do GastrISu přihlášená. QR token je
jednorázový — po spárování appka při restartech načítá čistý origin uzlu a
přihlašuje se z uloženého JWT (plugin drží `server_url` zvlášť od jednorázové
`claim_url`, takže restart nezkouší spotřebovaný token).

**Odpárování:** v appce (číšnická PWA) odhlášení / na kase zruš zařízení.

## Build & distribuce (GitHub Actions)

Push do `main` spustí `.github/workflows/build.yml`:
1. Node 20 + Java 17 + Android SDK
2. `npx cap add android` + `cap sync` (vygeneruje Android projekt)
3. Injektne nativní `GastrisServer` plugin + vlastní `MainActivity` (balíček `cz.gastris.app`)
4. Doplní CAMERA permission, `./gradlew assembleDebug`
5. Re-sign konzistentním klíčem (uber-apk-signer) → update se nainstaluje přes předchozí
6. Nahraje `gastris-cisnik.apk` jako asset release `android` v TOMTO (veřejném) repu
   přes vestavěný `GITHUB_TOKEN` → přesně tam míří QR „Stáhnout appku" v Nastavení
   GastrISu. **Žádný extra secret/PAT** — build je plně samostatný.

Odkaz na APK (stabilní, přepisuje se při každém buildu):
`https://github.com/9tutin9/gastris-mobile-app/releases/download/android/gastris-cisnik.apk`

## Update

- **Změna web kódu GastrISu** (JS/HTML) → NErebuilduj APK. Server zaktualizuje obsah
  WebView sám (PWA cache invalidovaná verzí); telefon stáhne při dalším otevření.
- **Změna Capacitor configu / pluginu** → push → nový APK z Actions → re-sideload.

## Oprávnění
- **CAMERA** (sken párovacího QR), **INTERNET** (komunikace s uzlem).

První sken stáhne ML Kit modul z Play Services (~3 MB, jednou).
