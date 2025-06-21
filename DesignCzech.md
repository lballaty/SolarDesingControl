# Systémový Návrh: Expanse Solaru Velenice

---

### ✅ Přehled Systému – dvě varianty

#### Varianta A – 10 kWp / 140 kWh (Plný výkon)

| Položka                   | Parametr                                |
|-----------------------------|------------------------------------------|
| **Off-grid FV pole**       | 10 kWp, bifaciální panely (skleník)       |
| **Baterie**                | 140 kWh FLA (Flooded Lead Acid), 48 V     |
| **MPPT regulátory**        | Victron SmartSolar MPPT 150/100 (2x)     |
| **Měnič/střídač**          | Victron MultiPlus-II 48/5000/70           |
| **Monitoring**             | Victron Cerbo GX + SmartShunt             |
| **Zálohované výstupy**     | Rybník filtrace, osvětlení, lednička, Wi-Fi, topná spirála v AKU nádrži |
| **On-grid systém**         | GoodWe 10 kWp (nelze upravit)            |
| **Nabíjení EV**            | Pouze v létě (manuálně)                  |

#### Varianta B – 5 kWp / 70 kWh (Zmenšený systém)

| Položka                   | Parametr                                |
|-----------------------------|------------------------------------------|
| **Off-grid FV pole**       | 5 kWp, bifaciální panely (skleník)        |
| **Baterie**                | 70 kWh FLA (např. 48 V / 1450 Ah)          |
| **MPPT regulátory**        | Victron SmartSolar MPPT 150/70 (1x)       |
| **Měnič/střídač**          | Victron MultiPlus-II 48/3000/35           |
| **Monitoring**             | Cerbo GX nebo Color Control GX            |
| **Zálohované výstupy**     | Rybník, LED osvětlení, základní DC zátěže  |
| **Rozšiřitelnost**         | Možnost pozdějšího navýšení               |

---

### ♻️ Sezónní Provoz (A i B)

#### ❄️ Zima (listopad - únor)
- Varianta A: produkce 3–6 kWh/den, prioritizace všech kritických zátěží
- Varianta B: produkce 2–4 kWh/den, prioritizace pouze rybníku a základního světla

#### ☀️ Léto (březen - říjen)
- Varianta A: přebytky až 50–60 kWh/den, možné EV nabíjení, topná spirála v AKU nádrži
- Varianta B: pokrytí základních zátěží + občasné přepínání zátěží z gridu

---

### ⚡ Řízení systému (Cerbo GX)

| Pravidlo | Podmínka                           | Akce (A/B)                         |
|----------|------------------------------------|-----------------------------------|
| R1       | SoC < 35 %, čas 22:00–06:00        | Vypni osvětlení (A i B)           |
| R2       | SoC < 30 %                         | Vypni vše kromě rybníku (A i B)   |
| R3       | SoC > 60 %, FV > 500 W             | Zapni lednici, router (jen A)     |
| R4       | SoC > 80 %, čas 10:00–16:00, léto  | Povolit EV nabíjení, topnou spirálu (jen A) |
| R5       | SoC > 95 %, žádné aktivní zátěže   | Odpojit FV vstup (relé nebo Remote OFF MPPT) |

---

...(předchozí obsah beze změny)...

---

### 🖼 Podrobné zapojení – Varianta A (aktualizováno s duálním nabíjením)

```mermaid
flowchart TD
    subgraph Panels_A [Ostrovní FV pole 10 kWp]
        PA1["String 1 (5 kWp)"] --> MPPT_A1["MPPT 150/100"]
        PA2["String 2 (5 kWp)"] --> MPPT_A2["MPPT 150/100"]
        DIS_A["Odpojovač panelů"]
        MPPT_A1 --> DIS_A --> BATA["Baterie 140 kWh / 48 V FLA"]
        MPPT_A2 --> DIS_A
    end

    subgraph Grid_A [Systém připojený k síti – GoodWe 10 kWp]
        GW_INV_A["Střídač GoodWe"] --> AC_MAINA["Hlavní AC rozvaděč"]
        AC_MAINA --> GW_BATT_A["Baterie GoodWe 12 kWh"]
        GW_BATT_A -->|Je-li plná| RELAYA["Chytré relé"] --> AC_CHG_A["AC nabíječka do FLA baterie"] --> BATA
    end

    BATA --> MPA["Victron MultiPlus-II 48/5000"]
    MPA --> AC_OUT_A["Rozvod kritických zátěží"]
    AC_OUT_A --> L1A["Filtrace ryb"]
    AC_OUT_A --> L2A["LED osvětlení"]
    AC_OUT_A --> L3A["Lednice, router"]
    AC_OUT_A --> HEATA["Topná spirála (bojler)"]
    AC_OUT_A --> AUTO1["Auto Nabijeni 2 (zasuvka 3)"]
    AC_MAINA --> AUTO2["Auto Nabijeni 1 (zasuvka 1)"]
    AC_MAINA --> L4A["Zásuvky, sporák, tepelné čerpadlo"]
```

> **Poznámka:** Stejná logika programovatelného relé platí i zde. Jakmile je 12kWh baterie GoodWe plně nabita, AC výstup se přepne do ostrovního systému pomocí vhodné AC nabíječky.

---
### 🖼 Podrobné zapojení – Varianta B (aktualizováno s duálním nabíjením)

```mermaid
flowchart TD
    subgraph Panels_B [Ostrovní FV pole 5 kWp]
        PB1["String (5 kWp)"] --> MPPT_B["MPPT 150/70"]
        DIS_B["Odpojovač panelů"]
        MPPT_B --> DIS_B --> BATB["Baterie 70 kWh / 48 V FLA"]
    end

    subgraph Grid_B [Síťové FV pole GoodWe 10 kWp]
        GW_INV["Střídač GoodWe"] --> AC_MAINB["AC rozvaděč domu"]
        AC_MAINB --> GW_BATT["Baterie 12 kWh GoodWe"]
        GW_BATT -->|Je-li plná| TRANSFER["Chytré relé / přepínač"] --> CHG_CTRL["AC nabíječka do FLA baterie"] --> BATB
    end

    BATB --> MPB["Victron MultiPlus-II 48/3000"]
    MPB --> AC_B_OUT["Rozvod kritických zátěží"]
    AC_B_OUT --> L1B["Filtrace ryb"]
    AC_B_OUT --> L2B["LED osvětlení"]
    AC_B_OUT --> L3B["DC spotřebiče"]

    AC_MAINB --> L4B["Zásuvky, sporák, tepelné čerpadlo"]
```

> **Poznámka:** Programovatelné přepínací relé řízené SoC sledováním může přesměrovat přebytečný výkon ze sítě (GoodWe) do ostrovní baterie pomocí Victron Phoenix Charger nebo jiné vhodné AC nabíječky.

> **Poznámka:** Programovatelné relé řízené Cerbo GX spíná AC nabíječku:
pokud je GoodWe baterie plná nebo
pokud je SoC FLA baterie pod 50 %
Tím je zajištěna minimální dostupnost energie pro kritické zátěže i při slabém slunečním svitu.

---

### 🛠️ Montážní plán a instalační poznámky

#### Montáž panelů (skleník / polykarbonátová střecha)

* Použít hliníkové C-profily na ocelové nosníky pod polykarbonátem
* Zajistit větrací mezeru min. 10 cm pod panely
* Použít manuálně naklápěcí úchyty: \~25° léto / \~45° zima (nastavení 2× ročně)

#### Umístění baterií

* Dobře větrané, chladné místo (ideálně sklep nebo technická místnost)
* Na nevodivou podložku s ochranou proti elektrolytu
* Vzdálenost k měniči max. 2 metry kvůli ztrátám

#### Kabeláž a jištění

* DC stringy chráněny jističem 25 A / 100 VDC
* AC výstup z měniče chráněn 25 A dvoupólovým jističem
* Signální kabely (VE.Bus, SmartShunt) vést odděleně od silových

#### Cerbo GX + chytré relé

* Cerbo GX sleduje SoC a řídí logiku přepínání
* Chytré relé (např. Victron Digital Input Relay nebo Shelly Pro) sepne AC nabíječku, jakmile je 12kWh baterie plná
* Doporučená nabíječka: Victron Phoenix Charger 24/48 V


