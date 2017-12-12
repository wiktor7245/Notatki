# Bazy danych

## Praca z SQL

### SQL Management Studio
#### Logowanie (lokalnie, po 24h usuwane):
Serwer type: Database engine
Server name: localhost
Authentication: Windows Authentication
#### Logowanie (normalna baza danych)
Server name: mssql-2016.labs.wmi.amu.edu.pl
Authentication: SQL Server Authentication
Login: s426272 
Hasło: na pulpicie
#### Nowa baza: New query i w oknie wpisujemy Create Database nazaw i wciskamy F5
    
## Praca z select
### Wyświelenie kolumny (kolumn) z tabeli
  
    Select nazwa1,nazwa2
    From Pracownicy;
    
### Wyświetlenie wszystkich kolumn (atrybutów) z tabeli
  
    Select *
    From Pracownicy;
    
 ### Zmiana nazwy kolumny (tymczasowa)
  
    I sposób:
    
      Select nazwaStata As nazwaNowa
        From Projekty;
        
    II sposób
    
        Select nazwaNowa = nazwaStara
        From Projekty;
        
 ### Wyświelenie aktualnego roku, op. matematycznej oraz przykładowego złączenia (koncentracji stringów)
    
      Select YEAR{GETDATE()) AS 'aktualny rok',
        2+3 As 'suma',
        'to' +'jest' + 'napis' As 'napis';
        
To jest przypadek, gdzie nie trzeba używać FROM (powyżej 
znajdue się T-SQL, dialekcie sql zwanym Transact-SQL)
UPPER(nazwa) - wyswietlenie kolumny z wielkich liter, np. 
    
      selcet UPPER(nazwa) AS nazwa_projeku
      From Projekty;
      
 ### Konwersja typów:
CAST - konwersja typu atrybutu praca z typu money na varchar 
CONVERT - konwersja typu daty na typ znakowy w formacie 103 (dd/mm/yyyy)
    
      Select nazwisko,
        Cast(placa AS varchar(10)) + 'zl' AS placa,
        Convert(varchar, zatrudniony, 103)
      From Pracwonicy;
      
  ### Distinct - usuwanie powtórzeń z wyniku
    
      Select distinct stanowisko
      
  ### Sortowanie wierszy
     Order by Desc/ASC
  ### Top - ograniczenie wyników do pewnej liczny
  
      Select TOP 3 nazwisko, placa
      FROM Pracwonicy Order by placa DESC;
      (TOP powinien iść w parze z order by desc
      
  ### WHERE - filtrowanie wyników
operatory - podobne jak w programowaniu, ale są też dodatkowe
<> - nierówność (to samo co !=)
IS NULL, IS NOT NULL - przyrównanie do nulla
Between .. and ... sprawdzanie, czy dana wartosc wchodzi w dany przedzial
Like - spr, czy wartość jest zgodna ze wzorcem, gdzie, % - dowolna ilość znaków,  _-pojedyńczy znak, []- zbiór znaków, [^] - zbiór niedozwolonych znaków
        
        Przykłady:
        
          Select *
          From Pracownicy
          Where stanowisko = 'profesor';
          
          Select *
          From Pracownicy
          Where (szef = 1 OR szef = 5) AND 
          dod_funkc IS NOT NULL;
          
          SELECT *
          From Pracownicy
          Whre nazwisko Like 'W%' AND 
          placa Between 1000 and 2000;
          
          SELECT *
          From Pracownicy
          Whre nazwisko Like '%WAR%' AND 
          placa Between 1000 and 2000;
          
          SELECT *
          FROM   Projekty
          WHERE  dataRozp > '2005-01-01' AND 
          kierownik IN (4, 5); 
 ### Obsługa wartości pustych
  Zamienianie wartości null na wartość 0 (bądź inną)
  
    SELECT nazwisko, 
      ISNULL(dod_funkc,0) as dodatek
    From Pracownicy;
    
Zwracanie pierwszej niepustej wartosci z listy parametrów

    SELECT COALESCE(nazwisko,nick,'anonim') AS uzytk
    FROM Uzytkownicy;
   
## Złączenia 
  
    Select t1.*,
      t2.*,
      t3.*
    From tabela t1
      Join tabela2 t2
        ON t1.atr1 = t2.atr2
      Join tabela3 t3
        ON t2.atr2 = t3.atr3;
        
### Inner join - łączenie wewnętrzne, łączenie, w którym łączone są te wiersze, które spełniają warunek połączenia po słowie kluczowym ON

Zapytanie zwraca nazwiska pracowników oraz nazwy projektów, którymi kierują. Łączone są wiersze z tabeli Projekty z odpowiednimi wierszami z tabeli Pracownicy (tzn. projekt łączony jest z tym pracownikiem, który jest kierownikiem tego projektu - warunek: Pracownicy.id = Projekty.kierownik, obie kolumny zawierają id pracowników):

    SELECT Pracownicy.nazwisko AS kierownik,
        Projekty.nazwa
    FROM Pracownicy
    INNER JOIN Projekty 
        ON Pracownicy.id = Projekty.kierownik;
   
Lub z użyciem aliasów
  
  SELECT P.nazwisko AS kierownik, 
       R.nazwa
  FROM Pracownicy P

       INNER JOIN Projekty R
               ON P.id = R.kierownik;
### Self-join - łączenie tabeli samej z sobą
  Znajdowanie pary projektów prowadzonych przez tą samą osobę ( 'w' warunku wyświetliłoby powtórzenia)
  
    SELECT P1.nazwa, 
        P2.nazwa
    FROM Projekty P1
       JOIN Projekty P2
         ON P1.kierownik = P2.kierownik
            AND P1.id > P2.id;
            
### Left/right/full outer join - złączenie zewnętrzne. Wynik złączenia zewnętrznego zawiera także wynik inner joina oraz dodatkowo te wiersze, które nie zostały połączone 
Left outer join - złączenie zwenętrzne lewostronne; każdą wiersz z lewej strony uzupełniamy tymi z prawej, które odpowiadają warunkom połączenia. Jeżeli nie ma takiego wiersza, to jego wartość z lewej jest nullem.

Zapytanie zwraca nazwiska wszystkich pracowników oraz nazwę projektu, którym pracownik kieruje (jeśli taki projekt istnieje) - jeśli pracownik nie kieruje żadnym projektem podawana jest wartość NULL. 

    SELECT P.nazwisko AS kierownik,
       R.nazwa
    FROM Pracownicy P
       LEFT OUTER JOIN Projekty R
                    ON P.id = R.kierownik;
Złączenia zewnętrzne są często wykorzystywane do znajdowania wierszy, które nie spełniają warunku połączenia, np: znajdź przedmioty, których student nie zaliczył; znajdź projekt, w którym nikt nie pracuje; znajdź miasto, które nie ma połączenia lotniczego z Warszawą. 

Poniższe zapytanie znajduje stanowiska, na których nie zatrudniono pracowników. 

    SELECT nazwa
    FROM Stanowiska S 
       LEFT OUTER JOIN Pracownicy P
                    ON S.nazwa = P.stanowisko
      WHERE P.id IS NULL;
      
### Cross join - złączenie krzyżowe, iloczyn kartezjański (każdy z każdym)
Zapytanie generuje grafik zajęć na bieżący rok, łącząc każdy kurs z każdym miesiącem w roku:

    SELECT k.nazwa AS nazwa_kursu, 
      d.miesiąc
    FROM Kursy k
      CROSS JOIN Kalendarz d
    WHERE d.rok = YEAR(GETDATE());
    
## Podzapytania
Podzapytania - zagnieżdżanie zapytań - wynik zwrócony przez jedno zapytanie select może zostać wykorzystane w innym zapytaniu. Można je umieszczać w różnych miejscach polecenia select - trzeba jednak uważać na to jakiego rodzaju zbiór może być w danym miejscu użyty (wieloelementowy, jednoelementowy). Podzapytanie umieszcza się w nawiasie okrągłym.

### Podzapytania w WHERE
Najczęściej stosowane są zapytania w where. Takie podzapytanie może zwracać element skalarny lub zbiór wartości.

Zapytanie zwraca informacje o pracownikach pracujących na stanowiskach, na których minimalna płaca jest nie większa od 1500.
Podzapytanie może zwrócić zbiór wartości, dlatego też uzywamy in, a nie =.

    Select *
    From Pracownicy 
    Where stanowisko In
      (Select nazwa
        From stanowiska
        Where placa_min > 1500);

Poniższe zapytanie zwraca informacje o projektach, których stawka jest wyższa niż stawka dla projektu e-learning. Podzapytanie było użyte po to, aby pobrać informacje o stawce dla projektu e-learning. Zagnieżdżenie zwraca pojedyńczą wartość.

    Select *
    From Projekty
    Where stawka > (Select stawka
                              From Projekty
                              Where nazwa = 'e-learning');
                              
### Podzapytania skorelowane (powiązane)
Wynik jest zwracany przez podzapytanie skolerowane jest zależny od aktualnie analizowanego wiersza w zapytaniu nadrzędnym.

Zapytanie zwraca informacje o tych pracownikach, których płaca jest większa od tego, jak przewidują widełki dla tego stanowiska
Podzapytanie wykorzystano do pobrania informacji o maksymalnej płacy dla tego pracownika, na którym aktualnie pracuje pracownik analizowany w nadzapytaniu - warunek s.nazwa = p.stanowisko (podzapytanie będzie zwracało różne wartości, w zależności od stanowiska pracownika)
    
    Select *
    From Pracownicy P
    Where placa > (Select placa_max
                            From Stanowiska S
                            Where S.nazwa = P.stanowisko);
**Uwaga!** Należy unikać podzapytań skorelowanych, ich wykonanie jest nieefektywne. Przeważne można to zrobić inaczej, np.:

    Select *
    From Pracownicy P
      Inner join Stanowiska S
        On P.stanowisko = S.nazwa
    Where P.placa > S.placa_max;
    
## Some, All
Some i All umożliwiają porównanie pewnej wartości ze zbiorem danych (zbiór danych jest często zwracany poprzez podzapytanie):
  * some zwraca prawdę, jeżeli porównanie jest prawdziwe dla jakiejkolwiek wartości z danego zbioru,
  * all zwraca prawdę, jeżeli porównanie jest prawdziwe dla wszystich wartości z tego zbioru.
#### **Uwaga!**
  * Zapis != All jest tożsame z not in
  * Zapis = Some jest tożsame z in 

Wyświetl nazwy stanowisk nie obsadzonych przez żadnego pracownika:
  
    Select nazwa
    From Stanowiska
    Where nazwa != All (Select stanowisko
                                    From Pracownicy);
                                    
## Exists, not exists
Jest operatorem jednoargumentowym - testuje istnienie wartości (sama wartość nie ma znaczenia). Zwraca wartość true, jeżeli argument zwracany jako wartość podzapytania jest niepusty. Jeśli podzapytanie zwraca wartość pustą (Null) wówczas exists zwraca false. 
Exists często idzie w parze z podzapytaniem skorelowanym.

Podaj nazwiska profesorów nie posiadających pod opieką doktorantów:

      Select nazwisko
      From Pracownicy p1
      Where Not Exists (Select *
                                  From Pracownicy p2
                                  Where p2.stanowisko = 'doktorant'
                                    And szef = p1.id)
        And p2.stanowisko = 'profesor';
        
## Funkcje agregujące 
 Funkcje, które działają na zbiorze krotek i zwracają pojedyńczą,zagregowaną wartość. Podstawowe funkcje to:
* Count - zliczanie elementów w zbiorze,
* Avg - średnia wartość elementów w zbiorze,
* Min - wartość minimalna w zbiorze.
* Max - wartość maksymalna w zbiorze,
* Sum - suma wartości elementów w zbiorze.

Zapytanie zwraca liczbę wszystkich pracowników i ich średnie zarobki.
  
    SELECT COUNT(*) 'liczba pracowników', 
                AVG(placa) 'średnia płaca'
    FROM Pracownicy;

**Uwaga!** Niedopuszczalne jest odwoływanie się w jednym zapytaniu do wartości zagregowanych i nie zagregowanych, np.:

      SELECT COUNT(*)   'liczba pracowników', 
       AVG(placa) 'średnia płaca',
       nazwisko
      FROM Pracownicy;
    
Powyższe zapytanie spowoduje wyświetlenie błędu.
  
W funkcji agregującej możemy usunąć duplikaty za pomocą Count (distinct idproj)
  
### Group by
Group by tworzy podział zbioru wierszy na podzbiory, według wartości atrybutów podanych w klauzuli group by.
Funkcje agregujące są następnie wykonywane na każdym z tych podzbiorów.
  
Liczba pracowników i ich średnie zarobki z podziałem na stanowiska:
  
    SELECT   stanowisko, 
         COUNT(*)   'liczba pracowników', 
         AVG(placa) 'średnia płaca'
    FROM     Pracownicy
    GROUP BY stanowisko;

### Having
Po słowie having definiujemy warunek/warunki , które maja być spełnione dla zbioru wierszy (w przeciwieństwie do where, które działa dla poszczególnej krotki).

Liczba pracowników i ich średnie zarobki z podziałem na stanowiska; zwracana jest informacja dotycząca tylko tych stanowisk, na których minimalna wypłacana płaca jest większa niż 2000;

      SELECT   stanowisko, 
         COUNT(*)   'liczba pracowników', 
         AVG(placa) 'średnia płaca'
      FROM     Pracownicy
      GROUP BY stanowisko
      HAVING   MIN(placa) > 2000
      
## Operacje na zbiorach
* Union - suma zbiorów,
* Union all - suma zbiorów wraz z duplikatami,
* Expect - różnica zbiorów,
* Intersect - przekrój (iloczyn) zbiorów
Operacje tego typu mogą być wykonywane tylko na tabelach tego samego typu(o tej samej liczbie kolumn tego samego typu)

Nazwiska praowników zarabiających powyżej 2500 razem z pracownikami zarabiającymi nie więcej niż 2900 (można porównać union z union all):

    SELECT nazwisko, placa
    FROM   Pracownicy
    WHERE  placa > 2500

    UNION -- ALL

    SELECT nazwisko, placa
    FROM   Pracownicy
    WHERE  placa <= 2900;
      
Pracownicy zarabiający nie więcej niż 2900, po usunięciu tych, co zarabiają powyżej 2500:

    SELECT nazwisko, placa
    FROM   Pracownicy
    WHERE  placa <= 2900

    EXCEPT

    SELECT nazwisko, placa
    FROM   Pracownicy
    WHERE  placa > 2500;
    
Pracownicy zarabiający więcej niż 2500 i nie więcej niż 2900:

    SELECT nazwisko, placa
    FROM   Pracownicy
    WHERE  placa > 2500

    INTERSECT

    SELECT nazwisko, placa
    FROM   Pracownicy
    WHERE  placa <= 2900;


## Podzapytania - część II
### From
Ponieważ wynikiem select jest relacja (tabela), to można ją umieścić w klauzuli from.
**Uwaga!** Składnia t-sql wymaga nazwania takiego podzapytania przy użyciu As oraz nazwania każdej z kolumn.

Średnia liczba pracowników na stanowiskach:

    SELECT AVG(liczba)
    FROM   (SELECT   COUNT(*) AS liczba
        FROM     Pracownicy
        GROUP BY stanowisko) AS Tabela;
        
### Select 
Stosunek liczby pracowników na oszczególnych stanowiskach do liczby wszystkich pracowników 

    SELECT   stanowisko, 
         CAST(COUNT(*) AS REAL)/(SELECT COUNT(*) FROM Pracownicy) AS 'udzial'
    FROM     Pracownicy
    GROUP BY stanowisko;
:heavy_exclamation_mark:**Uwaga!** Funkcje analityczne pozwalają na wykonanie powyższego zapytania w wydajniejszy sposób.
## Widoki
Widok (perspektywa, ang (view)) jest nazwanym, zapisanym po stronie serwera zapytaniem select, którego wynik może być wielokronie odczytywany.

:heavy_exclamation_mark: Widok nie zawiera kopii danych

:information_source:  Zasięg widoku - obiekt zapisany na stałe w bazie danych.

Główne cechy widoku:
* pozwala na wygodne prezentowanie użytkownikowi danych jednej lub wielu tabel oraz danych wyliczanych,
* ogranicza dostęp do poufnych danych, ukrywa strukturę bazy danych, 
* ułatwia zarządzanie uprawnieniami użytkowników,
* widok nie może dotyczyć tabel tymczasowych,
* widoki nie są usuwane razem z tabelą - trzeba je usunąć oddzielnie,
* można dla nich tworzyć tzw, wyzwalacze (ang. triggers, procedury wykonywane automatycznie jako reakcja na pewne zdarzenia w tabeli bazy danych),
* widoki pamiętają ograniczenia obowiązujące w tabelach bazowych,
* możliwe jest (z pewnymi ograniczeniami) modyfikowanie danych w tabelach bazowych poprzez widok,
* definicja widoku nie może zawierać order by (chyba,że zawiera top), compute i into.

Polecenia tworzące, zmieniające i usuwające widok:
* Creare view,
* Alter view,
* Drop view.

Wyświetlanie informacji o istniejących widokach:

    Select *
    From INFORMATION_SCHEMA.VIEW_TABLE_USAGE;
    
Wyświetlanie definicji widoków:
  
    SELECT TABLE_NAME,
                VIEW_DEFINITION,
    FROM INFORMATION.SCHEMA.VIEWS;
    
Wyświetlanie listy obiektów powiązanych z danym widokiem:
    
    EXEC sp_demands nazwa_widoku;

#### Przykład 1
Utworzenie widoku wyświetlającego informacje o adiunktach (zwróć uwagę gdzie są definiowane nazwy zwracanych  kolumn):

    CREATE VIEW Adiunkci(nazwisko, staz, zarobki)
    AS
    (
      SELECT nazwisko,
                  DATEDIFF(YEAR, zatrudniony, GETDATE()),
                  placa + ISNULL(dod_funkc, 0)
      FROM Pracownicy
      WHERE stanowisko = 'adiunkt'
    );
    GO
    
  :information_source: Polecenie CREATE VIEW musi byc jedyną instrukcją w serii zapytań (ang. branch), zatem zaraz po nim najbezpieczniej jest użyć polecenia GO (które nota bene nie należy do t-sql).
  
  Powyższy widokmożna odczytać za pomocą:
  
      Select *
      From Adiunkci;
      
  Modyfikacja danych (nazwiska) w widoku (:heavy_exclamation_mark:modifikowana jest źródłowa tabela Pracownicy):
      
      UPDATE Adiunkci
      SET nazwisko = 'Fiołkowska-Bąk'
      WHERE nazwisko = 'Fiołkowska';
      
:heavy_exclamation_mark:Tutaj nie jest możliwa modyfikacja danych w widoku, ponieważ kolumny są wyliczane w widoku:

      UPDATE Adiunkci
      SET  zarobki = 1.1 * zarobki;
      
:heavy_exclamation_mark:Tutaj nie jest możliwe dodanie danych w widoku, ponieważ kolumny są wyliczane w widoku:

    INSERT INTO Adiunkci VALUES('Bogucki, 1, 2000);
    
:information_source: Po modyfikacji tabeli Pracownicy przywracamy jej pierwotny stan poprzez zmianę w widoku:

    UPDATE Adiunkci
    SET nazwisko = 'Fiokłkowska'
    WHERE nazwisko = 'Fiołkowska-Bąk';
    
## Tabele tymczasowe

Tabele tymczasowe (lokalne) są tworzone w bazie tempdb. Taka tabela jest obliczana tylko raz i jest usuwana automatycznie po zakończeniu sesji. Nazwa tabeli poprzedzona jest #.

:information_source: Zasięg tabeli - bieżąca sesja (usuwana po rozłączeniu).

#### Przykład 2
Utworzenie tabeli tymczasowej z informacjami o adiunktach:

    CREATE TABLE #Adiunkci(
        nazwisko VARCHAR(50),
        staz INT,
        zarobki MONEY
    );
    
    INSERT INTO #Adiunkci
      SELECT nazwisko,
                  DATEDIFF(YEAR, zatrudniony, GETDATE()),
                  placa + ISNULL(dod_funkc, 0)
      FROM Pracownicy
      WHERE stanowisko = 'adiunkt';
      
lub w inny sposób:
  
    SELECT nazwisko,
                DATEDIFF(YEAR, zatrudniony, GETDATE()) staz,
                placa + ISNULL(dod_funkc, 0)
    INTO #Adiunkci
    FROM Pracownicy
    WHERE stanowisko = 'adiunkt';
    
Odczyt danych z tabeli tymczasowej:

    Select *
    From #Adiunkci;
    
Modyfikacja tabeli tymczasowej:

    UPDATE #Adiunkcki
    SET nazwisko = 'Fiołkowska-Bąk'
    WHERE nazwisko = 'Fiołkowska';
    
W porównaniu z widokiem, tutaj jest możliwość modyfikacji zarobków i powoduje zmianę w tabeli tymczasowej:

    UPDATE #Adiunkci
    SET zarobki = 1.1 * zarobki;
    
W porównaniu z widokiem, tutaj możliwe jest wstawienie do wiersza tabeli tymczasowej:

    INSERT INTO #Adiunkci VALUES('Bogucki, 1, 2000);

## Zmienne tablicowe

Zmienna tablicowa to zmienna, która może przechowywać tablicę

:information_source: Zasięg - tylko w ramach skryptu.

#### Przykład 3

Tworzenie zmiennej tablicowej z informacjami o adiunktach i odczyt tej zmiennej (:heavy_exclamation_mark: uwaga, jeden skrypt, wykonyawny w całości):

    DECLARE @Adiunkci TABLE(
        nazwisko = VARCHAR(50),
        staz     INT,
        zarobki  MONEY
    );
  
    INSERT INTO @Adiunkci
    SELECT nazwisko, 
           DATEDIFF(YEAR, zatrudniony, GETDATE()), 
           placa + ISNULL(dod_funkc, 0)
    FROM   Pracownicy 
    WHERE  stanowisko = 'adiunkt';

    SELECT * 
    FROM   @Adiunkci;
    
Modyfikacja danych w ww. zmiennej tablicowej:

    DECLARE @Adiunkci TABLE(
    nazwisko VARCHAR(50),
    staz     INT,
    zarobki  MONEY
    );

    INSERT INTO @Adiunkci
        SELECT nazwisko, 
              DATEDIFF(YEAR, zatrudniony, GETDATE()), 
              placa + ISNULL(dod_funkc, 0)
        FROM   Pracownicy
        WHERE  stanowisko = 'adiunkt';

    SELECT * 
    FROM   @Adiunkci;

    UPDATE @Adiunkci
    SET    zarobki = 1.1 * zarobki;

    SELECT * 
    FROM   @Adiunkci;
    
## Wspólne wyrażenia tablicowe

Wspólne wyrażenia tablicowe (ang. Common Table Expression, CTE) to wyrażenia ze słowem WITH, po których podajemy nazwę wynikowego zbioru oraz jego definicję za pomocą polecenia SELECT. Do tak zdefiniowanego zbioru możemy odwoływać się tak jak do zwykłej tabeli.

:information_source: Zasięg CTE - zapytanie SELECT

#### Przykład 4

Tworzenie i odczyt CTE o adiunktach:

    WITH CTE_Adiunkci(nazwisko, staz, zarobki)
    As
    (
        SELECT nazwisko,
                  DATEDIFF(YEAR, zatrudniony, GETDATE()),
                  placa + ISNULL(dod_funkc, 0)
        FROM Pracownicy
        WHERE stanowisko = 'adiunkt'
    )
    Select *
    FROM CTE_Adiunkci;
    
W poniższym zapytaniu modyfikacja jest możliwa i w rezultacie modyfikowana jest źródłowa tabela Pracownicy:

    WITH CTE_Adiunkci(nazwisko, staz, zarobki)
    AS
    (
        SELECT nazwisko,
                    DATEDIFF(YEAR, zatrudniony, GETDATE())
                    placa + ISNULL(dod_funkc, 0)
        FROM Pracownicy
        WHERE stanowisko = 'adiunkt'
    )
    UPDATE CTE_Adiunkci
    SET nazwisko = 'Fiołkowka-Bąk'
    WHERE nazwisko = 'Fiołkowska';
    
:heavy_exclamation_mark: W poniższym zapytaniu **nie jest możliwa** modyfikacja danych, ponieważ kolumna zarobki jest wyliczana:

    WITH CTE_Adiunkci(nazwisko, staz, zarobki)
    AS
    (
        SELECT nazwisko,
                    DATEDIFF(YEAR, zatrudniony, GETDATE())
                    placa + ISNULL(dod_funkc, 0)
        FROM Pracownicy
        WHERE stanowisko = 'adiunkt'
    )
    UPDATE CTE_Adiunkci
    SET zarobki = 1.1 * zarobki;

:information_source: Po modyfikacji tabeli Pracownicy przywracamy jej pierwotny stan:

    UPDATE Pracownicy
    SET nazwisko = 'Fiołkowska'
    WHERE nazwisko = 'Fiołkowska-Bąk';
    
### Wykorzystanie do budowania zapytań rekurencyjnych

Schemat zapytania rekurencyjnego:

    WITH CTE_nazwa(nazwa_kolumny [,.. n]
    AS
    (
        definicja_zapytania_CTE --definicja tzw. zapytania zakotwiczonego
        --(zbiór elementów stanowiących korzeń lub korzenie)
        
        UNION ALL
        
        definicja_zapytania_CTE --definicja zapytania rekursywnego, odwołującego się do CTE_nazwa 
    )
    --zapytanie korzystające z CTE
    SELECT *
    FROM CTE_nazwa;
    
#### Przykład 5

Zapytanie wyświetla całą hierarchię szef-podwładny w firmie:

    WITH CTE_Podwładni(nazwisko, id, poziom)
    AS
    (
        SELECT nazwisko,         --krok zerowy, zaczynamy od war pocz.
                    id,                                 --w tym przypadku jest to pracownik ze szczytu hierarchii
                    0
        FROM Pracownicy
        WHERE szef IS NULL
        
        UNION ALL             --sumujemy zbiory wynikowe kolejnych kroków rekurencji
        
        SELECT P.nazwisko,   --krok n-ty rekurencji, odwołujemy się do CTE_Podwładni z poprzedniego kroku!!!
                    P.id,
                    T.poziom +1
        FROM Pracownicy P
                    JOIN CTE_Podwładni T
                        ON P.szef = T.id
    )
    SELECT nazwisko,
                poziom
    FROM CTE_Podwładni;
    
#### Przykład 6

Liczenie silni:

    WITH CTE_Silnia(n, biezacy_iloczyn)
    AS
    (
        SELECT 1,                         -- element początkowy
                    1
           
      UNION ALL                         -- laczymy zbiory wynikowe kolejnych kroków rekurencji
    
      SELECT n + 1, 
                    (n + 1) * biezacy_iloczyn  -- rekursja
        FROM   CTE_Silnia
        WHERE  n < 10                     -- terminator
    )
    SELECT n, 
        biezacy_iloczyn
    FROM   CTE_Silnia;
    
## Polecenia DDL (Data Definition Language)

### Tworzenie i usuwanie bazy danych

Instrukcja tworząca bazę danych:

    CREATE DATABASE Nazwa_Bazy_Danych;
    
Usunięcie bazy danych:
  
    DROP DATABASE Nazwa_Bazy_Danych;
    
### Tworzenie i usuwanie schematu:

    CREATE SCHEMA nowy_schemat AUTHORIZATION dbo;
    
### Tworzenie tabel

Skrócona składnia polecenia tworzącego tabelę:

      CREATE TABLE nazwa_tabeli
      (
          nazwa_atrybutu typ [ograniczenie],
          nazwa_atrybutu typ [ograniczenie],
          [ograniczenie],
          ...
      );
      
Typy danych:
* numeryczne(dokładne): BIGINT, NUMERIC, BIT, SMALLINT, DECIMAL, SMALLMONEY, INT, TINYINT, MONEY;
* numeryczne(przybliżone): FLOAT, REAL;
* daty i czasu: DATE, DATETIMEOFFSET, DATETIME2, SMALLDATETIME, DATETIME, TIME;
* znakowe: CHAR, VARCHAR, TEXT;
* binarne: BINARY, VARBINARY, IMAGE;
* inne: CURSOR, TIMESTAMP,, HIERARCHYID, UNIQUEIDENTIFIER, SQL_VARIANT, XML, TABLE

Ograniczenia:
* PRIMARY KEY - ograniczenie definiujące klucz podstawowy dla relacji;
* [FOREIGN KEY] REFERENCES - ograniczenie definiujące, że dany atrybut jest kluczem obcym w relacji;
* NULL | NOT NULL - ograniczenie weryfikujące, czy wartość może/ nie może być pusta;
* CHECK - ograniczenie zawęża dziedzinę atrybutu;
* UNIQUE - wymusza unikalne wartości dla atrybutu;

DEFAULT - nadawanie atrybutowi wartości domyślnej

IDENTITY - tworzenie automatycznego identyfikatora (w MSSQL)
* Często definiując tabele chcemy, aby identyfikator krotek był automatycznie tworzony przez silnik bazy danych. W ten sposób wstawiając dane do tabeli nie musimy sprawdzać, czy wartość identyfikatora krotki, którą chcemy dodać jest unikalna. W przypadku MSSQL aby uzyskać automatyczne tworzenie identyfikatora tabeli wystarczy, że w definicji tabeli przy polu, które chcemy aby było kluczem głównym dodamy ograniczenie IDENTITY(START, KROK), gdzie parametr START określa od jakiej wartości chcemy rozpocząć numerowanie, KROK określa o jaką wartość zwiększane będą wartości atrybutu w kolejnych krotkach.

#### Przykład 1

    CREATE TABLE Nauczyciele
    (
      id       INT NOT NULL PRIMARY KEY,
      nazwisko VARCHAR(30) CHECK (nazwisko LIKE '[A-Z]%'),
      dyzur    VARCHAR(30) CHECK (dyzur IN ('pon', 'wt', 'sr', 'czw', 'pt')),
      zarobek  FLOAT,
      CHECK (zarobek >= 1000)
    );

    CREATE TABLE Przedmioty
    (
      id_przed    INT IDENTITY(1,1) PRIMARY KEY,
      nazwa       VARCHAR(50),
      prowadzacy  INT REFERENCES Nauczyciele(id),
      rodzaj      VARCHAR(7) CHECK (rodzaj IN ('obowiazkowy', 'obieralny')) DEFAULT 'obowiazkowy',
      rok_studiow INT
    );
    
Ograniczeniom można (a nawet należy) nadawać włąsne nazwy:

    CREATE TABLE Nauczyciele
    (
      id       INT NOT NULL CONSTRAINT pk_naucz_id PRIMARY KEY,
      nazwisko VARCHAR(30) CONSTRAINT ck_naucz_nazw CHECK (nazwisko LIKE '[A-Z]%'),
      dyzur    VARCHAR(30) CHECK (dyzur IN ('pon', 'wt', 'sr', 'czw', 'pt')),
      zarobek  FLOAT,
      CONSTRAINT ck_naucz_zarob CHECK (zarobek >= 1000)
    );
    
### Modyfikacja tabeli

Istniejącą tabelę możemy zmodyfikować, tzn.:
* dodać, usunąć lub zmodyfikować atrybuty,
* dodać lub usunąć ograniczenia.

Skrócona składnia polecenia:

    ALTER TABLE nazwa_tabeli <operacja> [, <operacja> ...];
    
gdzie:

    <operacja> = 
        ADD <definicja atrybutu> - dodanie nowego atrybutu (kolumny)
        ADD CONSTRAINT <ograniczenie> - dodanie ograniczenia
        ALTER [COLUMN] <definicja atrybutu> - modyfikacja atrybutu
        DROP COLUMN nazwa_atrybutu - usunięcie atrybutu
        DROP CONSTRAINTnazwa_ograniczenia - usunięcie ograniczenia
        
#### Przykład 2

Dodanie atrybutu data_urodz do tabeli Nauczyciele:

    ALTER TABLE Nauczyciele
    ADD data_urodz SMALLDATETIME;
    
Dodanie ograniczenia na atrybucie rok_studiów:

    ALTER TABLE Nauczyciele
    ADD CONSTRAINT ck_stu_rok CHECK (rok_studiow IN (1, 2, 3, 4, 5));
    
Usunięcie z tabeli Nauczyciele ograniczenia CHECK na nazwisku:

    ALTER TABLE Nauczyciele
    DROP CONSTRINT ck_naucz_nazw;
    
### Usuwanie tabel

Aby usunąć tabele należy zastosować polecenie:

    DROP TABLE Nazwa_tabeli;
    
lub lepiej:
    
    IF OBJECT_ID('Nazwa_tabeli', 'U') IS NOT NULL 
    DROP TABLE Nazwa_tabeli'
    
Polecenie usuwa tabelę, wszystkie ograniczenia, które zostały dla niej utworzone i wszystkie dane w niej przechowywane.

#### Przykład 3

    DROP TABLE Nauczyciele;
    
:heavy_exclamation_mark: Należy pamiętać, że nie możemy usunąć tabeli, do której isnieją powiązania (klucze obce) bez uprzedniego usunięcia powiązań lub całych tabel, które zawierają odwołania do usuwanej tabeli.

## Polecenia DML (Data Manipulation Language)

Na DML składają się cztery podstawowe komendy:

* SELECT - wyświetla wiersze z wybranych tabel spełniające podane warunki,
* INSERT - wstawia nowy wiersz do tabeli,
* UPDATE - modyfikuje wartość instniejącego wiersza w tabeli,
* DELETE/TRUNCATE - usuwa wiersze z tabeli.

Polecenia te implementują w bazie danych CRUD'a.

Ponadto, mamy również polecenie MERGE, które jest odpowiedzialne za scalanie.

### SELECT ... INTO

Umożliwia umieszczenie wyniku polecenia SELECT w nowo utworzonej tabeli:

    SELECT atr1, atr2, ...
    INTO Nowa_tabela
    FROM Tabela
    WHERE ...;

### INSERT INTO

Umożliwia wstawienie jednego lub wielu wierszy do tabeli. Uproszczona składnia:

* Wstawienie pojedyńczego wiersza:

    INSERT INTO Nazwa_tabeli [(atrybut, atrybut, ...)]
    VALUES (wartosc, wartosc, ...);

* Wstawienie wielu wierszy:

    INSERT INTO Nazwa_tabeli [(atrybut, atrybut, ...)]
    VALUES
        (wartosc, wartosc, ...),
        (wartosc, wartosc, ...),
        ...
        (wartosc, wartosc, ...);

* Wstawienie wierszy pobranych z tabeli:

    INSERT INTO Nazwa_tabeli
        SELECT ...

#### Przykład 1

Wstawienie pojedyńczego wiersza:

    INSERT INTO Nauczyciele(nazwisko, stanowisko, pensja) 
    VALUES ('Kościuszko', 'stażysta', 1200);

#### Przykład 2  

Listę atrybutów można pominąć, jeżeli wstawiane wartośći podane są w kolejnośći zgodnej z kolejnością arybutów w tabeli:

    INSERT INTO Nauczyciele
    VALUES ('Kosciuszko','stazysta',1200);

#### Przykład 3

Można wstawić do tabeli również dane pobrane z innej tabeli za pomocą SELECT:

    INSERT INTO Nauczyciele2006(nazwisko)
        SELECT nazwisko
        From nauczyciele
        WHERE rok_zatrudnienia = 2006;

### UPDATE

Umożliwia modyfikację danych w tabeli. Uproszczona składnia:

    UPDATE Nazwa_tabeli
    SET atrybut_1 = wartosc_1
        ...
        atrybut_n = wartosc_n
    [WHERE warunki];

Modyfikowanie tabeli na podstawie danych z innej tabeli:

    UPDATE
        Tab_A
    SET
        Tab_A.col1 = Tab_B.col1,
        Tab_A.col2 = Tab_B.col2,
    FROM
        TabelaGlowna AS Tab_A
        INNER JOIN TabelaDruga AS Tab_B
            ON Tab_A.id = Tab_B.id
    WHERE Tab_A.col12 = 'value';

:heavy_exclamation_mark: Jeżeli nie zastosujemy WHERE, zmianie ulegną wszystkie wiersze w relacji.

#### Przykład 4

Poniższy przykład zwiększa wszystkim nauczycielom dyplomowanym pensję o 200 zł.

    UPDATE nauczyciele
    SET pensja = pensja + 200
    WHERE stanowisko = 'dyplomowany';

### DELETE

Usuwa wybrane wiersze z tabeli:

    DELETE FROM Tabela
    [WHERE warunki];

#### Przykład 5

Usuwamy wszystkich nauczycieli o nazwisku 'Nowak';

    DELETE FROM Nauczyciele
    WHERE nazwisko = 'Nowak';

:heavy_exclamation_mark: Należy zawsze stosować WHERE, wskazujący na wiersze, które mają zostać usunięte, najlepiej poprzez wskazanie ich kluczy podstawowych.

### ON DELETE

Domyślnie niemożliwe jest usunięcie krotek, do których odwołują się krotki innych tabeli poprzez klucz obcy. Można to zmienić - jeżeli przy klauzuli definiującej klucz obcy umieszczono dyrektywę ON DELETE CASCADE, wówczas usunięcie wierszy spowoduje kaskadowe usunięcie wszystkich powiązanych z nimi wierszy. Aby podczas usuwania krotki w powiązanych tabelach zamienić wartość klucza obcego na wartość nieokreśloną, należy posłużyć się dyrektywą ON DELETE SET NULL. Inna możliwość to ustawienie wartości domyślnej - ON DELETE SET DEFAULT.

#### Przykład 6

Usunięcie nauczyciela spowoduje ustawienie wartości pustej w kolumnie prowadzący dla wszystkich powiązanych z nim przedmiotów:

   CREATE TABLE Przedmioty
        (
            id_przed    INT IDENTITY(1,1) PRIMARY KEY,
            nazwa       VARCHAR(50),
            prowadzacy  INT REFERENCES Nauczyciele(id) ON DELETE SET NULL,
            rodzaj      VARCHAR(7) CHECK (rodzaj IN ('obowiazkowy', 'obieralny')) DEFAULT 'obowiazkowy',
            rok_studiow INT
        ); 

### TRUNCATE

Polecenie to działa podobnie jak DELETE bez klauzuli WHERE, z tym nie zapisuje logów dotyczących usunięcia konkrenych rzeczy. Polecenie to korzysta ze znacznie mniejszych zasobów systemowych oraz logów tranzakcyjnych. Z najważniejszych ograniczeń należy wspomnieć o braku możliwości wyczyszczenia tabeli tym poleceniem, jeżeli dana tabela ma odniesienia typu FOREIGN KEY z innych tabel.

### MERGE

Polecenie to jest połączeniem INSERT, UPDATE I DELETE - powoduje wstawienie, zmodyfikowanie lub usunięcie wierszy w jedenej tabeli na podstawie złączenia z tabelą źródłową. W ten sposób można synchronizować tabele.

#### Przykład 4

Poniższe polecenie scala tabele Towary i Towary_AGD do tabeli Towary - istniejące wiersze (dokładnie - cena) w tabeli Towary zostają zaktualizowane na podstawie danych w tabeli Towary_AGD, a brakujące zostają wstawione.

    MERGE Towary
    USING Towary_AGD
    ON (Towary.id_towaru = Towary_AGD.id)
    WHEN MATCHED THEN
        UPDATE SET Towary.cena = Towary_AGD.cena
    WHEN NOT MATCHED THEN
        INSERT (id_towaru, cena, nazwa, kategoria)
        VALUES (Towary_AGD.id, Towary_AGD.cena,
        Towary_AGD.nazwa, 'AGD')
    OUTPUT deleted.*,inserted.*;

## Projektowanie baz - model ER

    http://www.staff.amu.edu.pl/~mwisla/BAD210/W03%20Projektowanie%20baz.pdf

## Skrypty i procedury

### Zmienne

**Zmienna lokalna**

Nazwa zmiennej lokalnej zaczyna się od znaku @.

    DECLARE @x INT; --deklaracja
    SET @x = 4;     --przypisanie wartosci

    SELECT *
    FROM PRACOWNICY
    WHERE id_prac = @x; --uzycie zmiennej

    GO

#### Przykład 1

Do wyświetlania wartości zmiennej można użyć funkcji PRINT

Podstawienie wyniku SELECT pod zmienną

    DECLARE @max MONEY;
    SET @max = (SELECT MAX(zarobki)
                FROM Pracownicy);
    
    PRINT @max;

    GO

lub

    DECLARE @max MONEY;
    SELECT @max = MAX(zarobki)
                FROM Pracownicy;

    PRINT @max;

    GO

**Zmienna typu tablicowego**

#### Przykład 2

    DECLARE @tab TABLE
    (
        nazw VARCHAR(20),
        stanow VARCHAR(20)
    );

    INSERT INTO @tab
        SELECT nazwisko,
                stanowisko
        FROM Pracownicy;
    
    SELECT *
    FROM @tab;
    
    GO

**Zapytanie tworzone dynamicznie**

#### Przykład 3

    DECLARE @nazwa VARCHAR(20);
    SET @nazwa = 'Pracownicy';
    EXEC ('SELECT * FROM' + @nazwa);

**Zmienna systemowa**

Zaczynają się od @@

#### Przykład 4

Zmienna systemowa @@ROWCOUNT przechowuje liczbę wierszy ostatniego polecenia SQL.

    SELECT *
    FROM Pracownicy;

    SELECT @@ROWCOUNT AS 'liczba wierszy ostatniego polecenia SQL';

#### Przykład 5

Zmienna systemowa @@ERROR przechowuje numer ostatniego błędu.

    INSERT INTO Pracownicy VALUES ('zla wartosc');
    SELECT @@ERROR AS 'numer bledu';

### Instrukcje warunkowe

Do warunkowego wykonywania kodu SQL możemy użyć dwóch konstrukcji:

* IF...ELSE,
* CASE.

#### Przykład 6

    SELECT nazwisko,
            zarobki,
            premia = CASE
                        WHEN premia IS NOT NULL
                        THEN premia
                        ELSE 0
                    END
    FROM Pracownicy;

### Instrukcje sterujące

Do wykonywania pętli można też użyć poleceń WHILE, BREAK, CONTINUE oraz bloku BEGIN...END.

#### Przykład 7

Poniższy skrypt dokonuje wielkokronej podwyżki zarobków pracowników o 10% tak, aby średnie podstawowe zarobki wynosiły przynajmniej 3000 zł, przy czym nie może zostać przekroczony budżet na pensje o wysokości 35000zl.

    SELECT *
    INTO   Pracownicy_kopia
    FROM   Pracownicy;


    WHILE (SELECT AVG(placa)
      FROM   Pracownicy_kopia) < 3000
    BEGIN
        UPDATE Pracownicy_kopia
        SET    placa = placa * 1.1;

        IF (SELECT SUM(placa + ISNULL(dod_funkc, 0))
                FROM   Pracownicy_kopia) > 35000
            BEGIN
                PRINT 'Nie stac nas na dalsze podwyzki!';
                BREAK;
            END;
        ELSE
            CONTINUE;
    END;

    PRINT 'Podwyzki zakonczone.';


## Procedury składowane

Procedury składowane są to prekompilowane wyrażenia SQL, które są przechowywane na serwerze. Procedury:

* nie mogą zawierać poleceń CREATE, TRIGGER, PROCEDURE, DEFAULT ORAZ RULE,
* pozwalają na definiowanie logiki bazy danych (logika aplikacji, logika przetwarzania),
* dostarczają mechanizmy bezpieczeństwa,
* mogą być uruchamiane automatycznie przy starcie systemu (procedura musi być umieszczona w bazie master, sp_procoption = TRUE),
* mogą być zagnieżdżone (@@NESTLEVEL przechowuje poziom zagnieżdżenia),
* mogą poprawić wydajność (prekompilacja),
* mogą zredukować obciążenia sieci,
* mogą mieć parametry wejściowe i wyjściowe (lokalne zmienne procedury).

### Tworzenie

    CREATE PROC [ EDURE ] procedure_name [ ; number ] 
        [ { @parameter data_type } 
            [ VARYING ] [ = default ] [ OUTPUT ] 
        ] [ ,...n ] 
    [ WITH 
        { RECOMPILE | ENCRYPTION | RECOMPILE , ENCRYPTION } ] 
    [ FOR REPLICATION ] 
    AS sql_statement [ ...n ]

### Modyfikacja i usuwanie 

    ALTER PROC nazwa_procedury tresc_procedury;
    DROP PROC nazwa_procedury;

### Wywołanie

    [ [ EXEC [ UTE ] ] 
        { 
            [ @return_status = ] 
                { procedure_name [ ;number ] | @procedure_name_var 
        } 
        [ [ @parameter = ] { value | @variable [ OUTPUT ] | [ DEFAULT ] ] 
        [ ,...n ] 
    [ WITH RECOMPILE ] 

### Obsługa błędów

Do obsługi błędów możemy użyć szeregu funkcji, w zależności od naszych potrzeb.

Funkcja RAISERROR() generuje komunikat o błędzie i rozpoczyna przetwarzanie błędu w sesji, np.:

    RAISERROR('Komunikat o bledzie',16,1);

Aby przechwytywać błędy możemy użyć konstrukcji TRY...CATCH:

    BEGIN TRY  
        { sql_statement | statement_block }  
    END TRY  
    BEGIN CATCH  
        [ { sql_statement | statement_block } ]  
    END CATCH  
    [ ; ]

Poniższe funkcje możemy użyć w konstrukcji TRY...CATCH:

* ERROR_NUMBER() zwraca numer błędu,
* ERROR_SEVERITY() zwraca istotność błędu,
* ERROR_STATE() zwraca numer stanu błędu,
* ERROR_PROCEDURE() zwraca nazwę procedury składowanej lub wyzwalacza, w którym wystąpił błąd,
* ERROR_LINE() zwraca numer linii wewnątrz procedury, gdzie wystąpił błąd,
* ERROR_MESSAGE() zwraca kompletny tekst komunikatu o błędzie; tekst zawiera podane wartości dla dowolnych parametrów zastępowalnych, takich jak długości, nazwy obiektów lub czasy.

#### Przykład 8

Procedura kierownik zwraca jako parametr wyjściowy nazwisko kierownika projektu, którego numer jest podany jako parametr wejściowy. Procedura wyrzuca błąd w przypadku, jeśli projekt o danym numerze nie istnieje:

    CREATE PROCEDURE kierownik @nr_proj  INT,
                           @nazwisko VARCHAR(20) OUTPUT
    AS
        BEGIN TRY
            IF EXISTS (SELECT *
                   FROM   Projekty
                   WHERE  id = @nr_proj)
                BEGIN
                    SELECT @nazwisko = nazwisko
                    FROM   Pracownicy
                    WHERE  id = (SELECT kierownik
                             FROM   Projekty
                             WHERE  id = @nr_proj);
                END;
            ELSE
                RAISERROR('Nie ma takiego projektu', 11, 1);
        END TRY
        BEGIN CATCH
            SELECT ERROR_NUMBER() AS 'NUMER BLEDU',
               ERROR_MESSAGE() AS 'KOMUNIKAT';
        END CATCH;

Wywołanie bez obsługi błędów:

    DECLARE @nazw VARCHAR(20);
    EXEC kierownik,
            10,
            @nazw OUTPUT;
    PRINT @nazw;

Wywołanie z obsługą błędów:

    DECLARE @nazw VARCHAR(20);
    EXEC kierownik,
        100,
        @nazw OUTPUT;
    PRINT @nazw;

#### Przykład 9

Procedura sprawdz wywłuje procedure wstaw i przechwytuje ewentualny błąd zwrócony przez tę procedurę:

    CREATE TABLE tmp(nr INT); --tabela pomocnicza


    CREATE PROCEDURE wstaw @nr INT
    AS
        IF NOT EXISTS (SELECT *
                        FROM tmp
                        WHERE nr = @nr)
            INSERT INTO tmp
            VALUES (@nr);
        ELSE 
            RAISERROR('juz jest taki numer', 11 ,1);

    
    CREATE PROCEDURE sprawdz @nr INT
    AS
        BEGIN TRY
            EXEC wstaw
                @nr;
            PRINT 'wstawilem wierszy: ' +
            CAST(@@ROWCOUNT AS VARCHAR(10));
        END TRY;
        BEGIN CATCH
            PRINT 'nie wstawilem wierszy, bo: ' + CAST(ERROR_NUMBER() AS VARCHAR(20)) + ' ' + ERROR_MESSAGE();
        END CATCH;

    PRINT 'i dzialam dalej';

Wywołanie:

    sprawdz 1;

RAISERROR() nie powoduje przerwania działania procedury. Aby przerwać procedurę należy użyć np. RETURN.

### Odczytanie definicji procedury

Aby uzyskać treść polecenia definiującego daną procedurę,należy odwołać się widoków systemowych sys.sysobjects oraz sys.syscomments, przy czym interesują nas wiersze o wartościach kolumny xtype z widoku sys.sysobjects równych P.

#### Przykład 10

Wyświetlenie zawartości procedury kierownik:

    SELECT c.text
        FROM   sys.sysobjects o 
        JOIN sys.syscomments c
            ON c.id = o.id 
    WHERE  xtype = 'P'
        AND name = 'kierownik';