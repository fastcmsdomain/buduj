# Buduj

Gra 3D w przeglądarce: jesteś budowniczym w lesie z klocków. Kopiesz drewno,
kamień, ziemię i piasek, a potem stawiasz z nich, co chcesz.

Cały kod to jeden plik `index.html` — bez instalacji, bez budowania, bez serwera.
Wystarczy otworzyć plik w przeglądarce. Jedyna zewnętrzna zależność to
[three.js](https://threejs.org/) r128 wczytywane z CDN.

## Sterowanie

| Klawisz | Działanie |
| --- | --- |
| `W` `A` `S` `D` | chodzenie |
| mysz | rozglądanie się (po kliknięciu gra blokuje kursor; bez blokady — przeciąganie) |
| lewy / prawy przycisk | kopanie / stawianie klocka |
| `E` / `Q` | to samo z klawiatury |
| `spacja` | skok, dwa razy pod rząd — tryb latania |
| `shift` | bieg (w locie: w dół) |
| `1`–`8`, kółko myszy | wybór klocka z paska |
| `V` lub `F5` | widok zza pleców |
| `R` | zapis |
| `Esc` | pauza |

Na telefonie: lewa połowa ekranu chodzi, prawa rozgląda, a przyciski po prawej
kopią, stawiają i skaczą.

## Jak to działa

- **Świat** — 64 × 48 × 64 klocków w jednej tablicy `Uint8Array`. Teren powstaje
  z kilku warstw szumu wartościowego z ziarnem, więc ten sam świat da się
  odtworzyć z samego ziarna. Drzewa rosną tam, gdzie osobny szum lasu przekracza
  próg i jest dość miejsca.
- **Rysowanie** — jeden `THREE.InstancedMesh` na rodzaj klocka. Rysowane są
  tylko klocki mające choć jednego sąsiada z powietrzem, czyli skorupa świata
  (~7 tysięcy sześcianów zamiast 200 tysięcy). Po kopnięciu albo postawieniu
  klocka aktualizowane są wyłącznie zmienione instancje i ich sąsiedzi.
- **Tekstury** — rysowane w locie na `<canvas>`: dla każdego rodzaju klocka
  atlas 48 × 16 px z kafelkami góry, boku i spodu. Każda ściana sześcianu ma
  własną jasność zapisaną w kolorach wierzchołków, dzięki czemu bryły są
  czytelne nawet w płaskim świetle.
- **Fizyka** — własne AABB gracza (0,6 × 1,78) z osobnym rozwiązywaniem kolizji
  na każdej osi i automatycznym wchodzeniem na jeden klocek w górę.
- **Celowanie** — promień po kratce algorytmem Amanatidesa–Woo, zasięg 6 klocków.
- **Zapis** — w `localStorage` ląduje ziarno świata, pozycja, ekwipunek i lista
  samych zmienionych klocków, więc zapis jest mały niezależnie od wielkości mapy.

## Rodzaje klocków

Trawa, ziemia, kamień, drewno, liście, piasek, deski i bruk. Kopanie trawy daje
ziemię, kopanie kamienia — bruk. Na start dostajesz 24 deski.
