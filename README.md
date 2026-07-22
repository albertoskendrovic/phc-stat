# PHC — Arhiva prvenstava i statistika Hrvatskog curling saveza

Statična web aplikacija za pregled povijesti prvenstava (Poretci, Rezultati, Sastavi), agregatnu statistiku (Klubovi, Ekipe, Igrači, NHC bodovi), međusobne susrete i sustav postignuća (bedževi) igrača.

Nema poslužiteljskog dijela — sve radi izravno iz statičnih datoteka preko GitHub Pagesa.

## Datoteke u repozitoriju

| Datoteka / mapa | Što je |
|---|---|
| `index.html` | Cijela aplikacija (obje stranice: "Prvenstva" i "Statistika") |
| `badges-data.js` | Podaci o bedževima (SVG ikonice + kriteriji + popisi igrača po razini) |
| `phc-data.json` | Sve utakmice, prvenstva i klupske veze — generira se iz Excela (vidi ispod) |
| `podaci/` | PDF zapisnici, fotografije i videi po prvenstvu (vidi strukturu ispod) |
| `generate-data.py` | Skripta koja phc-data.xlsx pretvara u `phc-data.json` |

`index.html` i `badges-data.js` moraju biti u **istoj mapi** (root repozitorija) — prvi učitava drugi preko `<script src="badges-data.js">`.

## Kako ažurirati podatke o utakmicama (phc-data.xlsx)

1. Izvezite najnoviju verziju baze u `phc-data.xlsx` (listovi: `Utakmice`, `Prvenstva`, `DBIgraci`, `DBEkipe`)
2. Pokrenite:
   ```
   pip install pandas openpyxl --break-system-packages
   python3 generate-data.py phc-data.xlsx phc-data.json
   ```
3. Zamijenite `phc-data.json` u repozitoriju novom datotekom, commit + push

Ako umjesto skripte pošaljete `phc-data.xlsx` u razgovor s Claudeom, on može isto ovo napraviti umjesto vas.

## Kako dodati fotografije, videe i zapisnike

Struktura mape `podaci/`:

```
podaci/
  {sezona-1}-{sezona}/          npr. 2025-2026
    {disciplina}-{kategorija}/  npr. M-S (Muškarci-Seniori)
      PHC-{sezona}-{dis}-{kat}-Sustav.pdf       sustav i raspored (opcionalno)
      PHC-{sezona}-{dis}-{kat}-{skupina}-{faza}-{broj}.pdf   zapisnik utakmice
      1.jpg, 2.jpg, 3.jpg, ...   fotografije, redom bez preskakanja
      1.mp4, 2.mp4, ...         video zapisi, redom bez preskakanja
      foto.txt                  natpisi ispod fotografija (opcionalno)
      video.txt                 natpisi ispod videa (opcionalno)
```

**Kodovi discipline**: `M` (muškarci), `Ž` (žene), `MC` (mješoviti curling), `MP` (mješoviti parovi)
**Kodovi kategorije**: `S` (seniori), `J` (juniori), `V` (veterani)

Format `foto.txt` / `video.txt` (jedan redak po natpisu, ostali brojevi bez natpisa se preskaču):
```
3: Brončana medalja: Vis
5: Pobjednici finala
```

Ako datoteka za neko prvenstvo ne postoji (npr. nema fotografija), taj dio se jednostavno ne prikazuje — ništa dodatno nije potrebno podešavati.

**Video format**: koristite `.mp4` (H.264). Format `.MOV` s iPhonea često ne radi u svim preglednicima.

## Kako ažurirati samu aplikaciju (izgled, funkcionalnost)

1. Klonirajte repozitorij lokalno (GitHub Desktop: File → Clone repository)
2. Zatražite izmjenu od Claudea, preuzmite ažuriranu `index.html` (i/ili `badges-data.js`)
3. Prekopirajte preuzetu datoteku u lokalnu mapu repozitorija (prepišite postojeću)
4. U GitHub Desktopu: Commit → Push origin

## Bedževi (Postignuća)

Bedževi za igrače (brončani/srebrni/zlatni) računaju se automatski iz `phc-data.json` prema pragovima definiranima u `badges-data.js` (`BADGE_META`). Ako se promijene pragovi ili se doda novi bedž, potrebno je ponovno izračunati `BADGES` podatke u `badges-data.js` (zatražite od Claudea).

## Lokalno testiranje

Budući da `index.html` učitava `badges-data.js` i `phc-data.json` preko `fetch`/`<script src>`, otvaranje datoteke izravno dvoklikom (`file://`) neće raditi u svim preglednicima. Pokrenite lokalni server u mapi repozitorija, npr.:

```
python3 -m http.server 8080
```

pa otvorite `http://localhost:8080/index.html`.

## Tehnički detalji

- Bez build koraka, bez npm/webpack — čisti HTML/CSS/JavaScript
- NHC bodovi ("Najbolji hrvatski curlingaši") računaju se prema formuli: `100 × (1 − (mjesto−1)/broj_ekipa)²`, gledajući zadnjih 5 sezona s težinom 100%/80%/60%/40%/20%
- Datumi u `phc-data.xlsx` su formata DD.MM.GGGG — `generate-data.py` ih parsira s `dayfirst=True`
