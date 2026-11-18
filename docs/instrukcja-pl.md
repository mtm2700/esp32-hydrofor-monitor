# Monitor ciśnienia hydroforu

Moduł pokazuje w Home Assistant napięcie z toru pomiarowego i ciśnienie wody w barach. Historia pozwala prześledzić cykle ciśnienia przy poborze wody. Sterowanie pompą i zabezpieczenia hydrauliczne działają niezależnie od monitora.

## Sprzęt i połączenia

| Rysunek | Polski | English |
|---|---|---|
| Schemat połączeń | [SVG](hydrofor-schemat-polaczen-pl.svg) | [SVG](hydrofor-schemat-polaczen-en.svg) |
| Mapa montażowa A4 | [PDF](hydrofor-mapa-montazowa-pl-a4.pdf) | [PDF](hydrofor-mapa-montazowa-en-a4.pdf) |
| Dzielnik napięcia i filtr A0 | [SVG](hydrofor-dzielnik-a0-pl.svg) | [SVG](hydrofor-dzielnik-a0-en.svg) |
| Widok całego modułu | [SVG](hydrofor-widok-modulu-pl.svg) | [SVG](hydrofor-widok-modulu-en.svg) |

Widok całego modułu pokazuje połączenia według funkcji. Kolejność zacisków czujnika jest poglądowa. Położenie pinów Olimexa pokazuje mapa montażowa A4.

Lista podzespołów i ich status są w [liście części](parts.md). Mapa dotyczy Olimex ESP32-POE-ISO **Rev.M z ESP32-WROOM-32E**. Odczytaj wariant z metalowej osłony modułu. Nadruk `ESP32-WROVER` na czerwonej płytce odnosi się do oznaczenia PCB.

| Sygnał | Połączenie w opisanym wariancie |
|---|---|
| +5 V | Olimex EXT1.1 → VCC czujnika |
| +3,3 V | Olimex EXT1.2 → ADS1115 VDD |
| Wspólna masa | Olimex EXT1.3 → GND ADS1115 i GND czujnika |
| SDA | Olimex EXT2.10 / GPIO13 → ADS1115 SDA |
| SCL | Olimex EXT2.7 / GPIO16 → ADS1115 SCL |
| Adres I²C | ADS1115 ADDR → GND, adres `0x48` |
| Sygnał czujnika | OUT → R1 10 kΩ → węzeł A0 ADS1115 → R2 10 kΩ → GND |

Użyte są boczne listwy EXT1/EXT2. Gniazdo UEXT jest wolne. GPIO17 jest zegarem Ethernetu i nie jest przewodem I²C. Kondensatory 100 nF w projekcie są wpięte między 3,3 V i GND przy ADS1115, 5 V i GND przy złączu czujnika oraz A0 i GND równolegle do R2. Rozpoznaj funkcje przewodów według dokumentacji własnego czujnika przed podaniem napięcia. Nie podawaj sygnału 0,5–4,5 V bezpośrednio na A0.

## Uruchomienie i integracja

1. Przy odłączonym zasilaniu sprawdź połączenia. Przed dołączeniem czujnika sprawdź szyny 5 V i 3,3 V oraz adres ADS1115 w skanie I²C.
2. Użyj [przykładu ESPHome](../esphome/hydrofor-monitor.example.yaml) dla tej rewizji płytki. Obok pliku YAML utwórz własny `secrets.yaml` z wpisem `hydrofor_api_key`. Wartość to własny klucz szyfrowania ESPHome Native API, zapisany jako 32 bajty zakodowane w Base64. Zachowaj go lokalnie. Pierwsze wgranie można wykonać przez USB, a po odłączeniu USB podłączyć PoE.
3. Dodaj urządzenie do Home Assistant przez integrację ESPHome. Sprawdź osobno napięcie A0 i ciśnienie. Próbkowanie ustawiono co sekundę. Po pięciu sekundach bez próbki filtr ciśnienia publikuje stan nieprawidłowy zamiast 0 bar.
4. Przed montażem czujnika w przewodzie wodnym potwierdź dopuszczalne medium, zakres temperatur i parametry ciśnieniowe czujnika, króćca i uszczelnień. W przypadku wody pitnej potrzebne jest potwierdzenie przeznaczenia elementów mających z nią kontakt.

## Przeliczenie i sprawdzenie wskazań

Czujnik ma nominalne wyjście 0,5–4,5 V dla 0–150 psi (około 10,342 bar). Przy dwóch równych rezystorach A0 ma połowę napięcia wyjściowego. Po odjęciu dolnego punktu i podzieleniu przez rozpiętość 2 V otrzymujemy ułamek zakresu czujnika:

```text
P [bar] = (A0 [V] − 0,25) / 2,00 × 10,34213594
        ≈ 5,17106797 × A0 [V] − 1,29276699
```

ESPHome stosuje dwa nominalne punkty, 0,25 V → 0 bar i 2,25 V → 10,34213594 bar. Przy sprawdzaniu własnego egzemplarza odczytaj manometr na tym samym odcinku instalacji i zanotuj w tym samym czasie A0 oraz wskazanie HA. Zapisz rozbieżność z uwzględnieniem czytelności manometru. W opisanym wdrożeniu sprawdzono jeden punkt po zmianie wzoru, około 3,00 bar w HA wobec 3,00–3,05 bar na manometrze. Dokładność w pozostałej części zakresu pozostaje nieustalona.

## Rozwiązywanie problemów

| Objaw | Co sprawdzić |
|---|---|
| Brak ADS1115 w skanie | Zasilanie 3,3 V, wspólną masę, ADDR do GND oraz kolejność SDA/SCL na obu płytkach. |
| Napięcie A0 bliskie zera przy ciśnieniu | Zasilanie czujnika, ciągłość OUT i węzeł R1/R2. Przy przerwanym przewodzie R2 może ściągnąć A0 do masy. Przy nominalnych 0 bar spodziewane A0 to około 0,25 V. |
| Odczyt odbiega od manometru | Zgodność wersji czujnika, wartość i układ rezystorów, napięcie 5 V oraz to, czy odczyty pochodzą z tej samej chwili i z tego samego odcinka. |
| Brak nowych odczytów | PoE, połączenie Ethernet, integrację ESPHome i świeżość raportów. Brak danych oznacza brak wiedzy o ciśnieniu. |
| Alarm częstych cykli | Zestaw historię z rzeczywistym poborem. W opisanej instalacji takie alarmy wystąpiły podczas zwykłego korzystania z wody. Sam zapis ciśnienia nie mierzy startów silnika. |

[Dokumentacja ADS1115 w ESPHome](https://esphome.io/components/sensor/ads1115/) · [Filtry sensorów ESPHome](https://esphome.io/components/sensor/)
