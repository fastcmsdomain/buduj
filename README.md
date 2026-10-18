# Buduj

Gra 3D w przeglądarce: jesteś budowniczym w lesie z klocków. Kopiesz drewno,
kamień, ziemię i piasek, przepływasz jeziora, szukasz rozsianych po lesie domków,
a potem stawiasz z tego, co zdobędziesz. Po mapie chodzą sarny, krowy i kury, a przy domkach mieszkają ludzie.

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
| `spacja` | skok, dwa razy pod rząd — tryb latania; w wodzie wypływanie w górę |
| `shift` | bieg (w locie i w wodzie: w dół) |
| `1`–`9` | wybór celi z paska (na telefonie: dotknięcie celi) |
| `I` lub `0` | skrzynia z zebranymi surowcami (dziesiąta cela paska) |
| `C` | przy stole w domku: 1 drewno → 4 deski |
| `V` lub `F5` | widok zza pleców |
| `R` | zapis |
| `Esc` | pauza |

Na telefonie: lewa połowa ekranu chodzi, prawa rozgląda, a przyciski po prawej
kopią, stawiają i skaczą.

## Jak to działa

- **Świat** — 288 × 48 × 288 klocków w jednej tablicy `Uint8Array`, czyli ponad
  20 razy większy teren niż pierwotne 64 × 64. Teren powstaje
  z kilku warstw szumu wartościowego z ziarnem, więc ten sam świat da się
  odtworzyć z samego ziarna. Drzewa rosną tam, gdzie osobny szum lasu przekracza
  próg i jest dość miejsca.
- **Rysowanie** — mapa jest pocięta na kawałki 32 × 32 kolumny, a w każdym
  kawałku stoi po jednym `THREE.InstancedMesh` na rodzaj klocka. Rysowane są
  tylko klocki mające choć jednego sąsiada z powietrzem, czyli skorupa świata
  (~170 tysięcy sześcianów zamiast 4 milionów), a z tego co klatkę pokazywane są
  wyłącznie kawałki w polu widzenia kamery i w zasięgu mgły — zwykle ok. 110
  wywołań rysowania. Po kopnięciu albo postawieniu klocka aktualizowane są
  wyłącznie zmienione instancje i ich sąsiedzi w jednym kawałku.
- **Tekstury** — rysowane w locie na `<canvas>`: dla każdego rodzaju klocka
  atlas 48 × 16 px z kafelkami góry, boku i spodu. Każda ściana sześcianu ma
  własną jasność zapisaną w kolorach wierzchołków, dzięki czemu bryły są
  czytelne nawet w płaskim świetle.
- **Fizyka** — własne AABB gracza (0,6 × 1,78) z osobnym rozwiązywaniem kolizji
  na każdej osi i automatycznym wchodzeniem na jeden klocek w górę.
- **Celowanie** — promień po kratce algorytmem Amanatidesa–Woo, zasięg 6 klocków.
- **Woda** — niecki poniżej poziomu 11 są zalane i dodatkowo pogłębione, więc
  jeziora mają średnio 4–5 klocków głębokości (najgłębsze 8). Woda nie blokuje
  ruchu, ale też nie zasłania sąsiadów, więc spod tafli widać piaszczyste dno;
  z samej wody rysowane są tylko klocki stykające się z powietrzem, czyli tafla
  i ściany przy brzegu. W wodzie opada się powoli, spacja wynosi w górę, shift
  zanurza, a pod powierzchnią ekran dostaje niebieski filtr i gęstszą mgłę.
  Promień celowania przechodzi przez wodę, więc kopie się dno, a wykop poniżej
  poziomu morza zalewa się sam (ograniczone zalewanie wszerz, bez symulacji
  prądów). Klocki można stawiać w wodzie — tak buduje się groble i pomosty.
- **Domki** — po mapie rozstawione jest 15 domków w trzech wzorach: *leśna
  chatka* 5 × 5 (bale w narożnikach, deski, spadzisty dach i okno na trzy
  strony), *dom z gankiem* 7 × 6 (kamienna podmurówka, dwa słupy ganku, sześć
  okien, trzystopniowy dach) i *kamienna chatka* 5 × 4 (bruk, płaski dach,
  komin). Każdy stoi na płaskim gruncie powyżej poziomu wody, dostaje
  podmurówkę wyrównującą teren i wyciętą dookoła polanę, a wzór trafia tam w
  jednym z czterech obrotów. W środku jest łóżko (dwa klocki: poduszka i koc),
  stolik i stół do tworzenia — przy nim klawisz `C` zamienia 1 drewno na
  4 deski. Domki są częścią generowanego świata, więc wynikają z samego ziarna
  i nie zajmują miejsca w zapisie.
- **Ekwipunek** — pasek na dole ma dziesięć cel: dziewięć na rzeczy i dziesiątą
  z guzikiem skrzyni. Zaczyna się pusty i wypełnia tym, co wykopiesz; stos ma 64
  sztuki, a gdy pasek się zapełni, nadmiar idzie do skrzyni (27 cel). Skrzynia
  otwiera się klawiszem `I`, dotknięciem dziesiątej celi albo kliknięciem, gdy
  kursor nie jest zablokowany — przy zablokowanym kursorze przeglądarka nie
  przepuszcza kliknięć w HUD, więc tam działa klawisz. W oknie widać skrzynię
  i pasek obok siebie: kliknięcie w rzecz przerzuca ją między nimi. Każdy rodzaj
  klocka wypada teraz jako on sam — szkło daje szkło, dachówka dachówkę, a meble
  meble — więc rozebrany domek da się postawić od nowa. Zapis pamięta obie
  listy i potrafi wczytać stary zapis sprzed podziału na cele.
- **Zwierzęta** — po świecie chodzi 60 sztuk: 16 krów na łąkach, 20 saren
  w lesie i 24 kury trzymające się domków. Każde zwierzę to kilkanaście kostek,
  ale całe stado rysuje się w 13 wywołaniach — jedna `InstancedMesh` na gatunek
  i kolor, a pozycja, obrót i wymach nóg siedzą w macierzy instancji. Zwierzę
  co kilka sekund losuje, czy stoi, czy idzie, i w którą stronę; wchodzi o
  klocek w górę, schodzi o klocek w dół, ale nie wejdzie do wody ani nie spadnie
  z urwiska, bo sprawdza grunt przed sobą. Ruszają się tylko sztuki w promieniu
  115 klocków od gracza — reszta stoi i nic nie kosztuje.
- **Mieszkańcy** — przy każdym domku mieszka jedna osoba, plus pięciu wędrowców
  w głębi lasu: razem 20 ludzi w trzech wariantach koszuli. Korzystają z tego
  samego systemu co zwierzęta (kostki w instancjach, wspólne chodzenie,
  omijanie wody i urwisk), ale mają dwie własne zasady: pamiętają swój dom
  i zawracają, gdy oddalą się o ponad 22 klocki, a gdy podejdziesz na ok. cztery
  klocki, przystają i odwracają się w twoją stronę.
- **Start** — gracz budzi się na najwyższym trawiastym wzgórzu w pobliżu środka
  mapy, żeby od razu widzieć teren.
- **Zapis** — w `localStorage` ląduje ziarno świata, pozycja, ekwipunek i lista
  samych zmienionych klocków, więc zapis jest mały niezależnie od wielkości mapy.

## Rodzaje klocków

Zbierać i stawiać można czternaście rodzajów: trawa, ziemia, kamień, drewno,
liście, piasek, deski, bruk, szkło, dachówka, łóżko (poduszka i koc), stolik
i stół do tworzenia. Kopanie trawy daje ziemię, kopanie kamienia — bruk, reszta
wypada jako ona sama. Gra zaczyna się z pustymi rękami.

Wody nie da się wykopać ani podnieść — można w niej pływać i zasypywać ją innymi
klockami. Meble mają własne, niższe bryły niż pełny sześcian, więc łóżko jest
niskie, a stolik węższy.
