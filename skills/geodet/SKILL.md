---
name: geodet
description: >-
  Skill vynikajícího geodeta s hlubokými znalostmi geodetických měření,
  kartografie a geospatiaálních technologií. Odborník na práci s GPS, GNSS,
  měřickou technikou, mapováním a přesností měření. Chápe specifika stavební
  geodézie, zaměřování pozemků, výšková měření a integraci geodetických dat
  do stavebních projektů. Vhodný pro vývoj funkcí týkajících se měření, mapování
  a geodetických služeb v aplikaci.

---

# Geodetické Know-how

## Kdy Aktivovat Skill

Aktivuj tento skill když potřebuješ:

- Výpočty vzdáleností, ploch a objemů
- Převody souřadnic (WGS84, S-JTSK, ETRS89)
- Výšková měření a převýšení
- Analýzu přesnosti měření
- Tvorbu map a grafů měření
- Zpracování GPS/GNSS dat
- Stavební geodetické výpočty
- Kontrolu měřických dat

## Geodetické Výpočty

### Vzdálenosti a Azimuty

Haversine vzdálenost mezi dvěma body v WGS84:

<code-snippet name="Haversine Distance" lang="php">
function haversineDistance(float $lat1, float $lon1, float $lat2, float $lon2): float
{
    $R = 6371000; // Poloměr Země v metrech
    $dLat = deg2rad($lat2 - $lat1);
    $dLon = deg2rad($lon2 - $lon1);
    
    $a = sin($dLat / 2) ** 2 +
         cos(deg2rad($lat1)) * cos(deg2rad($lat2)) * sin($dLon / 2) ** 2;
    $c = 2 * atan2(sqrt($a), sqrt(1 - $a));
    
    return $R * $c;
}
</code-snippet>

### Převody Souřadnic

#### S-JTSK vs JTSK03

Česká státní souřadnicová síť má **dva systémy** s různými počátky:

**S-JTSK (Jednotná Trigonometrická Síť Katastrální)**
- Původní systém (používaný od 1920. let)
- **Počátek**: Ferrův bod v Itálii (imaginární bod mimo Česko)
- **Souřadnice**: Kladné čísla ve velkém rozsahu
  - X (sever): 900 000 - 1 100 000 m
  - Y (východ): 400 000 - 700 000 m
- **Příklad**: X=654 321, Y=456 789
- Používaný v katastrech a starší dokumentaci

**JTSK03 (Digitální Jednotná Trigonometrická Síť)**
- Novější, modernizovaný systém (od 2010)
- **Počátek**: Blízko středu Česka (praktičtější)
- **Souřadnice**: Záporná a kladná čísla (menší rozsahy)
  - X (sever): -200 000 až +200 000 m
  - Y (východ): -150 000 až +150 000 m
- **Příklad**: X=−45 678, Y=−12 345
- Moderní systém pro GPS/GNSS aplikace

#### Rozdíly

| Vlastnost | S-JTSK | JTSK03 |
|-----------|--------|-------|
| Počátek | Ferrův bod (Itálie) | Střed Česka |
| Souřadnice | Kladné (velký rozsah) | Záporné/Kladné (menší rozsah) |
| Přesnost | ±0.5 m | ±0.01 m (vyšší) |
| Rotace | 0° | +1.5° (oproti S-JTSK) |
| Měřítko | 0.9999 | 1.0 (jednotkové) |

#### Převod S-JTSK ↔ JTSK03

<code-snippet name="JTSK Conversion" lang="php">
function sJtskToJtsk03(float $x, float $y): array
{
    // Transformační parametry
    $dx = -485_063.74;  // Posun X
    $dy = -301_767.19;  // Posun Y
    $alpha = deg2rad(1.5);  // Rotace v radiánech
    $scale = 0.9999;     // Měřítko
    
    // Normalizace (odstranění posunu původního S-JTSK)
    $x_norm = $x - 500_000;
    $y_norm = $y - 500_000;
    
    // Aplikace rotace a měřítka
    $x_rot = $scale * ($x_norm * cos($alpha) + $y_norm * sin($alpha));
    $y_rot = $scale * (-$x_norm * sin($alpha) + $y_norm * cos($alpha));
    
    // Aplikace posunu na JTSK03
    $x_03 = $x_rot + $dx;
    $y_03 = $y_rot + $dy;
    
    return ['x' => $x_03, 'y' => $y_03];
}

function jtsk03ToSJtsk(float $x, float $y): array
{
    // Inverzní transformace
    $dx = -485_063.74;
    $dy = -301_767.19;
    $alpha = deg2rad(1.5);
    $scale = 0.9999;
    
    // Odebrání posunu JTSK03
    $x_norm = $x - $dx;
    $y_norm = $y - $dy;
    
    // Inverzní rotace a měřítko
    $inv_scale = 1 / $scale;
    $x_rot = $inv_scale * ($x_norm * cos(-$alpha) + $y_norm * sin(-$alpha));
    $y_rot = $inv_scale * (-$x_norm * sin(-$alpha) + $y_norm * cos(-$alpha));
    
    // Návrat na S-JTSK se součtem posunu
    $x_jtsk = $x_rot + 500_000;
    $y_jtsk = $y_rot + 500_000;
    
    return ['x' => $x_jtsk, 'y' => $y_jtsk];
}
</code-snippet>

#### Praktické Poznámky

- **Katastry**: Stále používají S-JTSK (kladná čísla)
- **Moderní GPS/GNSS**: Převádějí na JTSK03 (včetně záporných souřadnic)
- **Rozdíl**: ~485 km na X a ~302 km na Y
- **Přesnost převodu**: Typicky ±0.1 m

### Výšková Měření

- Relativní výšky (převýšení)
- Absolutní výšky (nadmořská výška)
- Kontrola nivelace a sklonitosti

## Stavební Geodézie

- Zaměřování pozemků a stavbišť
- Usazování staveb
- Kontrola svislosti a vodorovnosti
- Monitoring posunů konstrukcí
- Tachymetrické měření

## Přesnost a Kvalita Měření

- Středních chyb a odchylek
- Tolerancí a mezních odchylek
- Statistické analýzy měrených hodnot
- Identifikace hrubých chyb
