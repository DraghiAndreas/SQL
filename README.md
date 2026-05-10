Notite SQL:


LAB 1 (GENERAL)
-----

Notite:

`PRIMARY KEY` : un identifier unic, not null, pt fiecare record dintr-un table (fiecare element dintr un table trb sa aiba alta valoare)

`ed + /` : pt a edita ultima comanda data, si pt a trimite-o

`WHERE` - pt filtrare (SELECT, UPDATE...)

`ORDER BY (ASC/DESC)` - ASC by default

#### Comenzi:
```
CREATE TABLE tab (
	x integer PRIMARY KEY,
	y varchar(10) NOT NULL,
	z number(7,2) NOT NULL,
	d date
);
```
-Creeaza tabelul 'tab'
```
DESCRIBE tab;
```
-Descrie tabelul 'tab'

```
INSERT INTO tab VALUES(1, 'test', 70, '10.05.2018')
```
-Exemplu de insert

```
SELECT * FROM tab;
```
-Vizualizeaza toate record urile din tabel

```
UPDATE tab (numele tabel)
SET z = z-5 (expresia care sa se aplica)
WHERE y = 'test'; (conditia de filtrare)
```

```
SELECT trunct(sysdate - d) as CEVA;
```

```
WHERE y = 'test' OR y = 'test2' (OR, AND)
WHERE y LIKE 'N%' (sa inceapa cu N si sa vina orice dupa) ('%ct', '%ol%','____' - exact 4 caractere)
WHERE d IS (NOT) NULL
```
```
DELETE FROM tab WHERE ... (sau fara WHERE pt tot) (sterge record)
DROP TABLE tab (sterge tabelul)
```

LAB 2 (CONSTRAINTS SI ALTERARE)
-----

#### Notite

- `NOT NULL` - Coloana nu poate avea valori NULL
- `UNIQUE` - Valori unice (pot fi multiple NULL)
- `PRIMARY KEY` - Nu pot fi NULL, si trb sa fie unice
- `CHECK` - Conditie ce trb respectata de toate elementele

---
- `DEFAULT` - Valoarea implicita a coloanei, daca nu se mentioneaza alta

#### Comenzi :

![alt text](image.png)

sau

![alt text](image-1.png)

Alterare:

```
ALTER TABLE tab
    ADD nume_col tip_data;
    DROP COLUMN nume_col;
    MODIFY nume_col nou_tip_data;
    
    ADD CONSTRAINT nume_const def_const;
    DROP
        PRIMARY KEY;
        UNIQUE (nume_col);
```

Copiere tabel (gol):

```
CREATE TABLE tab2 AS
SELECT * FROM tab1
WHERE 1 = 0; (sau orice conditie false)

sau WHERE la row-urile pe care le vrei, gen 

CREATE TABLE Frigidere AS
SELECT * FROM tab1
WHERE Produs = 'Frigider';
```

Insert cu select :

```
INSERT INTO tab2
SELECT * (toate coloanele, sau putem specifica) FROM tab1
WHERE conditie
```

Adaugarea unui constraint de CHECK :

```
ALTER TABLE tab1
ADD CONSTRAINT chk_const1
CHECK (Discount BETWEEN 0 AND 1);
```

Adaugarea unei coloane : 

```
ALTER TABLE tab1
ADD Col varchar(20)
```

Modificarea unei coloane :

```
ALTER TABLE tab1
MODIFIY Old_col VARCHAR(20) <- Tipul nou de data
```

Stergerea unei coloane:

```
ALTER TABLE tab1
DROP COLUMN col_name
```

LAB 3 (GRUPARE)
----

#### Notite :

```
SELECT [DISTINCT] * / coloane [ [AS] alias]
FROM table
    WHERE - conditie de cautare asupra LINIILOR
    GROUP BY - lista de atribute care permit gruparea
    HAVING - conditie de cautare asupra GRUPARILOR
    ORDER BY
```

##### Functii de grup:

- `MIN` - returneaza minimul de pe o coloana
- `MAX` - returneaza maximul de pe o coloana
- `SUM` - returneaza suma de pe o coloana numerica
- `AVG` - returneaza media de pe o coloana numerica
- `COUNT` - returneaza numarul de articole care indeplinesc o anumita conditie

Select in select:

```
SELECT * FROM Evidenta_carti
WHERE Pret = (
    SELECT MAX(PRET) FROM Evidenta_Carti
    );
```

Group by:

```
SELECT Gen, COUNT(*) AS NR FROM Evidenta_carti
WHERE Pret IS NOT NULL
GROUP BY Gen
ORDER BY Gen DESC;
```

```
SELECT Editura, Gen, COUNT(*) as NR
FROM Evidenta_carti
GROUP BY Editura, Gen
ORDER BY Editura
```

```
SELECT Editura, COUNT(*) AS NR
FROM Evidenta_carti
GROUP BY Editura
HAVING COUNT(*) = (
    SELECT MIN(NR)
    FROM (
        SELECT COUNT(*) AS NR
        FROM Evidenta_carti
        GROUP BY Editura
    )
);
```

LAB 4 (FOREIGN KEYS .. REFERENCES)
----


References :
```
CREATE TABLE CONSULTATII (
  CNP CHAR(13) NOT NULL REFERENCES PACIENTI(CNP)   
)
```
- Verifica daca CNP-ul introdus in `CONSULATII` exista si in tabela `PACIENTI` **Daca nu exista, da eroare**
- O linie nu poate fi stearsa din tabela `PACIENTI` daca are 'fii' in tabela `CONSULATII`, de asemenea, daca exista linii in tabela `CONSULTAII`, aceasta trebuie **OBLIGATORIU** stearsa in prealabil, pt a putea sterge tabela `PACIENTI`

On delete cascade:
```
CREATE TABLE PLATI(
    Id_consult INTEGER NOT NULL REFERENCES CONSULTATII(Id_cosnult) ON DELETE CASCADE
)
```

- `ON DELETE CASCADE` indica faptul ca atunci cand o linie din tabela 'Parinte' (`CONSULTATII`) este stearsa, vor fi sterse si liniile dependente din tabela 'copil' (`PLATI`)

```
SELECT A.id_consult, A.CNP, A.plata - nvl(sum(B.Rata),0) as de_plata
FROM consultatii A, plati B
WHERE A.id_consult = B.id_consult (+)
GROUP BY A.id_consult, A.CNP, A.plata
HAVING nvl(sum(B.Rata),0) = 0
```

- `nvl()` returneaza prima valoare non-nula, de exemplu `nvl(NULL, 50)` returneaza `50`
- `(+)` Afiseaza toate liniile din A (`CONSULTATII`) chiar daca nu au match in B (`PLATI`) (`LEFT OUTER  JOIN`)

##### Tipuri de JOIN-uri

- INNER :
    
    Afiseaza doar liniile care au mathc in ambele tabele

- LEFT [OUTER] JOIN :

    Afiseaza TOT din stanga si ce gaseste in dreapta (Daca nu gaseste pune `NULL`)

- RIGHT JOIN :

    Afiseaza tot din dreapta + ce gaseste in stanga

- FULL OUTER JOIN :

    Tot din ambele tabele

- CROSS JOIN :
    
    Fiecare linie cu fiecare linie