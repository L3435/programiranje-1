---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.12
    jupytext_version: 1.8.0
kernelspec:
  display_name: OCaml 4.11
  language: OCaml
  name: ocaml-jupyter
---

# Spremenljive podatkovne strukture

```{code-cell}
:tags: [remove-cell, remove-stdout]

(* Ko se v Jupytru prvič požene OCaml, program Findlib izpiše neko sporočilo.
   Da se to sporočilo ne bi videlo v zapiskih, je tu ta celica, ki sproži izpis,
   vendar ima nastavljeno, da je v zapiskih v celoti skrita. *)
```

Podatkovne strukture, ki smo jih v OCamlu spoznali do sedaj so bile nespremenljive. Res smo govorili, da smo seznamu na začetek dodali glavo, vendar s tem prvotnega seznama nismo spremenili, temveč smo naredili nov seznam, ki je imel starega za rep. V Pythonu smo seznam lahko dejansko spremenili. Na primer, sledeča funkcija `f` ob vsakem klicu seznam `sez` razširi z enim elementom:

```python
sez = [1, 2, 3]
def f(x):
    sez.append(x)
    return len(sez)
def g(x):
    return f(x) + f(x)
```

```python
>>> g(3)
9
>>> f(3) + f(3)
13
>>> g(3)
17
```

Podobne funkcije v OCamlu sploh (še) ne znamo napisati. Tudi če napišemo nekaj podobnega:

```{code-cell}
let sez = [1; 2; 3]
let f x =
    let sez = x :: sez in
    List.length sez
let g x =
    f x + f x
```

funkcija `f` ne spremeni seznama `sez`, temveč naredi nov seznam in ga lokalno poimenuje z istim imenom. Posledično bo ta novi seznam vedno dolžine 4:

```{code-cell}
g 3
```

```{code-cell}
f 3 + f 3
```

```{code-cell}
g 3
```

## Reference

Kot praktičen programski jezik pa OCaml pozna tudi spremenljive podatkovne strukture. Najenostavnejša od njih so _reference_. Reference imajo tip oblike `τ ref` in predstavljajo eno samo spremenljivo vrednost tipa `τ`. Predstavljamo si jo lahko kot sklic (oz. referenco) do nekega prostora v pomnilniku. Novo referenco ustvarimo s funkcijo `ref`, ki sprejme začetno vrednost, spravljeno v prostoru, in vrne sklic nanj:

```{code-cell}
ref
```

```{code-cell}
let r = ref 20
```

Sklic `r` torej kaže na prostor v pomnilniku, v katerem je trenutno shranjena vrednost `20`. Če želimo dostopati do trenutne vrednosti reference, uporabimo operacijo `!`:

```{code-cell}
(!)
```

```{code-cell}
!r + 100
```

Če želimo spremeniti vrednost, uporabimo operacijo `:=`, ki sprejme referenco ter njeno novo vrednost.

```{code-cell}
(:=)
```

Tip rezultata `unit` nam nakazuje, da funkcija ne vrne ničesar koristnega temveč sproži stranski učinek spremembe pomnilnika.

```{code-cell}
r := 5
```

```{code-cell}
!r
```

Kljub temu da so seznami v OCamlu nespremenljivi, pa so reference spremenljive, zato lahko z njimi naredimo program, ki se obnaša kot tisti v Pythonu:

```{code-cell}
let sklic_na_seznam = ref [1; 2; 3]
let f x =
    sklic_na_seznam := x :: !sklic_na_seznam;
    List.length !sklic_na_seznam
let g x =
    f x + f x
```

Tokrat v funkciji `f` nismo s pomočjo `let` definirali nove vrednosti z istim imenom, temveč smo uporabili operacijo `:=` in spremenili vsebino obstoječega sklica.

```{code-cell}
g 3
```

```{code-cell}
f 3 + f 3
```

```{code-cell}
g 3
```

Reference so v resnici samo poseben primer spremenljivih zapisov. Ob definiciji zapisnega tipa v OCamlu lahko namreč nekatera polja s ključno besedo `mutable` označimo za spremenljiva ter jih potem spreminjamo s pomočjo operacije `<-`.

```{code-cell}
type racun = {
  lastnik : string;
  stevilka : int;
  mutable znesek : float;
}
```

```{code-cell}
let moj_racun = {
  lastnik = "jaz";
  stevilka = 27004498;
  znesek = 0.0
}
```

```{code-cell}
moj_racun.znesek
```

```{code-cell}
moj_racun.znesek <- 1000.0
```

```{code-cell}
moj_racun.znesek
```

Reference ter operacije na njih pa bi lahko definirali kot:

```ocaml
type 'a ref = {mutable contents : 'a}

let ref x = {contents = x}

let (!) r = r.contents

let (:=) r x = r.contents <- x
```

## Predstavitev seznamov v pomnilniku

Videli smo že, da je OCamlov tip `list` v pomnilniku predstavljen z verižnimi seznami. Vsak seznam je bodisi prazen, bodisi je sestavljen iz glave in repa. Na primer, seznam `let s = [6; 2; 4]` ustreza predstavitvi

![](slike/seznam-ocaml.png)

Po pomnilniku seveda ne moremo risati puščic. Namesto tega so celice oštevilčene, kazalci pa so potem samo njihovi naslovi:

```ocaml
s : 19         0   0   0   0   0   0
               0   0   0   0   0   0
               0   0   1   2  22   0
               1   6  15   1   4  30
               0   0   0   0   0   0
```

OCaml pa poleg verižnih seznamov pozna tudi tip `array`, kjer so podatko predstavljeni s _tabelami_, torej v zaporednih prostorih v pomnilniku. V tem primeru moramo na začetku tabele podati tudi njeno velikost, da se ve, kje se konča (verižni seznami se končajo, ko kazalec kaže na prazen seznam). Tabele tako kot sezname pišemo v oglate oklepaje, le da dodamo še dve navpični črti. Preden si pogledamo funkcije za delo s tabelami, si oglejmo še njihovo predstavitev v pomnilniku. Tabelo, definirano z `let s = [|6; 2; 4|]`, v pomnilniku predstavimo kot

![](slike/tabela-ocaml.png)

oziroma natančneje kot

```ocaml
s : 19         0   0   0   0   0   0
               0   0   0   0   0   0
               0   0   0   0   0   0
               3   6   2   4   0   0
               0   0   0   0   0   0
```

Pythonovi seznami so prav tako predstavljeni s tabelami in imajo podobno predstavitev kot tip `array` v OCamlu. Ena razlika je, da so Pythonove tabele razširljive, zato poleg podatka o velikosti vsebujejo še podatek o rezerviranem prostoru (če ga velikost preseže, Python rezervira večji kos pomnilnika in tabelo prenese vanj). Poleg tega pa je Python dinamičen jezik, zato mora v pomnilniku hraniti še podatke o trenutnem tipu. Torej je slika Pythonovega seznama `s = [6, 2, 4]` približno taka:

![](slike/tabela-python.png)

malo bolj natančno pa kot

```python
s : 19         0   0 123   6   0   0
               0   0   0   0 123   4
               0   0   0   0   0   0
             246   3   6   3  29  11
               0   0   0   0 123   2
```

kjer si lahko predstavljamo, da sta na mestih `123` in `246` shranjeni še definiciji razredov `int` in `list`. Različne predstavitve v pomnilniku vodijo v različne računske zahtevnosti operacij. Za seznam dolžine $n$ so časovne zahtevnosti osnovnih operacij sledeče:

|        operacija | OCaml `array` | Python `list` | OCaml `list`
| ---------------: | :-----------: | :-----------: | :----------:
|     indeksiranje |     O(1)      |     O(1)      |     O(n)
| dodaj na začetek |     O(n)      |     O(n)      |     O(1)
|   dodaj na konec |     O(n)      |     O(1)      |     O(n)
|          dolžina |     O(1)      |     O(1)      |     O(n)
|     izračun repa |     O(n)      |     O(n)      |     O(1)

## Tip `array`

Tabele v OCamlu so v veliko pogledih podobne seznamom. Tip `'a array` predstavlja tabelo elementov tipa `'a`, v modulu [`Array`](https://caml.inria.fr/pub/docs/manual-ocaml/libref/Array.html) pa so na voljo običajne funkcije:

```{code-cell}
Array.length [|10; 20; 30|]
```

```{code-cell}
Array.map succ [|10; 20; 30|]
```

```{code-cell}
Array.make 3 'a'
```

```{code-cell}
Array.of_list [5; 2; 1; 4; 3; 6; 8; 7]
```

```{code-cell}
Array.make_matrix 2 2 0
```

Tako kot v Pythonu so OCamlove tabele indeksirane in do elementa z indeksom `i` dostopamo prek `tabela.(i)`:

```{code-cell}
let t = [|1; 2; 3|]
```

```{code-cell}
t.(0)
```

```{code-cell}
t.(2)
```

Poleg predstavitve v pomnilniku pa je tip `array` tudi spremenljiv (sicer ne bi bil preveč uporaben, saj bi že za majhno spremembo morali prekopirati celotno tabelo).

```{code-cell}
t
```

```{code-cell}
t.(2) <- 5
```

```{code-cell}
t
```

Tako imamo pri funkcijah na tabelah vedno dve možnosti. Lahko vrnemo novo tabelo ali spremenimo obstoječo na mestu. Na primer, tabelo lahko obrnemo tako, da naredimo novo, v kateri so elementi ustrezno preštevilčeni:

```{code-cell}
let vrni_obrnjeno tabela =
  let n = Array.length tabela in
  Array.init n (fun i -> tabela.(n - i - 1))
```

```{code-cell}
let t = vrni_obrnjeno [|1; 5; 10; 20|]
```

Pri tem se prvotna tabela ni spremenila:

```{code-cell}
t
```

Lahko pa tabelo obrnemo tudi na mestu tako, da prvi element zamenjamo z zadnjim, drugega s predzadnjim in tako naprej do sredine. Pri tem si lahko pomagamo tudi z zanko `for`:

```{code-cell}
let obrni_na_mestu tabela =
  let n = Array.length tabela in
  for i = 0 to n / 2 - 1 do
    let t = tabela.(i) in
    tabela.(i) <- tabela.(n - i - 1);
    tabela.(n - i - 1) <- t
  done
```

```{code-cell}
let t' = [|3; 33; 333; 3333|]
```

```{code-cell}
obrni_na_mestu t'
```

Funkcija `obrni_na_mestu` ni vrnila ničesar, je pa imela stranski učinek, da se je spremenila tabela `t'`:

```{code-cell}
t'
```

## Zanki `for` in `while`

Kot smo videli, OCaml pozna tudi zanko `for`, ki pa je v primerjavi s Pythonovo precej omejena. V njej imamo lahko le eno spremenljivko, ki se sprehaja le naraščajoče ali padajoče po zaporednih celih številih. Zanke ne vračajo ničesar, zato tudi zahtevamo, da je njihovo telo tipa `unit`.

```{code-cell}
for i = 1 to 9 do
  print_endline (string_of_int i)
done
```

```{code-cell}
for i = 9 downto 1 do
  print_endline (string_of_int i)
done
```

Poleg zanke `for` poznamo tudi zanko `while`, ki telo tipa `unit` ponavlja toliko časa, dokler je izpolnjen pogoj tipa `bool`. Če nočemo neskončne zanke, je koristno, da za pogoj izberemo spremenljiv izraz, npr. tak, ki uporablja reference:

```{code-cell}
let i = ref 10
```

```{code-cell}
while !i > 1 do
  decr i;
  print_endline (string_of_int !i)
done
```

## Fisher-Yatesov algoritem

## [Naivni algoritmi za urejanje](https://visualgo.net/en/sorting)
