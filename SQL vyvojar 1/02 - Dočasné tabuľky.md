# Dočasné tabuľky
- vytvoriť ich môže ktorýkoľvek používateľ, ak má práva prihlásiť sa na databázový server
- vytvárajú sa v databáze `tempdb`
- zanikajú najneskôr vtedy, keď dôjde k ukončeniu session, v ktorej bola tabuľka vytvorená

## Lokálna dočasná tabuľka
- je viditeľná iba v rámci aktuálnej session (jedno okno v Management Studiu)
- jej názov začína na jednu mriežku
```TSQL
SELECT FirstName, LastName, BirthDate, HomePhone INTO #Ludia FROM Northwind.dbo.Employees;
```

## Globálna dočasná tabuľka
- vidia ju všetci používatelia
- jej názov začína na dve mriežky
```TSQL
SELECT FirstName, LastName, BirthDate, HomePhone INTO ##Ludia FROM Northwind.dbo.Employees; 
```