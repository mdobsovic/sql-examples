# Práca s `IDENTITY`

- Nastavenie stĺpca, ktoré bude do stĺpca vkladať hodnotu z postupnosti definovanej v parametri `IDENTITY`
- Je možné spraviť iba jeden stĺpec s parametrom `IDENTITY` v rámci tabuľky
- Pri definícii `IDENTITY` musíme buď zadefinovať seed aj increment, alebo oba vynecháme

## Definícia stĺpca s `IDENTITY`:
Vytvoríme tabuľu IdntTbl s dvoma stĺpcami:

```TSQL
    CREATE TABLE IdntTbl (
        Id int identity
        , Stlpec2 varchar(50)
    )
```
Na stĺpci `Id` je nastavený parameter `IDENTITY`. Keďže sme nezadali `seed` a `increment`, oba parametre 
budú mať hodnotu 1, čo znamená, že prvý riadok dostane Id = 1 (`seed`) a každý ďalší hodnotu o 1 vyššiu (`increment`).

```TSQL
    CREATE TABLE IdntTbl2 (
        Id int identity(50, 5)
        , Stlpec2 varchar(50)
    )
```
Vyššie uvedený príkaz vytvorí tabuľku s `IDENTITY` stĺpcom Id s tým, že prvá hodnota bude 50 (`seed`) a každý ďalší riadok
dostane hodnotu o 5 vyššiu (`increment`).

> Všimnite si, že vo vyššie uvedených príkladoch nie sú vytvorené primárne kľúče. To znamená, že stĺpec s parametrom `IDENTITY` 
> môžeme použiť bez ohľadu na to, či je alebo nie je súčasťou primárneho kľúča.

## Zistenie hodnoty `IDENTITY` posledného vloženého riadka:
Existujú 3 možnosti, ako zistiť hodnotu `IDENTITY`:

### Premenná @@IDENTITY

```TSQL
    SELECT @@IDENTITY;
```

Vráti sa hodnota IDENTITY, ktorá bola použitá v poslednom príkaze INSERT, ktorý vykonalo moje spojenie (aktuálne okno).
Neberie sa ohľad na to, do ktorej tabuľky bola hodnota vložená. Majme nasledujúci príklad:

```TSQL
    INSERT INTO Ludia (Meno, Priezvisko) VALUES ('Jozef', 'Mak');
    SELECT @@IDENTITY; --> 5  (Id vloženého človeka Jozef Mak)
    INSERT INTO Oddelenia (Nazov) VALUES ('IT oddelenie');
    SELECT @@IDENTITY; --> 13 (Id vloženého oddelenia IT oddelenie)
```

### Funkcia SCOPE_IDENTITY()

Vytvorme si procedúru, kt. vloží nový záznam do tabuľky Ludia a vypíše Id vloženého záznamu:
```TSQL
    CREATE PROCEDURE VlozCloveka @meno varchar(50), @priezvisko varchar(50) AS BEGIN
        INSERT INTO Ludia (Meno, Priezvisko) VALUES (@meno, @priezvisko);
        PRINT CONCAT('Vložil sa záznam s Id = ', SCOPE_IDENTITY());
    END
    GO
```

Majme nasledovný skript:

```TSQL
    USE Test;
    GO
    SELECT SCOPE_IDENTITY(); --> NULL
    INSERT INTO Ludia (Meno, Priezvisko) VALUES ('Peter', 'Malý');
    SELECT SCOPE_IDENTITY(); --> 24 (Id záznamu Peter Malý)
    EXECUTE VlozCloveka 'Martin', 'Rýchly'; --> Vložil sa záznam s Id = 25
    SELECT @@IDENTITY; --> 25
    SELECT SCOPE_IDENTITY(); --> 24
```

Vo vyššie uvedenom príklade je evidentné, že funkcia `SCOPE_IDENTITY()` pracuje na úrovni aktuálneho "modulu" - pozerá iba na to,
čo sa deje v skripte ALEBO v procedúre (podľa toho, kde ju použijeme).

### Posledná vložená hodnota IDENTITY v konkrétnom objekte:

```TSQL
    SELECT IDENT_CURRENT('Ludia'); --> 25
```

Funkcia `IDENT_CURRENT()` vráti poslednú vloženú `IDENTITY` hodnotu bez ohľadu na to, kto a kedy ju vložil. Rovnako sa neberie ohľad na to, či záznam v tabuľke ešte existuje. Majme teda nasledovný príklad:

```TSQL
    USE Test;
    GO
    TRUNCATE TABLE Ludia; -- Aby sme vyčistili dáta a resetli IDENTITY na hodnotu seed (1)
    GO
    INSERT INTO Ludia (Meno, Priezvisko) VALUES ('Daniel', 'Šikovný'); -- záznam dostane Id = 1
    GO
    SELECT IDENT_CURRENT('Ludia'); --> 1
    GO
    DELETE FROM Ludia;
    GO
    SELECT IDENT_CURRENT('Ludia'); --> 1
    GO
    INSERT INTO Ludia (Meno, Priezvisko) VALUES ('Viktória', 'Bystrá'); -- záznam dostane Id = 2
    GO
    SELECT IDENT_CURRENT('Ludia'); --> 2
    GO
    DELETE FROM Ludia;
    GO
    SELECT IDENT_CURRENT('Ludia'); --> 2
    GO
    TRUNCATE TABLE Ludia;
    GO
    SELECT IDENT_CURRENT('Ludia'); --> NULL
```