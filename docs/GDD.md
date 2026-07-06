# Dokument Designu Gry (GDD)

## „Sortownia" — VR arcade z obiektami Gaussian Splatting

> Wersja: 0.2 (draft) · Data: 2026-06-29 · Silnik: Unity · Platforma: VR (HTC Vive Focus 3, standalone)

---

## 1. Wizja i pitch

**Pitch w jednym zdaniu:** Stoisz przy stanowisku sortowniczym w VR — segregujesz przyjeżdżające na taśmie obiekty (zeskanowane techniką Gaussian Splatting) do trzech koszyków wg kategorii, jednocześnie odpierając latające stworki, które chcą zniszczyć Twoje stanowisko.

**Gatunek:** VR arcade / action-sorting / reflex.

**Filar designu — co czyni grę ciekawą:** napięcie wynika z **dzielenia uwagi między dwa kanały** (spokojne, ale wymagające precyzji sortowanie na taśmie vs. nerwowe odpieranie stworków) oraz z **hamowania impulsu** (nie wszystko wolno łapać — część obiektów to pułapki, które trzeba świadomie przepuścić).

**Grupa docelowa:** gracze VR lubiący szybkie sesje arcade (Job Simulator, Pistol Whip, Fruit Ninja VR). Sesja pojedynczej rundy: ~90–120 s.

**Kontekst projektu:** demo technologiczne pokazujące użycie assetów Gaussian Splatting jako obiektów rozgrywki w Unity.

---

## 2. Pętla rozgrywki (core loop)

```
        ┌─────────────────────────────────────────────┐
        │                                             │
        ▼                                             │
  Obiekt pojawia się ──► Gracz ROZPOZNAJE typ ──► Decyzja:        │
  (taśma lub stwór)      (sortowalny/stwór/pułapka)               │
                                  │                               │
            ┌─────────────────────┼─────────────────────┐        │
            ▼                     ▼                     ▼        │
      SORTUJ (złap        ZNISZCZ (zestrzel       ZIGNORUJ       │
      i wrzuć do          stworka)                (przepuść      │
      koszyka)                                    pułapkę)       │
            │                     │                     │        │
            └─────────────────────┼─────────────────────┘        │
                                  ▼                               │
                       Punkty / życie / combo ──────────────────┘
                                  │
                                  ▼
                    Koniec czasu LUB życie = 0 → wynik
```

**Mikro-pętla (sekundowa):** zauważ → rozpoznaj → zdecyduj → wykonaj → feedback.
**Makro-pętla (rundowa):** przetrwaj czas, maksymalizuj wynik, pobij rekord.

---

## 3. Mechaniki szczegółowe

### 3.1 Kanał A — Taśma i sortowanie semantyczne

- Obiekty jadą na **taśmociągu** w stronę gracza ze stałą (rosnącą z czasem) prędkością.
- Gracz ma **okno czasowe** na rozpoznanie i reakcję, zanim obiekt dojedzie do końca taśmy.
- **3 koszyki = 3 kategorie semantyczne.** Motyw bazowy (do wyboru / łatwy do podmiany):
  - **Recykling:** Plastik / Papier / Szkło (spójne tematycznie, edukacyjne, czytelne).
  - Alternatywa: Jedzenie / Narzędzia / Elektrośmieci.
- Gracz **łapie obiekt ręką** (chwyt kontrolerem) i **wrzuca do koszyka**.
- Każdy koszyk jest wyraźnie oznaczony ikoną + kolorem kategorii.

**Wynik akcji sortowania:**
| Sytuacja | Efekt |
|---|---|
| Trafienie do właściwego koszyka | + punkty, podbicie combo |
| Trafienie do złego koszyka | − punkty, reset combo |
| Obiekt przejechał niezłapany (był sortowalny) | mała kara „przegapienia", reset combo |

### 3.2 Kanał B — Stworki i strzelanie

- Stworki (osobne assety) **latają w przestrzeni** i kierują się **w stronę stanowiska gracza**.
- Gracz **strzela** do nich z kontrolera (raycast + efekt trafienia).
- Zniszczony stwór = + punkty.
- Jeśli stwór **doleci do stanowiska**:
  - zabiera punkt życia,
  - dodatkowo „zakłóca" rozgrywkę (np. na chwilę zwalnia/zatrzymuje taśmę albo częściowo zasłania widok) — kara mechaniczna, nie tylko liczbowa.
- Stworki zmuszają do **priorytetyzacji**: czasem trzeba porzucić taśmę, odwrócić się i ostrzelać rój.

### 3.3 Obiekty-pułapki (do pominięcia)

- Wyglądają jak normalne obiekty sortowalne, ale mają **czytelny wyróżnik** (pulsowanie, czerwona/skażona poświata, ikona ostrzeżenia).
- **Wrzucenie pułapki do dowolnego koszyka = duża kara punktowa + ubytek życia.**
- Poprawna reakcja: **przepuścić** ją do końca taśmy (tam znika bez kary).
- Cel projektowy: wymusić powściągliwość — gracz nie może łapać wszystkiego odruchowo.

### 3.4 Combo i nagradzanie skupienia

- Seria poprawnych akcji (dobre sortowania + zniszczone stworki) bez błędu buduje **mnożnik combo**.
- Po osiągnięciu progu (np. ×5) — mnożnik punktów i drobny **regen życia**.
- Dowolny błąd (zły koszyk, pułapka, przegapienie, trafiony przez stwora) **resetuje combo**.

---

## 4. Ekonomia gry — punkty, życie, czas

### 4.1 Warunek końca rundy

- **Czas + pasek życia.** Cel: przetrwać rundę i zmaksymalizować wynik.
- Runda trwa **90–120 s** (parametr trudności).
- Koniec rundy następuje gdy: **upłynie czas** ALBO **życie spadnie do 0** (przedwczesny game over).

### 4.2 Tabela wartości (startowy balans — do tuningu)

| Akcja | Punkty | Życie |
|---|---:|---:|
| Dobry koszyk | +10 | — |
| Zły koszyk | −5 | — |
| Przegapiony obiekt sortowalny | −3 | — |
| Wrzucona pułapka | −20 | −1 |
| Zestrzelony stwór | +15 | — |
| Stwór dotarł do stanowiska | −10 | −1 |
| Combo ×5 (próg) | mnożnik ×2 | +1 (regen) |

- **Pasek życia startowy:** 5 punktów (do tuningu).
- **Życie maksymalne:** 5 (regen nie przekracza maksa).

### 4.3 Feedback dla gracza

- Każda akcja ma **natychmiastowy feedback**: dźwięk, efekt cząsteczkowy, „pop" punktów w world-space.
- Życie i czas widoczne na **panelu przy stanowisku** (world-space canvas), NIE przyklejone do twarzy (HUD na twarzy męczy w VR).
- Stan „mało życia" / „mało czasu" sygnalizowany wizualnie i dźwiękowo (puls, alarm).

---

## 5. Progresja trudności

Trudność rośnie w trakcie rundy (skalowanie po czasie lub po fazach):

| Parametr | Start | Późna faza |
|---|---|---|
| Prędkość taśmy | wolna | szybka |
| Częstość spawnu obiektów | rzadka | gęsta |
| Odsetek pułapek | ~10% | ~30% |
| Liczba stworków naraz | 0–1 | rój (3–5) |
| Prędkość stworków | wolna | szybka |

- **Wczesna faza:** głównie sortowanie, pojedyncze stworki — nauka mechanik.
- **Środkowa faza:** regularne stworki + więcej pułapek.
- **Późna faza (klimaks):** gęsta taśma + rój stworków jednocześnie — szczyt podziału uwagi.

---

## 6. Architektura techniczna (Unity)

### 6.1 Zasada dla assetów Gaussian Splatting

> **Splat = tylko warstwa wizualna.** Logika, collidery i interakcje siedzą na zwykłym GameObjekcie-rodzicu.

- Asset GS jest *dzieckiem* obiektu rozgrywki.
- Collider (prosty box/sphere), `Rigidbody`, komponent interakcji XR oraz logika punktowa są na **rodzicu**.
- **Nie** licz fizyki ani raycastów bezpośrednio na geometrii splata (wolne, niestabilne).

### 6.2 Kluczowe komponenty / klasy

| Komponent | Odpowiedzialność |
|---|---|
| `GameManager` | Czas rundy, punkty, combo, życie, faza trudności; emituje eventy stanu gry. |
| `ConveyorBelt` | Przesuwanie obiektów po torze; spawn z poola; despawn na końcu + zgłoszenie „przegapienia". |
| `ObjectSpawner` | Losowanie typu obiektu wg wag zależnych od fazy trudności; object pooling. |
| `SortableItem` | Dane obiektu: `Category` (enum), `IsTrap` (bool), wyróżnik wizualny; interakcja chwytu. |
| `Basket` | Trigger; porównuje `Category` wrzuconego obiektu; wykrywa pułapkę; woła event punktowy. |
| `Creature` | AI lecące ku stanowisku; `Health`; zdarzenia `OnKilled`, `OnReachPlayer`. |
| `CreatureSpawner` | Spawn stworków wg fazy trudności; pooling. |
| `ShooterController` | Raycast z kontrolera, input na trigger, trafienie warstwy `Creature`, efekt strzału. |
| `ScoreEvents` / channels | Luźne powiązanie logiki z UI i dźwiękiem (ScriptableObject event channels). |
| `VRStationUI` | World-space canvas: wynik, czas, pasek życia, combo. |

### 6.3 Typy danych (szkic)

```csharp
public enum ItemCategory { Plastic, Paper, Glass }   // lub Food / Tools / Ewaste

public enum ObjectType { Sortable, Creature, Trap }
```

### 6.4 Przepływ zdarzeń (event-driven)

- Akcje gameplayowe (`Basket`, `Creature`, `ConveyorBelt`) **nie** odwołują się bezpośrednio do UI/dźwięku.
- Zgłaszają zdarzenia do `GameManager` / kanałów ScriptableObject.
- UI, audio i efekty **subskrybują** te zdarzenia. Ułatwia to testowanie i rozbudowę.

### 6.5 Wydajność (krytyczne — standalone Vive Focus 3)

> Focus 3 to standalone na Snapdragon XR2 (klasa ~Quest 2). Cały rendering GS liczy się na headsecie — **budżet renderowania splatów jest głównym ograniczeniem projektu**. Pakiet GS (`org.nesnausk.gaussian-splatting`) jest GPU-heavy (compute shadery, sortowanie), więc wymaga ostrego limitowania.

- **Twardy budżet liczby splatów na klatkę.** Ustal limit łącznej liczby gaussianów widocznych naraz i nie przekraczaj go (mała liczba obiektów GS jednocześnie + niskie warianty assetów).
- **Decymacja assetów GS** — eksportuj obiekty z możliwie najmniejszą liczbą splatów, która jeszcze wygląda dobrze. Testuj na headsecie, nie w edytorze.
- **Object pooling** dla obiektów taśmy i stworków (brak `Instantiate`/`Destroy` w trakcie rundy).
- **Proste collidery zastępcze** (box/sphere) zamiast geometrii GS; zero raycastów/fizyki na splatach.
- Rozważ **stworki bez GS** (zwykłe lekkie meshe), a GS zarezerwuj dla obiektów na taśmie — ogranicza to liczbę splatów w „roju".
- Profiluj na docelowym sprzęcie wcześnie (OVR Metrics / Unity Profiler przez ADB). Cel: stabilne 72/90 Hz Focusa 3.
- **Foveated rendering** (jeśli dostępny przez OpenXR na Focus 3) jako dodatkowy zapas wydajności.

---

## 7. Sterowanie (VR)

| Akcja | Wejście |
|---|---|
| Chwyt obiektu | Przycisk grab na kontrolerze (ręka przy obiekcie) |
| Wrzut do koszyka | Puszczenie chwytu nad/we wnętrzu koszyka |
| Strzał do stwora | Trigger kontrolera (celowanie raycastem) |
| Pauza / menu | Przycisk menu |

- **Setup inputu:** **XR Interaction Toolkit 3.0.8** + Input System, runtime przez **OpenXR 1.14.3**.
- **Headset:** HTC Vive Focus 3 (standalone, build na Android). Kontrolery Focus 3 mapowane przez profil OpenXR.
- **XR Hands 1.4.3** dostępne w projekcie — hand tracking jako opcjonalny alternatywny sposób chwytania (poza zakresem MVP).

---

## 8. Oprawa audio-wizualna

- **Stanowisko sortownicze:** centralny punkt sceny, 3 wyraźnie oznaczone koszyki w zasięgu rąk.
- **Czytelność kategorii:** kolor + ikona na każdym koszyku i (subtelnie) na obiektach.
- **Wyróżnik pułapki:** jednoznaczny i czytelny w ruchu (pulsacja / poświata / ikona).
- **Audio:** osobne sygnały dla — dobry koszyk, zły koszyk, zniszczony stwór, alarm pułapki, niskie życie, low time.
- **Komfort VR:** statyczna pozycja gracza (brak sztucznej lokomocji → mniej choroby symulatorowej), UI w world-space.

---

## 9. Zakres MVP (pierwsze grywalne demo)

Minimalny zestaw, by gra była grywalna i testowalna:

1. ✅ Jeden taśmociąg + spawner obiektów (pooling).
2. ✅ 3 koszyki + sortowanie wg kategorii + punktacja.
3. ✅ Pułapki z wyróżnikiem + kara.
4. ✅ Podstawowe stworki + strzelanie z kontrolera.
5. ✅ `GameManager`: czas, życie, punkty, combo.
6. ✅ World-space UI (wynik/czas/życie).
7. ✅ Jedna scena, jedna runda, ekran wyniku.

**Poza MVP (później):** progresja faz, regen życia, efekty zakłócania taśmy przez stworki, tabela wyników, wiele motywów kategorii, balans/tuning, dodatkowe typy stworków.

---

## 10. Otwarte pytania / do decyzji

- [x] ~~Headset docelowy~~ → **HTC Vive Focus 3 (standalone)**. Budżet wydajności = priorytet (sekcja 6.5).
- [x] ~~Stack interakcji~~ → **XR Interaction Toolkit 3.0.8 + OpenXR 1.14.3** (potwierdzone w manifeście).
- [ ] Ile splatów na klatkę uniesie Focus 3 przy stabilnym 72/90 Hz? (wymaga testu na sprzęcie — wpływa na liczbę obiektów GS naraz)
- [ ] Czy stworki robimy na GS, czy na lekkich meshach (rekomendacja: meshe, sekcja 6.5)?
- [ ] Ostateczny motyw kategorii: recykling vs jedzenie/narzędzia/elektrośmieci.
- [ ] Czy stworki zakłócają taśmę (mechanicznie), czy tylko zabierają życie?
- [ ] Czy zły koszyk zabiera życie, czy tylko punkty?
- [ ] Dokładny balans wartości z tabeli 4.2 (wymaga playtestów).
- [ ] Liczba i czas trwania rund / tryb endless.

---

## 11. Słowniczek

- **Gaussian Splatting (GS):** technika reprezentacji/renderowania zeskanowanych obiektów 3D jako chmury gaussianów; tutaj źródło wizualne obiektów rozgrywki.
- **Obiekt sortowalny:** obiekt z taśmy należący do jednej z 3 kategorii.
- **Pułapka:** obiekt wyglądający jak sortowalny, ale karzący za złapanie/wrzucenie.
- **Stwór:** latający wróg do zniszczenia strzałem.
- **Combo:** seria poprawnych akcji bez błędu, budująca mnożnik.
```