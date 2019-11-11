# Príkaz MERGE
- slúži na spojenie dvoch tabuliek - z jednej tabuľky nahráva záznamy do druhej a umožňuje nastaviť, čo sa má stať, ak sa zodpovedajúca hodnota našla, resp. nenašla.

## Prípravný skript:
```TSQL
USE master;
GO
IF EXIST (SELECT * FROM sysdatabases WHERE name='Test')
		DROP DATABASE Test;
GO
CREATE DATABASE Test;
GO
USE Test;
GO
CREATE TABLE dbo.Ludia (
    Id int not null identity(1,1),
    Meno varchar(50),
    Priezvisko varchar(50) not null,
    Pohlavie char(1) not null default '?',
    DatumNarodenia date,
    Telefon varchar(50),
    Email varchar(50)
    CONSTRAINT PK_Ludia PRIMARY KEY (Id)
);
GO
INSERT dbo.Ludia (Meno, Priezvisko, Pohlavie, DatumNarodenia, Telefon, Email)
VALUES  ('Peter', 'Modry', 'm', '1990-01-05', '0900285828', 'peter@modry.sk')
        , ('Roman', 'Zeleny', 'm', '1986-04-28', '0900381947', 'roman@zeleny.sk')
        , ('Michal', 'Cerveny', 'm', '1975-12-23', '0900381928', 'michal@cerveny.sk')
        , ('Boris', 'Cierny', 'm', '1985-03-29', '0900371482', 'boris@cierny.sk');
GO
CREATE TABLE dbo.Ludia2 (
    Id int not null identity(1,1),
    Meno varchar(50),
    Priezvisko varchar(50) not null,
    Pohlavie char(1) not null default '?',
    DatumNarodenia date
    CONSTRAINT PK_Ludia2 PRIMARY KEY (Id)
);
GO
INSERT dbo.Ludia2 (Meno, Priezvisko, Pohlavie, DatumNarodenia)
VALUES  ('Zuzana', 'Ruzova', 'z', '1995-05-02')
        , ('Maria', 'Fialova', 'z', '1964-02-28')
        , ('Lucia', 'Cierna', 'z', '1982-05-14')
        , ('Boris', 'Cierny', 'm', '1990-01-01');
GO
```

Vyššie uvedený skript premaže databázu `Test` ak existuje a vytvorí v nej dve tabuľky:
### dbo.Ludia:

|Id|Meno|Priezvisko|Pohlavie|DatumNarodenia|Telefon|Email|
|---|---|---|---|---|---|---|
|1|Peter|Modry|m|1990-01-05|0900285828|peter@modry.sk|
|2|Roman|Zeleny|m|1986-04-28|0900285828|roman@zeleny.sk|
|3|Michal|Cerveny|m|1975-12-23|0900285828|michal@cerveny.sk|
|4|Boris|Cierny|m|1985-03-29|0900285828|boris@cierny.sk|

### dbo.Ludia2:

|Id|Meno|Priezvisko|Pohlavie|DatumNarodenia|
|---|---|---|---|---|
|1|Zuzana|Ruzova|z|1995-05-02|
|2|Maria|Fialova|z|1964-02-28|
|3|Lucia|Cierna|z|1982-05-14|
|4|Boris|Cierny|m|1990-01-01|

## Príklad:
Prekopírujte všetky záznamy z tabuľky `dbo.Ludia2` do tabuľky `dbo.Ludia`. 
Ak už záznam v tabuľke `dbo.Ludia` existuje, aktualizujte dátum narodenia podľa hodnoty v tabuľke `dbo.Ludia2`.
Zhodné záznamy vyhľadávajte podľa zhody stĺpcov `Meno` a `Priezvisko`.

## Riešenie:

```TSQL
MERGE
    INTO dbo.Ludia AS l
    USING dbo.Ludia2 AS l2
    ON (l.Meno = l2.Meno AND l.Priezvisko = l2.Priezvisko)
    WHEN NOT MATCHET BY TARGET THEN
        INSERT (Meno, Priezvisko, Pohlavie, DatumNarodenia) VALUES (l2.Meno, l2.Priezvisko, l2.Pohlavie, l2.DatumNarodenia)
    WHEN MATCHED THEN 
        UPDATE SET l.DatumNarodenia = l2.DatumNarodenia
;
```

## Výsledok:

```TSQL
SELECT * FROM dbo.Ludia;
```

Zmenené a vložené hodnoty sú vyznačené **tučným písmom**.

|Id|Meno|Priezvisko|Pohlavie|DatumNarodenia|Telefon|Email|
|---|---|---|---|---|---|---|
|1|Peter|Modry|m|1990-01-05|0900285828|peter@modry.sk|
|2|Roman|Zeleny|m|1986-04-28|0900285828|roman@zeleny.sk|
|3|Michal|Cerveny|m|1975-12-23|0900285828|michal@cerveny.sk|
|4|Boris|Cierny|m|**1990-01-01**|0900285828|boris@cierny.sk|
|**5**|**Zuzana**|**Ruzova**|**z**|**1995-05-02**|**NULL**|**NULL**|
|**6**|**Maria**|**Fialova**|**z**|**1964-02-28**|**NULL**|**NULL**|
|**7**|**Lucia**|**Cierna**|**z**|**1982-05-14**|**NULL**|**NULL**|