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