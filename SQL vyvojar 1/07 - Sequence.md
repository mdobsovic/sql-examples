# Postupnosti - `SEQUENCE`
Sequence je flexibilnejšia podoba `IDENTITY`, ktorá:
- je samostatný objekt v schéme, teda nie je súčasťou tabuľky (ako `IDENTITY`)
- môže byť zdieľaná viacerými stĺpcami, tabuľkami, či dokonca databázami
- umožňuje rotovať v hodnotách
- sa môže v jednej tabuľke nachádzať aj viackrát
- umožňuje získať hodnotu postupnosti aj bez toho, aby sa musel do tabuľky vložiť riadok (napr. vložiť hodnotu postupnosti do premennej)

## Vytvorenie postupnosti

```TSQL
    CREATE SEQUENCE JednaAzPat AS INT START WITH 1 INCREMENT BY 1 MINVALUE 1 MAXVALUE 5 CYCLE;
```

## Získanie hodnoty z postupnosti
1. Získanie jednej hodnoty
Na získanie hodnoty sa používa statement `NEXT VALUE FOR`

```TSQL
    SELECT NEXT VALUE FOR JednaAzPat AS PrvaHodnota; --> 1
    SELECT NEXT VALUE FOR JednaAzPat AS DruhaHodnota; --> 2
```

2. Získanie viacerých hodnôt
Získanie viacerých hodnôt z postupnosti je možné spraviť pomocou procedúry `sp_sequence_get_range`:
V nižšie uvedenom príklade budeme vyberať 15 hodnôt z postupnosti. Keďže postupnosť má iba 5 členov,
parameter `@PocetCyklov` bude obsahovať číslo 3, pretože prišlo k reštartu postupnosti od začiatku 3-krát.
Poznámka: Predpokladáme, že príklad spúšťame po tom, ako sme vytvorili postupnosť a spustili príklad v predošlej časti.

Tieto čísla predstavujú to, čo by sme chceli z postupnosti dostať:

`3` `4` `5` `1` `2` `3` `4` `5` `1` `2` `3` `4` `5` `1` `2`

```TSQL
    DECLARE @PrvaHodnota SQL_VARIANT
            , @PoslednaHodnota SQL_VARIANT
            , @PocetCyklov INT
            , @Inkrement SQL_VARIANT
            , @SeqMinimum SQL_VARIANT
            , @SeqMaximum SQL_VARIANT;
    EXECUTE sp_sequence_get_range
        @sequence_name = N'JednaAzPat'
        , @range_size = 15                                   --> Chceme vratit 15 hodnot z postupnosti
        , @range_first_value = @PrvaHodnota OUTPUT   
        , @range_last_value = @PoslednaHodnota OUTPUT   
        , @range_cycle_count = @PocetCyklov OUTPUT  
        , @sequence_increment = @Inkrement OUTPUT  
        , @sequence_min_value = @SeqMinimum OUTPUT  
        , @sequence_max_value = @SeqMaximum OUTPUT;
    
    PRINT CONCAT('Prva vratena hodnota: ', CAST(@PrvaHodnota AS VARCHAR));           --> 3
    PRINT CONCAT('Posledna vratena hodnota: ', CAST(@PoslednaHodnota AS VARCHAR));   --> 2
    PRINT CONCAT('Pocet cyklov: ', @PocetCyklov);                                    --> 2
    PRINT CONCAT('Inkrement: ', CAST(@Inkrement AS VARCHAR));                        --> 1
    PRINT CONCAT('Minimalna hodnota postupnosti: ', CAST(@SeqMinimum AS VARCHAR));   --> 1
    PRINT CONCAT('Maximalna hodnota postupnosti: ', CAST(@SeqMaximum AS VARCHAR));   --> 5
```