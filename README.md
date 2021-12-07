# Tutoriel pour les noob

## I. Smart Contract

### 1. Variables

contract MIM
TOKEN_AMOUNT
MIM_AMOUNT
MAX_CAP_PERSON
MAX_SUPPLY
SALE_START

### 2. Deployer
### 3. Verifier le contract

> Single file
> comiler: 0.7.5
> License: On s'en bat les couille

> Optimzation: true
> Paste le smart contract + RAT
> ABI: tout supprimer
> Nb optimisation: 200
> Alghorithme: byzantium

## II. Frontend

### 1. Index.vue

Le logo l.16
Discord l.57
Twitter l.63
Medium l.69

### 2. Sale.vue

Emoji l.14
Token name l.91
getMimContract l.161 (Changer l'address par celle de MIM)
presaleAddress: l.175 (Changer l'address par l'address du contract deployer)
cap: l.222 (Max que les utilisateur peuvent acheter doit etre coherant avec le smart contract)
allocated: l.224 et 225 (Max que un utilisateur peut depenser dans la presale whiteliste ou les autres)
dates: l.235 et 236 (Temps en coherance avec le smart contract du depart de la presale /!\ x1000 que le timestamp du contract)
whitelistText: l.247 (Apres avoir acheter le nom domaine changer /whitelist.txt -> {domaine}/whitelist.txt)
goodChain: l.432 (Rajouter le chainId correct)
Rajouter MetMask chain: l.480 - 483




