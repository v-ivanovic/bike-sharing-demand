# Predviđanje potražnje u sistemu javnih bicikala

Poređenje modela mašinskog učenja na satnom i dnevnom nivou.

Student: Vojislav Ivanović, 4004/2024

---

## Problem

Sistemi javnih bicikala su automatizovani sistemi u kojima korisnik uzima bicikl na jednoj stanici i vraća ga na bilo kojoj drugoj. Operateru je važno da unapred zna koliko će bicikala biti potrebno u kom trenutku, jer od toga zavisi raspoređivanje ekipa koje bicikle preraspodeljuju, planiranje održavanja i dimenzionisanje sistema.

Projekat predviđa ukupan broj iznajmljivanja u sistemu Capital Bikeshare u Vašingtonu, na osnovu kalendarskih i vremenskih podataka. Podaci su agregirani na nivou celog sistema, bez informacija o pojedinačnim stanicama, pa model odgovara na pitanje koliko i kada, a ne i gde. Isti problem je posmatran na dva nivoa, satnom i dnevnom, i u dve postavke, kao regresija broja iznajmljivanja i kao klasifikacija nivoa opterećenja na nisko, srednje i visoko.

---

## Skup podataka

[Bike Sharing Dataset](https://archive.ics.uci.edu/dataset/275/bike+sharing+dataset),
UCI Machine Learning Repository. Podaci sistema Capital Bikeshare za 2011. i 2012. godinu, spojeni sa vremenskim podacima.

| Fajl | Jedan red je | Broj redova |
|---|---|---|
| `data/hour.csv` | jedan sat | 17.379 |
| `data/day.csv` | jedan dan | 731 |

Atributi su kalendarski (godina, mesec, sat, dan u nedelji, praznik, radni dan, godišnje doba) i vremenski (temperatura, subjektivna temperatura, vlažnost, brzina vetra, opis vremena). Ciljna promenljiva je `cnt`, ukupan broj iznajmljivanja. Skup sadrži i `casual` i `registered`, broj povremenih i registrovanih korisnika, čiji je `cnt` tačan zbir.

Osobine skupa koje su odredile obradu podataka:

- `cnt` je tačan zbir atributa `casual` i `registered`, pa su oni izbačeni iz ulaza;
- `temp` i `atemp` imaju korelaciju 0.99, pa je `atemp` izbačen;
- veza sata u danu i potražnje je izrazito nelinearna i zavisi od tipa dana;
- rasipanje ciljne promenljive raste sa nivoom potražnje;
- potražnja je u 2012. godini bila 1.65 puta veća nego u 2011, pa je podela podataka hronološka, a ne nasumična;

---

## Modeli

Podela je hronološka: 2011. godina za treniranje i validaciju, 2012. za testiranje. Metaparametri su birani na skupu za validaciju, a svi konačni modeli su obučeni na uniji skupova za treniranje i validaciju.

| Model | Zadatak | Sveska |
|---|---|---|
| Trivijalna osnova, prosek po satu i tipu dana | regresija | 02 |
| Linearna regresija | regresija | 03 |
| Linearna regresija sa težinama | regresija | 03 |
| Metod k najbližih suseda | regresija | 04 |
| Slučajna šuma | regresija | 04 |
| Multinomijalna logistička regresija | klasifikacija | 05 |
| Slučajna šuma | klasifikacija | 05 |
| Potpuno povezana neuronska mreža | regresija | 06 |

Poređenja, matrice konfuzije i zaključci su u sveskama, a zbirno poređenje svih modela u svesci 07.

---

## Rezultati na satnom nivou

Skup za testiranje je 2012. godina. Mere su srednja apsolutna greška (MAE), koren srednje kvadratne greške (RMSE), koeficijent determinacije (R2) i prosečna razlika predviđene i stvarne vrednosti (odstupanje).

| Model | MAE | RMSE | R2 | Odstupanje |
|---|---|---|---|---|
| Trivijalna osnova | 108.5 | 157.1 | 0.435 | -91.9 |
| Linearna regresija | 110.2 | 158.0 | 0.428 | -86.7 |
| Linearna regresija sa težinama | 110.3 | 161.0 | 0.406 | -88.1 |
| Metod k najbližih suseda (k = 50) | 116.3 | 168.5 | 0.349 | -102.7 |
| Slučajna šuma (50 stabala, bez ograničenja dubine) | 93.0 | 129.7 | 0.614 | -81.2 |
| Neuronska mreža (2 sloja po 128 neurona) | 88.1 | 122.5 | 0.656 | -82.4 |

Linearni modeli su lošiji od trivijalne osnove, jer ne mogu da izraze interakciju sata i tipa dana koju osnova koristi. Metod najbližih suseda je lošiji iz drugog razloga: predviđa prosek suseda iz 2011. i ne može da dostigne nivo potražnje iz 2012. Slučajna šuma i neuronska mreža nadmašuju osnovu, jer uče interakcije i uz to koriste vremenske atribute. Svi modeli potcenjuju potražnju u 2012. godini za osamdeset do sto bicikala po satu, jer su obučeni na 2011, pre rasta sistema.

---

## Literatura

[1] Mladen Nikolić, Anđelka Zečević, Mašinsko učenje, Matematički fakultet, Beograd, 2019.
    https://ml.matf.bg.ac.rs/readings/ml.pdf

[2] Fanaee-T, H. (2013). Bike Sharing [Dataset]. UCI Machine Learning Repository.
    https://doi.org/10.24432/C5W894

[3] Fanaee-T, H., Gama, J. (2013). Event labeling combining ensemble detectors and
    background knowledge. Progress in Artificial Intelligence.

[4] Materijali sa vežbi iz predmeta.

[5] Dokumentacija biblioteka scikit-learn, TensorFlow / Keras i pandas.
