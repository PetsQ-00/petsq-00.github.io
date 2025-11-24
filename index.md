Tietokannat, Indeksit ja Transaktiot
Dia 1: Indeksointi
Indeksejä käytetään, jotta kyselyt voivat löytää taulujen tiedot nopeasti. Indeksejä luodaan tauluihin ja näkymiin.

Indeksi taulussa tai näkymässä on hyvin samanlainen kuin kirjan hakemisto. Jos kirjassa ei ole hakemistoa ja pyydän sinua löytämään tietyn luvun, joudut selaamaan kirjaa sivu sivulta alusta alkaen. Jos kirjassa on hakemisto, etsit luvun sivunumeron hakemistosta ja voit siirtyä suoraan oikealle sivulle. Hakemisto nopeuttaa siis merkittävästi luvun löytämistä.

Samalla tavalla taulujen ja näkymien indeksit auttavat kyselyä löytämään tiedot nopeasti. Oikeat indeksit voivat parantaa kyselyjen suorituskykyä huomattavasti.

Jos indeksia ei ole, kyselymoottori käy taulun läpi rivi riviltä alusta loppuun. Tätä kutsutaan table scaniksi. Table scan on hidasta ja heikentää suorituskykyä.

Dia 2: Indeksointiesimerkki
Esimerkki

Opettaja-taulukko:

opettajanro	etunimi	sukunimi	palkka	puhelin	syntpvm
h180	Seppo	Kokki	3165.00	09-808800	1967-02-03
h290	Sisko	Saari	4604.60	09-776655	1972-11-01
h303	Veli	Ponteva	1950.00	09-123456	1997-07-25
h430	Emma	Virta	5300.04	015-33002	1966-04-18
h560	Olka	Tahko	3180.00	09-666977	1972-05-30
h784	Veera	Vainio	4699.76	09-203749	1956-04-12
SELECT * FROM opettaja
WHERE palkka > 4600 AND palkka < 4700;
Kyselyn tulos:

opettajanro	etunimi	sukunimi	palkka	puhelin	syntpvm
h290	Sisko	Saari	4604.60	09-776655	1972-11-01
h784	Veera	Vainio	4699.76	09-203749	1956-04-12
Ilman indeksiä käydään taulun jokainen rivi läpi (table scan). Iso taulu, hidasta.

Tietokanta SWD03

Dia 3: Indeksin luonti
Indeksin luonti

CREATE INDEX IX_Opettaja_palkka
ON opettaja (palkka ASC);
Indeksitaulu:

Palkka (€)	Tietueen osoite
1950.00	Tietueen osoite
3165.00	Tietueen osoite
3180.00	Tietueen osoite
4604.60	Tietueen osoite
4699.76	Tietueen osoite
5300.00	Tietueen osoite
B+-tree rakenne:

        [3180]
       /      \
   [1950]    [4604.60]
   /   \      /       \
1950  3165  4604.60  4699.76
              |
           5300.00
Hakualgoritmi (palkka 4600-4700):

Vertaa 3180 → oikea haara
Vertaa 4604.60 → löytyi!
Lue lehtisolmu → 4604.60 ja 4699.76
Siirry suoraan tietueisiin osoitteiden kautta
Nyt etsittäessä opettajien palkkoja ei tarvitse käydä ensin koko taulua läpi, vaan voidaan indeksitaulusta nopeasti etsiä oikeat palkka-arvot. Indeksitaulussa palkat on järjestyksessä. Yleensä B+-tree (binäärihaku).

Dia 4: Clustered / Non-clustered / Paksu indeksit
Clustered / Non-clustered / Paksu indeksit

Klustoroitu indeksi (Clustered Index)
Opettaja-taulussa:

SQL Server luo klusteroiden indeksin automaattisesti pääavaimelle opettajanro
Taulun tiedot järjestetään fyysisesti opettajanro-arvon mukaan
Voi olla vain yksi klustoroitu indeksi taulussa
Kaikki tieto on heti saatavilla ilman osoitteen hakemista
Esimerkki:

-- Klustoroitu indeksi opettajanro:lle (luotu automaattisesti)
-- Taulun rivit fyysisessä järjestyksessä: h180, h290, h303, h430, h560, h784
Ei-klustoroitu indeksi (Non-clustered Index)
Opettaja-taulussa:

Esim. palkka-sarakkeelle luotu indeksi on ei-klustoroitu
Indeksi on erillinen hakemisto, joka viittaa taulun tietueisiin
Indeksitaulussa: palkka-arvot järjestyksessä + osoitteet tietueisiin
Esimerkki:

CREATE INDEX IX_Opettaja_palkka ON opettaja (palkka ASC);
-- Indeksitaulu sisältää: palkka-arvot + osoitteet opettaja-taulun riveihin
-- Haku: 1) Etsi indeksista palkka, 2) Siirry osoitteen kautta taulun riviin
Paksu indeksi (Covering Index)
Opettaja-taulussa:

Jos kysely hakee vain palkka ja etunimi:
SELECT palkka, etunimi FROM opettaja WHERE palkka > 4600;
Paksu indeksi sisältää sekä palkka että etunimi → kyselyn ei tarvitse mennä taulun tietueisiin
Kaikki tarvittava tieto on suoraan indeksissä
Ero:

Normaali indeksi: Indeksi → osoite → taulun rivi → hae tieto
Paksu indeksi: Indeksi → kaikki tieto suoraan indeksistä
Dia 5: Transaktiot - ACID
Transaktiot - ACID

Dia 6: Transaktion / tapahtuman hallinta
KÄSITE Joukko tietokannan käsittelytoimenpiteitä (kyselyjä, lisäyksiä, muutoksia, poistoja), jotka muodostavat loogisen kokonaisuuden.

MIKSI?

Estetään tietokannan samanaikaisesta käsittelystä aiheutuvat ongelmat
Varmistetaan tietokannan tietojen eheys mahdollisista virhetilanteista huolimatta
BEGIN TRANSACTION
  -- tehdään jotain
COMMIT;
-- TAI
ROLLBACK TRANSACTION;
Dia 7: Esimerkit
Katoavat päivitykset (Lost Update Problem)
Ongelma: Kaksi käyttäjää päivittää samaa tietuetta samanaikaisesti, toinen päivitys katoaa.

Esimerkki opettaja-taululla:

Alkutilanne:

Opettaja h290 (Sisko Saari): palkka = 4604.60 €
Käyttäjä A ja Käyttäjä B haluavat nostaa palkkaa 100 €:lla:

Aika	Käyttäjä A	Käyttäjä B	Tietokanta
T1	Lukee palkka = 4604.60		palkka = 4604.60
T2		Lukee palkka = 4604.60	palkka = 4604.60
T3	Laskee: 4604.60 + 100 = 4704.60		
T4		Laskee: 4604.60 + 100 = 4704.60	
T5	Päivittää: 4704.60		palkka = 4704.60
T6		Päivittää: 4704.60	palkka = 4704.60 ❌
Ongelma: Molemmat laskivat samasta alkuarvosta, joten toinen päivitys katoaa. Lopputulos olisi pitänyt olla 4804.60 € (4604.60 + 100 + 100).

Ratkaisu: Transaktiot ja lukitukset varmistavat, että päivitykset tehdään peräkkäin.

Holtiton luku (Dirty Read Problem)
Ongelma: Luetaan muutoksia, joita ei ole vielä vahvistettu (COMMIT). Jos transaktio peruutetaan (ROLLBACK), luettu tieto on virheellistä.

Esimerkki opettaja-taululla:

Alkutilanne:

Opettaja h290 (Sisko Saari): palkka = 4604.60 €
Käyttäjä A aloittaa transaktion ja muuttaa palkkaa:

Aika	Käyttäjä A (Transaktio 1)	Käyttäjä B (Transaktio 2)	Tietokanta
T1	BEGIN TRANSACTION		palkka = 4604.60
T2	Päivittää: palkka = 5000.00		palkka = 5000.00 (ei vielä COMMIT)
T3		Lukee palkka = 5000.00 ❌	palkka = 5000.00
T4	Virhe! ROLLBACK		palkka = 4604.60 (palautettu)
T5		Käyttää virheellistä tietoa 5000.00	
Ongelma: Käyttäjä B luki palkka-arvon 5000.00, vaikka se peruutettiin. Käyttäjä B käyttää nyt virheellistä tietoa.

Ratkaisu: Isolation-taso estää lukemasta vahvistamattomia muutoksia (READ COMMITTED).

Dia 8: Milloin ja miten käyttää indeksejä?
Milloin kannattaa luoda indeksi?
Käytä indeksiä, kun:

Taulu on suuri (tuhansia tai miljoonia rivejä)
Kyselyt suoritetaan usein WHERE-ehdolla
Kyselyt käyttävät JOIN-operaatioita
Kyselyt järjestävät tuloksia (ORDER BY)
Kyselyt ryhmittelevät tuloksia (GROUP BY)
Opettaja-taulun esimerkki:

-- Hyvä: Indeksi palkka-sarakkeelle
SELECT * FROM opettaja WHERE palkka > 4600;  -- Nopea indeksin avulla

-- Hyvä: Indeksi sukunimi-sarakkeelle
SELECT * FROM opettaja WHERE sukunimi = 'Saari';  -- Nopea indeksin avulla
Älä käytä indeksiä, kun:

Taulu on pieni (alle 100 riviä)
Sarake päivitetään hyvin usein (INSERT/UPDATE hidastuu)
Sarake sisältää vain muutamia eri arvoja (esim. sukupuoli: M/N)
Kyselyt hakevat useimmiten kaikki rivit (SELECT * ilman WHERE)
Miten luoda indeksi?
Perusindeksi:

CREATE INDEX IX_Opettaja_palkka ON opettaja (palkka);
Järjestetty indeksi (nouseva/laskeva):

CREATE INDEX IX_Opettaja_palkka_ASC ON opettaja (palkka ASC);   -- Nouseva
CREATE INDEX IX_Opettaja_palkka_DESC ON opettaja (palkka DESC); -- Laskeva
Yhdistetty indeksi (useita sarakkeita):

-- Hyödyllinen, jos kyselyt käyttävät molempia sarakkeita
CREATE INDEX IX_Opettaja_sukunimi_etunimi
ON opettaja (sukunimi, etunimi);
Paksu indeksi (covering index):

-- Sisältää kaikki kyselyssä tarvittavat sarakkeet
CREATE INDEX IX_Opettaja_palkka_etunimi
ON opettaja (palkka) INCLUDE (etunimi, sukunimi);
Parhaita käytäntöjä
Indeksien nimeäminen:

Käytä selkeää nimeämiskäytäntöä: IX_TaulunNimi_SarakkeenNimi
Esimerkki: IX_Opettaja_palkka, IX_Opettaja_sukunimi
Indeksien ylläpito:

Tarkista säännöllisesti, käyttävätkö kyselyt indeksejä
Poista käyttämättömät indeksit (ne hidastavat INSERT/UPDATE)
Päivitä indeksitilastot: UPDATE STATISTICS opettaja
Varoitukset:

Liian monta indeksiä hidastaa INSERT/UPDATE/DELETE -operaatioita
Jokainen indeksi vie tilaa ja vaatii ylläpitoa
Yhdistetty indeksi auttaa vain, jos WHERE käyttää ensimmäisiä sarakkeita
Opettaja-taulun suositus:

-- Pääavain: automaattinen klustoroitu indeksi opettajanro:lle
-- Suositeltavat lisäindeksit:
CREATE INDEX IX_Opettaja_palkka ON opettaja (palkka);
CREATE INDEX IX_Opettaja_sukunimi ON opettaja (sukunimi);
-- Älä luo indeksiä puhelin-sarakkeelle, jos sitä ei haeta usein
Dia 9: ACID
ACID

Atomicity / Jakamattomuus, atomisuus
Consistency / Eheys, oikeellisuus
Isolation / Erillisyys, eristyvyys
Durability / Pysyvyys
Samanaikaisuuden hallinta

Sarjallistuvuus (Serializability)
Tavoite: Transaktiot suoritetaan niin, että lopputulos on sama kuin jos ne suoritettaisiin peräkkäin.

Esimerkki opettaja-taululla:

Alkutilanne:

Opettaja h290: palkka = 4604.60 €
Ilman sarjallistuvuutta (ongelma):

Transaktio 1: Lukee palkka = 4604.60 → Laskee 4704.60 → Päivittää 4704.60
Transaktio 2: Lukee palkka = 4604.60 → Laskee 4704.60 → Päivittää 4704.60
Lopputulos: 4704.60 € (väärä, pitäisi olla 4804.60 €)
Sarjallistuvuudella (oikein):

Transaktio 1: Lukee → Laskee → Päivittää 4704.60 → COMMIT
Transaktio 2: Odottaa → Lukee 4704.60 → Laskee 4804.60 → Päivittää 4804.60 → COMMIT
Lopputulos: 4804.60 € (oikein)
Vahvistettujen luettavuus (Read Committed)
Tavoite: Luetaan vain vahvistettuja (COMMIT) muutoksia, ei kesken olevia transaktioita.

Esimerkki opettaja-taululla:

Alkutilanne:

Opettaja h290: palkka = 4604.60 €
Aika	Transaktio 1	Transaktio 2	Tietokanta
T1	BEGIN TRANSACTION		palkka = 4604.60
T2	Päivittää: 5000.00		palkka = 5000.00 (ei COMMIT)
T3		Yrittää lukea	Odottaa (ei lue vahvistamatonta)
T4	COMMIT		palkka = 5000.00 (vahvistettu)
T5		Lukee: 5000.00 ✅	palkka = 5000.00
Ratkaisu: Transaktio 2 odottaa, kunnes Transaktio 1 on vahvistettu (COMMIT).

Lukitukset (Locks)
Tavoite: Estetään samanaikaiset muutokset samaan tietueeseen.

Esimerkki opettaja-taululla:

Alkutilanne:

Opettaja h290: palkka = 4604.60 €
Ilman lukitusta:

Käyttäjä A: Lukee palkka = 4604.60
Käyttäjä B: Lukee palkka = 4604.60
Käyttäjä A: Päivittää 4704.60
Käyttäjä B: Päivittää 4704.60 (katoava päivitys!)
Lukituksella:

Käyttäjä A: Lukee palkka = 4604.60 → LUKITUS (exclusive lock)
Käyttäjä B: Yrittää lukea → ODOTTAA (lukitus estää)
Käyttäjä A: Päivittää 4704.60 → COMMIT → LUKITUS VAPAA
Käyttäjä B: Lukee 4704.60 → LUKITUS → Päivittää 4804.60 → COMMIT
Lukitustyypit:

Shared lock (S-lock): Useita lukuja samaan aikaan, ei päivityksiä
Exclusive lock (X-lock): Vain yksi päivitys kerrallaan, ei muita lukuja eikä päivityksiä
Deadlock (Kuollut lukko)
Ongelma: Kaksi transaktiota odottaa toisiaan, mikään ei voi edetä. Tietokantajärjestelmä tunnistaa deadlockin ja peruuttaa yhden transaktioista.

Esimerkki opettaja-taululla:

Alkutilanne:

Opettaja h290 (Sisko Saari): palkka = 4604.60 €
Opettaja h784 (Veera Vainio): palkka = 4699.76 €
Deadlock-tilanne:

Aika	Transaktio 1 (Käyttäjä A)	Transaktio 2 (Käyttäjä B)	Tietokanta
T1	BEGIN TRANSACTION	BEGIN TRANSACTION	
T2	Lukitsee h290 (X-lock)		h290: LUKITTU (A)
T3		Lukitsee h784 (X-lock)	h784: LUKITTU (B)
T4	Yrittää lukita h784		h784: ODOTTAA (B lukitsee)
T5	ODOTTAA	Yrittää lukita h290	h290: ODOTTAA (A lukitsee)
T6	ODOTTAA	ODOTTAA	DEADLOCK! ❌
T7		Tietokanta tunnistaa deadlockin	
T8		ROLLBACK (peruutetaan)	h784: VAPAA
T9	Lukitsee h784 → Päivittää → COMMIT		h290, h784: VAPAA
Selitys:

Transaktio 1 lukitsee h290:n ja yrittää lukita h784:n
Transaktio 2 lukitsee h784:n ja yrittää lukita h290:n
Molemmat odottavat toisiaan → deadlock
Tietokantajärjestelmä tunnistaa ongelman ja peruuttaa yhden transaktioista (ROLLBACK)
Toinen transaktio voi jatkaa normaalisti
Välttäminen:

Käytä samaa lukitusjärjestystä kaikissa transaktioissa (esim. aina ensin pienempi ID)
Pidä transaktiot lyhyinä
Vältä käyttäjän syötettä transaktioiden aikana
Lähde: Petteri Lappalainen, 24.11.2025
