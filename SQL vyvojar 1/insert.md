> Pozn. uvedené príklady pracujú s databázou TSQL, ktorej inštalačný skript je možné stiahnuť [tu](../tsql.sql).

# Príkaz insert
## Vkladanie jedného záznamu:
```TSQL
INSERT INTO Sales.OrderDetails (orderid, productid, unitprice, qty, discount) VALUES (10248, 1, 12.50, 5, 0);
```

Vloží do tabuľky OrderDetails v schéme Sales hodnoty podľa tabuľky:

|názov stĺpca|hodnota|
|---|---|
|orderid|10248|
|productid|1|
|unitprice|12.50|
|qty|5|
|discount|0|

## Vkladanie viacerých záznamov
Tento zápis je vhodné použiť, ak chceme vložiť viacero záznamov naraz. Výhodou a zároveň nevýhodou je,
že ak zlyhá vloženie v čo i len jednom prípade, nevloží sa žiadny záznam.

```TSQL
INSERT INTO Sales.OrderDetails (orderid, productid, unitprice, qty, discount) 
VALUES  (10248, 2, 1, 5, 0), 
        (10248, 3, 1, 5, 0), 
        (10248, 3, 1, 5, 0), 
        (10248, 4, 1, 5, 0), 
        (10248, 5, 1, 5, 0);
```

Vloží do tabuľky OrderDetails v schéme Sales hodnoty podľa tabuľky:

|orderid|productid|unitprice|qty|discount|
|---|---|---|---|---|
|10248|2|1|5|0|
|10248|3|1|5|0|
|10248|4|1|5|0|
|10248|5|1|5|0|

Majme nasledovný príkaz:

```TSQL
INSERT INTO Sales.OrderDetails (orderid, productid, unitprice, qty, discount) 
VALUES  (10248, 5, 1, 5, 0), 
        (10248, 6, 1, 5, 0), 
        (10248, 7, 1, 5, 0), 
        (10248, 8, 1, 5, 0);
```

Tento príkaz nevloží žiadny riadok, pretože kombinácia orderid-productid je primárnym kľúčom 
tabuľky a kombinácia 10248-5 už v tabuľke existuje.

## Práca s hodnotami NULL a DEFAULT
Vytvoríme si skúšobnú databázu Test a tabuľku dbo.Ludia:
```TSQL
CREATE DATABASE Test;
GO
USE Test;
GO
CREATE TABLE dbo.Ludia (
    Id int not null identity(100,1),
    Meno varchar(50),
    Priezvisko varchar(50) not null,
    Pohlavie char(1) not null default '?',
    DatumNarodenia date,
    Telefon varchar(50),
    Email varchar(50)
    CONSTRAINT PK_Ludia PRIMARY KEY (Id)
);
```
**Zopár poznámok k tabuľke Ludia**
- Tabuľka má zapnuté automatické číslovanie na stĺpci Id, prvá hodnota bude 100 a každý 
ďalší záznam bude mať hodnotu o 1 vyššiu, ako posledná vložená hodnota. Ak zmažeme ľubovoľný riadok,
hodnota identity sa už nikdy nepoužije. Ak chceme hodnoty stĺpca Id začať opäť číslovať od 1, musíme
buď vykonať príkaz 
```TSQL
TRUNCATE TABLE dbo.Ludia;
```
alebo musíme zmazať všetky dáta pomocou
```TSQL
DELETE FROM dbo.Ludia;
```
a následne vykonať RESEED identity pomocou 
```TSQL
DBCC CHECKIDENT('dbo.Ludia', RESEED, 1);
```

- Do stĺpca `Priezvisko` sa vždy musí vložiť hodnota a táto nikdy nesmie byť NULL. Ak hodnotu nevložíme (v príkaze `INSERT` nespomenieme stĺpec `Priezvisko`, príkaz skončí s chybou `Cannot insert the value NULL into column 'Priezvisko', table 'Test.dbo.Ludia'; column does not allow nulls. INSERT fails.`)
- Do stĺpca `Pohlavie` sa vždy musí vložiť hodnota a táto nikdy nesmie byť NULL. Ak hodnotu nevložíme, vloží sa štandardná hodnota '?'.

### Príklady:
Korektné vloženie záznamu - použijú sa všetky zadané hodnoty:
```TSQL
INSERT INTO Ludia (Meno, Priezvisko, Pohlavie, DatumNarodenia, Telefon, Email)
VALUES ('Martin', 'Zeleny', 'm', '1986-01-16', '0900123456', 'martin@zeleny.sk');
```
Korektné vloženie záznamu - nemusíme dodržať poradie stĺpcov, ktoré definuje tabuľka (všimnite si pozíciu stĺpca `Priezvisko`).
```TSQL
INSERT INTO Ludia (Meno, Pohlavie, DatumNarodenia, Telefon, Email, Priezvisko)
VALUES ('Jana', 'z', '1986-01-16', '0900654321', 'jana@modra.sk', 'Modra');
```
Chyba - nevyplnili sme stĺpec `Priezvisko` a ten nesmie nadobudnúť hodnotu `NULL` a zároveň nemá nastavenú `DEFAULT` hodnotu:
```TSQL
INSERT INTO Ludia (Meno, Pohlavie, DatumNarodenia, Telefon, Email)
VALUES ('Ivan', 'm', '1986-01-16', '0900789456', 'ivan@cerveny.sk');

--> Cannot insert the value NULL into column 'Priezvisko', table 'Test.dbo.Ludia'; column does not allow nulls. INSERT fails.
```
Chyba - do stĺpca pohlavie sme vložili viac ako 1 znak, došlo by k odrezaniu dát, príkaz sa nevykoná:
```TSQL
INSERT INTO Ludia (Meno, Priezvisko, DatumNarodenia, Telefon, Email, Pohlavie)
VALUES ('Annamaria', 'Ruzova', '1986-01-16', '0900147258', 'annamaria@ruzova.sk', 'zena');

--> String or binary data would be truncated.
```
Oprava chyby - ak chceme povoliť odrezanie hodnôt, môžeme použiť explicitnú typovú konverziu na správny dátový typ:
```TSQL
INSERT INTO Ludia (Meno, Priezvisko, DatumNarodenia, Telefon, Email, Pohlavie)
VALUES ('Annamaria', 'Ruzova', '1986-01-16', '0900147258', 'annamaria@ruzova.sk', CAST('zena' AS char(1)));
```

## INSERT INTO ... SELECT
Pomocou tohto zápisu je možné skopírovať dáta z tabuľky do inej tabuľky:
```TSQL
INSERT INTO Ludia (Meno, Priezvisko, DatumNarodenia, Telefon)
SELECT FirstName, LastName, BirthDate, HomePhone FROM Northwind.dbo.Employees;
```

## SELECT ... INTO ... FROM ...
Pomocou tohto zápisu sa vytvorí nová tabuľka a naplní sa dátami definovanými v príkaze `SELECT`
```TSQL
SELECT FirstName, LastName, BirthDate, HomePhone INTO Ludia2 FROM Northwind.dbo.Employees;
```
Je možné premenovať stĺpce v cieľovej tabuľke pomocou aliasov:
```TSQL
SELECT 
    FirstName AS Meno, 
    LastName AS Priezvisko, 
    BirthDate AS DatumNarodenia, 
    HomePhone AS Telefon 
INTO Ludia3 
FROM Northwind.dbo.Employees; 
```