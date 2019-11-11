# UPDATE JOIN
Používa sa vtedy, ak potrebujeme do klauzuly `SET` použiť hodnotu z inej tabuľky.

## Príklad: 
V tabuľke `Customers` (databáza [Northwind](../sql-examples/Northwind.sql)) sú v stĺpci `Country` názvy krajín.
Vytvorte z týchto hodnôt číselník a zmeňte hodnoty v tabuľke `Customers` tak, aby obsahovali namiesto názvov
čísla príslušných krajín.

## Riešenie:

1. Vytvoríme tabuľku krajín:
   
```TSQL
CREATE TABLE dbo.Krajiny (
    Id int not null identity
    , Nazov varchar(50)
    , CONSTRAINT PK_Krajiny PRIMARY KEY (Id)
);
```

2. Naimportujeme existujúce krajiny z tabuľky `Customers`:

```TSQL
INSERT INTO dbo.Krajiny (Nazov) SELECT DISTINCT Country FROM dbo.Customers;
```

3. Nastavíme pre každého zákazníka číslo jeho zodpovedajúcej krajiny:

```TSQL
UPDATE c
    SET c.Country = k.Id
FROM dbo.Customers c
INNER JOIN dbo.Krajiny k ON k.Nazov = c.Country;
```

4. Zmeníme dátový typ stĺpca `Country` v tabuľke `Customers`:

```TSQL
ALTER TABLE dbo.Customers ALTER COLUMN Country int;
```

5. Vytvoríme referenčnú integritu (cudzí kľúč):

```TSQL
ALTER TABLE dbo.Customers ADD CONSTRAINT FK_Customers_Krajiny FOREIGN KEY (Country) REFERENCES dbo.Krajiny (Id);
```