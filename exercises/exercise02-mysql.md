# Tietokannat 2 - Oppimistehtävä 2 - Sekalaisia MySQL-tehtäviä

Jarmo Syvälahti

2019

## 1. 

Tutki ja kirjoita ylös, mitä oheinen kysely tekee (siis ihan selvällä suomen kielellä että millaisiin tarpeisiin moinen rakennelma antaisi vastauksen).

```sql
SELECT luku FROM (
SELECT luku FROM a
UNION ALL
SELECT luku FROM b
) a2
WHERE 3 > (
SELECT COUNT(luku) FROM (
SELECT luku FROM a
UNION ALL
SELECT luku FROM b
) b2
WHERE a2.luku < b2.luku)
ORDER BY luku DESC
```

Kysely palauttaa a ja b taulun sisältämien lukujen joukosta kolme suurinta lukua järjestettynä suurimmasta pienempään.


Miten voisit tehdä kyselystä luettavamman? Onko kyselyn toteuttamiseen muita keinoja?

```sql
SELECT luku from a 
UNION ALL
SELECT luku from b
ORDER BY luku DESC
LIMIT 3;
```

## 2.

Tee kysely, joka tulostaa kaikki rivit pro-kannan prhe-taulusta siten että sarakkeina ovat ptun, htun ja tot_vs_suun. Tot_vs_suun-sarakkeen arvot saadaan laskemalla prosentuaalinen ero toteutuneiden ja suunniteltujen tuntien välillä (löytyy sqljatko.pdf-dokumentista). Muotoile sarakkeen tulostusmuotoa siten, että

- jos prosentuaalinen ero = 0, tulostuu "tasan"
- jos prosentuaalinen ero > 0, tulostuu "Yli", prosettiluku kokonaislukuna ja prosenttimerkki (esim. "Yli 55%")
- jos prosentuaalinen ero < 0, tulostuu "Ali", prosenttiluku positiivisena kokonaislukuna ja prosenttimerkki (esim. "Ali 50%")
- jos prosentuaalinen ero on NULL, tulostuu "Tiedot puuttuu"

```sql
SELECT ptun, htun, (
CASE 
    WHEN tunnit IS NULL OR tunnit_suun IS NULL THEN 'Tieto puuttuu'
    WHEN tunnit = tunnit_suun THEN 'Tasan'
    WHEN tunnit < tunnit_suun THEN CONCAT('Yli ', ROUND(((tunnit_suun / tunnit - 1)* 100)), '%')
    WHEN tunnit > tunnit_suun THEN CONCAT('Alle ', ABS(ROUND(((tunnit_suun / tunnit - 1)* 100))), '%')
END
) AS tot_vs_suun
FROM Prhe;
```

## 3.

Kirjoita tulokset-kantaan triggerit, jotka lisäävät uutiset-tauluun tiedotteen kun joukkueet-tauluun lisätään tai siitä poistetaan joukkue. Uusi joukkue ilmoitetaan tyyliin "6.2.2019: joukkue Tiko United liittyi sarjaan" ja joukkueen poistuminen sarjasta tyyliin "6.2.2019: joukkue Tiko United suljettiin sarjasta".

```sql
DROP TABLE IF EXISTS uutiset;
CREATE TABLE uutiset (
id INT NOT NULL AUTO_INCREMENT,
kuvaus VARCHAR(200) NOT NULL,
CONSTRAINT uutiset_pk PRIMARY KEY (id)
) ENGINE=InnoDB;

DROP TRIGGER IF EXISTS lisaa_joukkue_trg;
DELIMITER  //
CREATE TRIGGER lisaa_joukkue_trg AFTER INSERT ON joukkueet
FOR EACH ROW
BEGIN
    DECLARE uutisteksti VARCHAR(200);

    SELECT CONCAT(DATE_FORMAT(NOW(), '%d.%m.%Y'), ': joukkue ', 
    (SELECT nimi FROM joukkueet WHERE id = NEW.id),
    ' liittyi sarjaan') INTO uutisteksti;
    
    INSERT INTO uutiset(kuvaus) VALUES (uutisteksti);
END;
//

DELIMITER ;

DROP TRIGGER IF EXISTS poista_joukkue_trg;
DELIMITER  //
CREATE TRIGGER poista_joukkue_trg BEFORE DELETE ON joukkueet
FOR EACH ROW
BEGIN
    DECLARE uutisteksti VARCHAR(200);

    SELECT CONCAT(DATE_FORMAT(NOW(), '%d.%m.%Y'), ': joukkue ', 
    (SELECT nimi FROM joukkueet WHERE id = OLD.id),
    ' suljettiin sarjasta') INTO uutisteksti;
    
    INSERT INTO uutiset(kuvaus) VALUES (uutisteksti);
END;
//

DELIMITER ;

INSERT INTO joukkueet (nimi) VALUES ('Teletapit');
SELECT * FROM uutiset;
SET SQL_SAFE_UPDATES = 0;
DELETE FROM joukkueet WHERE joukkueet.nimi = 'Teletapit';
```

## 4.

Kirjoita tulokset-kantaan funktio "otteluidenLkm", joka saa parametrinaan kahden joukkueen id:t. Funktio palauttaa joukkueiden keskinäisten otteluiden lukumäärän. Muista huomioida sekä koti- että vierasottelut kummaltakin joukkueelta.

```sql
DROP FUNCTION IF EXISTS otteluidenLkm
DELIMITER //
CREATE FUNCTION otteluidenLkm(p_j1 INT, p_j2 INT) RETURNS INT
DETERMINISTIC READS SQL DATA
BEGIN
    DECLARE lkm INT;
    SELECT COUNT(*) FROM ottelut
    WHERE (kotijoukkue = p_j1 AND vierasjoukkue = p_j2)
    OR (kotijoukkue = p_j2 AND vierasjoukkue = p_j1)
    INTO lkm;
    RETURN lkm;
END;
// 

DELIMITER ;

SELECT otteluidenLkm(2,3);
```

## 5.

Kirjoita tulokset-kantaan proseduuri "kotisaldo", joka saa IN-parametrinaan joukkueen id:n. Proseduuri palauttaa kolmessa OUT-parametrissa tiedot parametrina välitetyn joukkueen kotivoittojen, kotitasapelien ja kotitappioiden määristä. Proseduurin otsikkorivi voi näyttää esimerkiksi tällaiselta:

CREATE PROCEDURE kotisaldo(IN p_j INT, OUT po_voitot INT, OUT po_tasurit INT, OUT po_tappiot INT)

- valitse kaikki rivit, jossa joukkue on kotijoukkueena
- iteroi rivit, laske voitoksi, tappioksi tai tasuriksi
- palauta arvot

```sql
DROP PROCEDURE IF EXISTS kotisaldo;

DELIMITER //
CREATE PROCEDURE kotisaldo(IN p_j INT, OUT po_voitot INT, OUT po_tasurit INT, OUT po_tappiot INT)
DETERMINISTIC READS SQL DATA
BEGIN
    DECLARE v_kotimaalit INT DEFAULT 0;
    DECLARE v_vierasmaalit INT DEFAULT 0;
    DECLARE v_finished BOOLEAN DEFAULT FALSE;

    DECLARE c_ottelut CURSOR FOR 
    SELECT kotimaalit, vierasmaalit FROM ottelut WHERE kotijoukkue = p_j;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_finished = TRUE;

    SET po_voitot = 0;
    SET po_tasurit = 0;
    SET po_tappiot = 0;

    OPEN c_ottelut;

    koti_tulokset: LOOP

    FETCH c_ottelut INTO v_kotimaalit, v_vierasmaalit;
    IF v_finished THEN LEAVE koti_tulokset;
    END IF;

    IF v_kotimaalit > v_vierasmaalit THEN SET po_voitot = po_voitot + 1;
    ELSEIF v_kotimaalit < v_vierasmaalit THEN SET po_tappiot = po_tappiot + 1;
    ELSE SET po_tasurit = po_tasurit + 1;
    END IF;

    END LOOP koti_tulokset;
    CLOSE c_ottelut;

END;
//

DELIMITER ;

CALL kotisaldo(2, @voitot, @tasurit, @tappiot);
SELECT @voitot AS kotivoitot, @tasurit AS kotitasurit, @tappiot AS kotitappiot;
```
