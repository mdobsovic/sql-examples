# Hodnota `NULL`
- predstavuje neurčitú hodnotu
- nie je to číslo 0
- nie je to prázdny text `''`

## Porovnanie hodnoty `NULL`

|Výraz|Výsledok|
|---|---|
|`NULL = 0`|`FALSE`|
|`NULL = ''`|`FALSE`|
|`NULL = NULL`|`FALSE`|
|`NULL != NULL`|`FALSE`|
|`NULL IS NULL`|`TRUE`|
|`LEN('')`|`0`|
|`LEN(NULL)`|`NULL`|
|`5 + 0`|`5`|
|`5 + NULL`|`NULL`|


*Vypíšte všetky záznamy z tabuľky `dbo.Ludia`, ktoré nemajú vyplnené telefónne číslo, alebo ich telefónne číslo je prázdne.*
```TSQL
SELECT * FROM dbo.Ludia WHERE LEN(Telefon) = 0 OR Telefon IS NULL;
```
alebo
```TSQL
SELECT * FROM dbo.Ludia WHERE Telefon = '' OR Telefon IS NULL;
```
Takto nie (nevypíšu sa záznamy, kde `Telefon IS NULL`):
```TSQL
SELECT * FROM dbo.Ludia WHERE Telefon IN ('', NULL);
```