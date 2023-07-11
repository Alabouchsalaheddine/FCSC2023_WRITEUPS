# Original challenge 🇫🇷

Un de vos collègues ne jure que par cette méthode de chiffrage révolutionnaire appelée rot13.

Il l'a utilisée pour dissimuler un flag dans ce texte. Démontrez-lui qu'il a tort de supposer que cet algorithme apporte une quelconque notion de confidentialité !

GBQB yvfgr :
- Cnva (2 onthrggrf)
- Ynvg (1 yvger)
- Pbevnaqer (fhegbhg cnf, p'rfg cnf oba)
- 4 onanarf, 4 cbzzrf, 4 benatrf
- Cbhyrg (4 svyrgf qr cbhyrg)
- 1 synt : SPFP{rq24p7sq86p2s0515366}
- Câgrf (1xt)
- Evm (fnp qr 18xt)
- Abheve zba qvabfnher

# Translated challenge 🇺🇸

A colleague of yours swears by this revolutionary encryption method called rot13.

He used it to hide a flag in this text. Show him that he is wrong to assume that this algorithm brings any notion of confidentiality!

GBQB yvfgr :
- Cnva (2 onthrggrf)
- Ynvg (1 yvger)
- Pbevnaqer (fhegbhg cnf, p'rfg cnf oba)
- 4 onanarf, 4 cbzzrf, 4 benatrf
- Cbhyrg (4 svyrgf qr cbhyrg)
- 1 synt : SPFP{rq24p7sq86p2s0515366}
- Câgrf (1xt)
- Evm (fnp qr 18xt)
- Abheve zba qvabfnher

# Solution

On [Wikipedia](https://en.wikipedia.org/wiki/ROT13), ROT13 ("rotate by 13 places", sometimes hyphenated ROT-13) is a simple letter substitution cipher that replaces a letter with the 13th letter after it in the latin alphabet. ROT13 is a special case of the Caesar cipher which was developed in ancient Rome.

It's a simple crypto challenge that can be decoded using a simple Python script or using an online decoder, in my case, I used [this one](https://www.dcode.fr/rot-13-cipher).

TODO liste :
- Pain (2 baguettes)
- Lait (1 litre)
- Coriandre (surtout pas, c'est pas bon)
- 4 bananes, 4 pommes, 4 oranges
- Poulet (4 filets de poulet)
- 1 flag : **FCSC{ed24c7fd86c2f0515366}**
- Pntes (1kg)
- Riz (sac de 18kg)
- Nourir mon dinosaure

Pretty simple, we can see that the decoded text contains the expected flag: **FCSC{ed24c7fd86c2f0515366}**
