# Tietokannat 2 - Oppimistehtävä 4 - MongoDB

Jarmo Syvälahti

2019

## Tehtävänanto

Suunnittele Mongo-kanta alla olevan kuvauksen perusteella, luo tarvittavat JSON-tiedostot ja importoi ne. Perustele, miksi teit kyseisen kantaratkaisun. Vastauksessa tulee olla importointikomennto ja tiedostojen sisällöt siinä muodossa, että niistä pystyy suoraan luomaan kannan (esimerkiksi erillisinä tiedostoina).

Casen kuvaus

Olet tekemässä kantaratkaisua pienimuotoiseen verkkopalveluun, jossa arvostellaan suomalaisia indie-pelejä. Jotta käyttäjä voi kirjoittaa arvostelun, hänen tulee olla rekisteröitynyt palveluun. Arvosteluja per peli arvioidaan tulevan maksimissaan noin 10 kappaletta ja pelejä palveluun saattaa tulla muutamia tuhansia kappaleita. Arvostelijoita odotetaan olevan muutamia kymmeniä. Järjestelmä on kokonaan suomenkielinen ja arvostelut pitää myös kirjoittaa suomeksi. Arvostelussa tulee olla pelin nimi, kehittäjien nimet, alustat joille peli on saatavissa, peliin liittyvät tagit (esim. genre), lyhyt kuvaus pelistä, arvosana (1-5), arvostelijan nimi ja arvostelupäivämäärä.

Kannassa on alkutilanteessa kolme arvostelijaa: Pekka Pennanen, Anneli Auvinen ja Sami Santanen. Pekan käyttäjätunnus on "pekka" ja salasana on "qwerty123", Annelin "ansku" ja "salasana1" sekä Samin "santtu" ja "passu#!". Lisäksi kannassa tulee olla seuraavat arvostelut:

- Peli: "Paha päivä Tikossa", Kehittäjät: "Reino Ruuskanen, Martti Mielikäinen, Iiris Ikävalko ja Taavetti Talikainen", Alustat: "PC", Tagit: "seikkailu, puzzle", Kuvaus: "Karulla grafiikalla ryyditetty seikkailupeli, jossa opiskelija joutuu ikäviin tilainteisiin ilkeiden opettajien vuoksi. Puzzlet pääasiassa helppoja ja pelin läpäisy ei kestä tuntia kauempaa.", Arvosana: 3, Arvostelija: "Pekka Pennanen", päivämäärä: "29.11.2015".
- Peli: "Flying Balls", Kehittäjät: "Pekka Pennanen, Gunilla Grunström", Alustat: "Android, iPhone", Tagit: "casual, lapset", Kuvaus: "Mukavaa ajanvietettä lapsille puhkomalla lentäviä palloja. Mitä nopeampi olet, sitä enemmän saat pisteitä. Ei mitään uutta tai mullistavaa, mutta kokeilemisen arvoinen", Arvosana: 4, Arvostelija: "Anneli Auvinen", Päivämäärä: "30.11.2015".
- Peli: "Töttöröö ja tärkeä tapaaminen", Kehittäjät: "Reino Ruuskanen, Iiris Ikävalko ja Taavetti Talikainen", Alustat: "PC, Android-tabletit, iPad", Tagit: "seikkailu, puzzle, lapset", Kuvaus: "Epäonnistunut yritys tehdä lapsia kiinnostava seikkailupeli. Kenttien ratkaisut epäloogisia, eikä pelaaja koe olevansa pelissä aktiivinen toimija lainkaan.", Arvosana: 1, Arvostelija: "Pekka Pennanen", päivämäärä: "30.11.2015".
- Peli: "Hajota!", Kehittäjät: "Gunilla Grunström, Martti Mielikäinen", Alustat: "PC", Tagit: "Arcade", Kuvaus: "Breakout-klooni, jossa ei mitään uutta.", Arvosana: 2, Arvostelija: "Sami Santanen", Päivämäärä: "30.11.2015".

Toteuta lisäksi shellissä seuraavat tehtävät tekevät komennot:

- Listaa kaikki kannassa olevat pelit lajiteltuna parhaasta arvosanasta huonoimpaan.
- Listaa kaikki pelit, joissa on tagi "puzzle".
- Listaa kaikki Pekka Pennasen tai Anneli Auvisen arvostelemat pelit. Näytä vain pelin nimi, arvosana ja päivämäärä.
- Näytä kaikkien kannassa olevien pelien arvostelujen keskiarvo.
- Listaa kaikki tagit ja niiden esiintymislukumäärä. Lajittele suurimmasta pienimpään.

## Kantasuunnitelma

Suunnitteluperustelut:
- käyttäjät omaan kokoelmaan, koska rekisteröityminen on oma toiminnallisuutensa
- pelit oma kokoelma, koska niitä arvioidaan tulevan tuhansia
- pelien alikokelmana voi olla arvostelut, koska niitä käsitellään pelien yhteydessä ja niitä tulee vain muutamia per peli
- kehittäjät: omaan kokoelmaan ja viittaustaulukkona pelit-kokoelmaan vai taulukossa pelit-kokelmassa?
- alustat: omaan kokoelmaan ja viittaustaulukkona pelit-kokoelmaan vai taulukossa pelit-kokelmassa?
- tagit: omaan kokoelmaan ja viittaustaulukkona pelit-kokoelmaan vai taulukossa pelit-kokelmassa?
- arvostelija: viittauksena käyttäjät-kokoelmaan vai nimi merkkijonona?

Jos sama tieto, joka toistuu useammassa paikkaa on omana kokoelmanaan ja viittauksena toisessa kokoelmassa, on tietoa helpompi ylläpitää mutta hakeminen hidastuu. Jos samaa tietoa monistetaan dokumentista toiseen, on taas riskinä virheet ja tiedon päivittämisen hankaluus. 

En tiedä kantaa käyttävän sovelluksen yleisistä käyttötavoista kovin tarkasti, joten arvaan, että pääasiassa sillä haetaan ja luetaan arvosteluja. Päätän laittaa sellaisetkin tiedot, joihin viitataan useammasta paikkaa pelit-kokoelman alle, mutta viitata arvostelijan nimen kohdalla kayttajat-kokoelmaan.

[MongoDB:n dokumentaation artikkeli aiheesta](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-referencing)  
[MongoDB:n blogiartikkeli aiheesta](https://www.mongodb.com/blog/post/thinking-documents-part-1?jmp=docs)  
[Stack Overflow ketju aiheesta](https://stackoverflow.com/questions/5373198/mongodb-relationships-embed-or-reference)  

Suunnitelma rakenteesta:

```
kayttajat:
- nimi
- kayttajatunnus
- salasana

pelit:
- nimi
- kehittajat: []
- alustat: []
- tagit: []
- arvostelut: []
  - kuvaus
  - arvosana
  - arvostelija
  - pvm
```
## Kokoelmat

kayttajat-kokoelma

```json
[{
  "_id": "pekka",
  "nimi": "Pekka Pennanen",
  "salasana": "qwerty123"
},
{
  "_id": "ansku",
  "nimi": "Anneli Auvinen",
  "salasana": "salasana1"
},
{
  "_id": "santtu",
  "nimi": "Sami Santanen",
  "salasana": "passu#!"
}]
```

pelit-kokoelma

```json
[{
  "nimi": "Paha päivä Tikossa",
  "kehittajat": ["Reino Ruuskanen", "Martti Mielikäinen", "Iiris Ikävalko", "Taavetti Talikainen"],
  "alustat": ["PC"],
  "tagit": ["seikkailu", "puzzle"],
  "arvostelut": [{
    "kuvaus": "Karulla grafiikalla ryyditetty seikkailupeli, jossa opiskelija joutuu ikäviin tilainteisiin ilkeiden opettajien vuoksi. Puzzlet pääasiassa helppoja ja pelin läpäisy ei kestä tuntia kauempaa.",
    "arvosana": 3,
    "kayttajat_id": "pekka",
    "pvm": "2015-11-29"
  }]
},
{
  "nimi": "Flying Balls",
  "kehittajat": ["Pekka Pennanen", "Gunilla Grunström"],
  "alustat": ["Android", "iPhone"],
  "tagit": ["casual", "lapset"],
  "arvostelut": [{
    "kuvaus": "Mukavaa ajanvietettä lapsille puhkomalla lentäviä palloja. Mitä nopeampi olet, sitä enemmän saat pisteitä. Ei mitään uutta tai mullistavaa, mutta kokeilemisen arvoinen",
    "arvosana": 4,
    "kayttajat_id": "ansku",
    "pvm": "2015-11-30"
  }]
},
{
  "nimi": "Töttöröö ja tärkeä tapaaminen",
  "kehittajat": ["Reino Ruuskanen", "Iiris Ikävalko", "Taavetti Talikainen"],
  "alustat": ["PC", "Android-tabletit", "iPad"],
  "tagit": ["seikkailu", "puzzle", "lapset"],
  "arvostelut": [{
    "kuvaus": "Epäonnistunut yritys tehdä lapsia kiinnostava seikkailupeli. Kenttien ratkaisut epäloogisia, eikä pelaaja koe olevansa pelissä aktiivinen toimija lainkaan.",
    "arvosana": 1,
    "kayttajat_id": "pekka",
    "pvm": "2015-11-30"
  }]
},
{
  "nimi": "Hajota!",
  "kehittajat": ["Gunilla Grunström", "Martti Mielikäinen"],
  "alustat": ["PC"],
  "tagit": ["arcade"],
  "arvostelut": [{
    "kuvaus": "Breakout-klooni, jossa ei mitään uutta.",
    "arvosana": 2,
    "kayttajat_id": "santtu",
    "pvm": "2015-11-30"
  }]
}]
```

## Importointi ja kyselyt

Kokoelmien importointi docker kontissa pyörivään mongokantaan:

```
docker cp kayttajat.json tk2-mongo:/tmp/kayttajat.json
docker exec tk2-mongo mongoimport -d pelikanta -c kayttajat --file /tmp/kayttajat.json --jsonArray

docker cp pelit.json tk2-mongo:/tmp/pelit.json
docker exec tk2-mongo mongoimport -d pelikanta -c pelit --file /tmp/pelit.json --jsonArray
```

Kyselyt:

```javascript
// Listaa kaikki kannassa olevat pelit lajiteltuna parhaasta arvosanasta huonoimpaan.
db.pelit.find().sort({'arvostelut.0.arvosana': -1});

// Listaa kaikki pelit, joissa on tagi "puzzle".
db.pelit.find({tagit: 'puzzle'});

// Listaa kaikki Pekka Pennasen tai Anneli Auvisen arvostelemat pelit. Näytä vain pelin nimi, arvosana ja päivämäärä.
db.kayttajat.find({nimi: {$in: ['Pekka Pennanen', 'Anneli Auvinen']}}, {_id: 1}); // pekka, ansku
db.pelit.find({'arvostelut.kayttajat_id': {$in: ['pekka', 'ansku']}}, {_id: 0, nimi: 1, 'arvostelut.arvosana': 1, 'arvostelut.pvm': 1});

// Näytä kaikkien kannassa olevien pelien arvostelujen keskiarvo.
db.pelit.aggregate([{$unwind: '$arvostelut'}, {$group: {_id: null, arvosana_avg: {$avg: '$arvostelut.arvosana'}}}, {$project: { _id: 0, arvosana_avg: 1}}]);

// Listaa kaikki tagit ja niiden esiintymislukumäärä. Lajittele suurimmasta pienimpään.
db.pelit.aggregate([{$unwind: '$tagit'}, {$group: {_id: '$tagit', lmk: {$sum: 1}}}, {$sort: {lmk: -1}}]);

```
