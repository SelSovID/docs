# Software architecture document

## Self-sovereign Identity

## Auteurs

Wouter de Boer  
Daniel Hofman  
Hylbren Rijnders  
Mees van Dijk

## Inleiding

### 1.1 Doel van dit document

Het software architectuur document bevat een uitgebreide architecturale kijk op
het systeem SSI. Het beschrijft een aantal verschillende architecturale views
van het systeem om zo verschillende aspecten van het systeem te belichten. Dit
document beschrijft de verschillend eRUP views op de software architectuur
volgens het 4+1 view model.

<img src="4plus_1_views.jpg"/>

Het 4+1 model stelt de verschillende belanghebbenden in staat vanuit hun eigen
perspectief de invloed van de gekozen architectuur te bepalen.

<!-- De Process View (communicatie van processen) is niet als los hoofdstuk
uitgewerkt maar ondergebracht bij de hoofdstukken 3.3 en 5. -->

### 1.2 referenties

| titel                                   | Auteur      | vindplaats                                                                                                               |
| --------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Software architectuur document template | RUP op maat | RUP op maat                                                                                                              |
| View diagram                            | IBM         | https://posomas.isse.de/PosoMAS/core.tech.common.extend_supp/guidances/examples/four_plus_one_view_of_arch_9A93ACE5.html |
| Acceptatieplan                          | SelSolvid   | https://github.com/SelSolvID/docs/blob/master/Acceptatieplan.md                                                          |

## 2 Architecturale eisen

### 2.1 Niet-functionele eisen

Omdat het bij dit project gaat om een proof-of-concept applicatie te bouwen zijn
de niet-functionele eisen anders dan bij een gewoon project. Alle
niet-functionele eisen zijn ook opgenomen in het
[acceptatieplan](https://github.com/SelSolvID/docs/blob/master/Acceptatieplan.md)
Een aantal gegeven niet-functionele eisen zijn:

#### 2.1.1 Performance

Zoals ook genoemd in het acceptatieplan moet de applicatie normaal te gebruiken
zijn, dit houdt in dat er geen lag-spikes zijn tijdens het gebruik. Verder moet
een gebruiker niet hoeven wachten op lange laadtijden.

#### 2.1.2 Beheerbaarheid

Voor het project wordt een teststraat ingericht. Dit houd in dat er drie
verschillende versies van de applicatie beschikbaar worden gemaakt op elk
moment. Praktisch betekent dat dat er drie versies van het complete deployment
diagram (zie 5.3) worden gedeployed.

#### 2.1.3 Betrouwbaarheid

Het systeem moet altijd beschikbaar zijn. Dit wordt behaald door dat de basis
interacties van het systeem (verifyen van een credential) niet met een centrale
server hoeven te praten. Hiermee ligt de verantwoordelijkheid bij de gebruiker
om een werkend apparaat te onderhouden.

#### 2.1.4 Beveiliging

Beveiliging staat centraal in dit project. Daarom wordt het OpenSSL package
gebruikt (zie 4.1). Dit package staat er om bekend betrouwbare beveiliging te
bieden.

### 2.2 Use case View

De use cases van het project worden in meer detail beschreven in het use-case
document. De volgende use-cases zijn ge??dentificeerd:

- Een issuer wilt requests goedkeuren en of afkeuren
- Een gebruiker wilt alcohol kopen, benodigd is de leeftijd
- Afsluiten van een dienst met voorwaarden
- Een gebruiker wilt solliciteren, daar zijn een ID en de diplomas van de
  middelbare school en de HBO instelling voor nodig
- Huren van goederen
- Het afsluiten en het checken van een verzekering.

## 3 Logical view

### 3.1 Lagen

De applicatie bestaat uit meerdere lagen. Voor het uitvoeren van acties met de
applicatie moet er een presentatie/interactie laag zijn. Deze laag communiceert
met een service laag om deze acties waar te maken. Deze service laag praat met
een data laag, welke zich op elk device zelf zal bevinden. Deze data laag bevat
ondertekende verifiable credentials. Omdat deze VC's ondertekend zijn is er geen
externe connectie nodig om te verifi??ren dat ze valide zijn. Daarom is het
voldoende om ze lokaal op te slaan.

Een andere data laag is verantwoordelijk voor het behandelen van globale data.
Deze globale data bestaat uit nog niet opgehaalde VC's en openstaande verzoeken
tot ondertekening. Zodra deze verzoeken volledig afgehandeld zijn kunnen ze ook
verwijderd worden uit deze data laag.

#### 3.1.1 Laag-diagram

<img src="http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/SelSolvID/docs/master/diagrams/layer.puml"/>

### 3.2 Deelsystemen

De applicatie vereist een aantal deelsystemen. In grote lijnen komen er een app,
een webapp en een api voor de webapp en app.

De app is bedoeld voor de gebruikers die willen verifyen en holden van
verifiable credentials (VC's).

De webapp is bedoeld voor issuers, die VC's willen uitgeven aan individuen.

De api is voor de webapp en de app, via de api krijgen de apps hun nieuwe VC's

### 3.3 Use case realizations

Alle use cases zullen gebouwd worden op een zelfde techniek. Bij het verifi??ren
van een VC wordt een peer-to-peer connectie opgezet tussen twee mobiele
telefoons. De holder stuurt dan de relevante VC naar de verifier. De verifier
kan, omdat deze de benodigde root certificaten al bezig, verifi??ren dat de VC
geldig is.  
Om de peer-to-peer connectie op te zetten wordt wordt er gebruik gemaakt van de
API om de beide users te verbinden aan elkaar, om zo data te wisselen met
elkaar. Hier wordt verder op ingegaan in Hoofdstuk 5. Verder wordt er gebruik
gemaakt van bekende cryptografie libraries om de certificaten aan te maken en te
verifi??ren.

## 4 implementation view

### 4.1 Package structuur

Voor het implementeren van de applicatie worden een aantal packages geschreven.
Omdat niet elk deelsysteem in dezelfde taal wordt gebouwd wordt het niet altijd
mogelijk om code uit deze packages te delen. Voor de api worden een aantal
models geschreven die ook door de webapp kan hergebruiken. De app zal, omdat
deze niet dezelfde programmeertaal gebruikt, dit model zelf moeten dupliceren.

Verder zullen er een aantal externe packages gebruikt worden. Voor het
realiseren van de api wordt gebruik gemaakt van [koajs](https://koajs.com/).
KoaJs is een populair, modern webframework voor het installeren van middleware
in een web applicatie. Verder biedt het uitbreidingen voor dingen zoals routing,
logging en automatisch requests parsen.

Verder wordt er bij alle cryptografische handelingen gebruik gemaakt van
[OpenSSL](https://www.openssl.org/). OpenSSL is de meest populaire toolkit voor
cryptografie en veilige communicatie.

Voor het realiseren van een user interface op android wordt gebruik gemaakt van
de standaard android libraries. Kotlin kan hiervan, net zoals java, gebruik
maken om gemakkelijk een user interface te bouwen.

De web applicatie gaat gebruik maken van [Svelte](https://svelte.dev/). Svelte
is een framework om responsive user interfaces te bouwen. Bovenop svelte gaan we
[Svelte material UI](https://sveltematerialui.com/) gebruiken. Dit is een set
componenten die makkelijk kunnen worden gebruikt in het bouwen van een ui.

#### 4.1.1 Package diagram

<img src="http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/SelSolvID/docs/master/diagrams/package.puml"/>

### 4.2 Invulling lagenstructuur

#### 4.2.1 Presentatie

De presentatie wordt op twee verschillende plekken op verschillende manieren
uitgevoerd. Dit komt omdat er twee verschillende applicaties worden gebouwd als
onderdeel van het systeem.

De twee plekken zijn: de app voor op mobiele telefoons en de website.  
Voor de app worden standaard Android libraries gebruikt die het gemakkelijk
maken om een user interface te maken. De taal waarin de android app geschreven
wordt is Kotlin. Kotlin wordt gebruikt omdat het een modern alternatief is op
java. Traditioneel wordt java gebruikt voor android apps, maar kotlin is daarop
een modern alternatief.

Voor de website wordt het web framework Svelte gebruikt. Dit is een web
framework dat zich focust op de presentatie-laag van websites. Dit framework
maakt het gemakkelijker om interactieve web applicaties te bouwen. Svelte is een
relatief nieuw web framework dat erg positief is ontvangen. Het zorgt er voor
dat developers snel en gestructureerd kunnen werken. Daarom gebruiken we Svelte.

#### 4.2.2 Service laag

De service laag is verantwoordelijk voor het uitvoeren van de business logic. In
dit geval zal dat zijn het aanvragen, aanmaken, ophalen, delen en verifieren van
VC's.

Voor het aanvragen en ophalen van VC's wordt http gebruikt. Een holder kan via
haar telefoon verbinden met de api om een VC op te halen, mits deze is
goedgekeurd, en dus ondertekend, door de relevante instantie. De holder kan ook
via haar telefoon een VC aanvragen. Deze moet dan worden goedgekeurd in de web
applicatie.

Om dit allemaal te realiseren moeten veel verschillende technieken gebruikt
worden. Veel van deze acties hebben te maken met cryptografie, hier zullen
standaard (ingebouwde) frameworks voor zijn. Deze worden ook gebruikt. Het
bouwen van deze API laag wordt gedaan met Node.js en typescript.

Voor het communiceren met de server en andere telefoons worden ook standaard
libraries gebruikt.

Verder biedt de servicelaag een Websocket server die als alternatief op
bluetooth dient. Hiermee kunnen verschillende telefoons met elkaar verbinden
voor real-time communicatie. Het protocol van deze websocket server is hieronder
in punt 4.2.2.2 beschreven.

##### 4.2.2.1 API surface

| url            | method | functie                                        |
| -------------- | ------ | ---------------------------------------------- |
| /token         | GET    | token ophalen voor gebruik in api communicatie |
| /requests      | GET    | een lijst ophalen van openstaande VC requests  |
| /requests/{id} | GET    | Details van een request ophalen                |
| /requests/{id} | PUT    | Reageren op een request (goed of afkeuren)     |
| /requests      | POST   | Als holder een nieuw request indienen          |

**GET /token** Hierbij wordt de header `Authentication` gebruikt met basic
authentication scheme.  
Dit endpoint zorgt ervoor dat de token als cookie wordt gezet in de browser
zodat de client hem automatisch bij volgende requests kan meesturen. Dit is
"inloggen" in het systeem.

**GET /requests** Hier wordt het token meegegeven als auth header of als cookie.
De response body ziet er als volgt uit:

```json
[
  {
    "id": 0,
    "fromUser": {
      "email": "some@example.com"
    },
    "date": 10123718972391823,
    "requestText": "Some claim"
  }
]
```

**GET /requests/{id}** Hierbij worden de details van een request opgehaald. De
ID staat in het path. De token wordt meegegeven als auth header. De response
body ziet er uit zoals:

```json
{
  "id": 0,
  "fromUser": {
    "email": "some@example.com"
  },
  "date": 10123718972391823,
  "requestText": "Some claim",
  "attachedVCs": ["**EXPORTED VC TEXT**", "**EXPORTED VC TEXT**"]
}
```

**PUT /requests/{id}** Hiermee kan een request goed of afgekeurd worden. Het
token wordt meegegeven in de header en de request body ziet er als volgt uit:

Een accepted request:

```json
{
  "accept": true
}
```

Een niet-accepted request:

```json
{
  "accept": false,
  // reason is optional and only applicable when "accept": false
  "reason": "You lied about your age"
}
```

**POST /holder/request** Hiermee kan een holder een nieuw request indienen. De
data ziet er als volgt uit:

```json
{
  "issuerId": 01923123,
  "fromUser": "some@example.com",
  "requestText": "some text",
  "attachedVCs": ["**EXPORTED VC TEXT**", "**EXPORTED VC TEXT**"]
}
```

##### 4.2.2.2 Websocket server protocol

De websocket server werkt met JSON berichten. Het protocol biedt alleen
mogelijkheid tot het openen(/verbinden met) van een kanaal met een id, het
sluiten van een kanaal met een id en het broadcasten van een arbitrair bericht
naar een kanaal.

**Open**

```json
{
  "type": "open",
  "channel": "*channelID*"
}
```

**Close**

```json
{
  "type": "close",
  "channel": "*channelID*"
}
```

**Message**

```json
{
  "type": "message",
  "payload": "arbitrairy data"
}
```

in het Geval dat een `message` of `close` bericht wordt gestuurd stuurt de
server een bericht naar alle clients in dat kanaal. Deze zien er als volgt uit:

**Message**

```json
{
  "type": "message",
  "payload": "the message payload"
}
```

**Close**

```json
{
  "type": "close"
}
```

Als een client een message stuurt zonder `type` of met een onbekend `type` dan
stuurt de server het volgende bericht terug:

```json
{
  "type": "erorr",
  "payload": "unknown message type"
}
```

#### 4.2.3 Data laag

Voor het opslaan van verzoeken tot VC's en de statussen van deze verzoeken wordt
een SQL database gebruikt. De api laag is de enige die direct met deze database
praat. De api laag zit tussen zowel de web applicatie als de mobiele applicatie.
Hoewel de mobiele applicatie over het algemeen de api weinig nodig zal hebben.
Dit is omdat de mobiele applicatie door verifyers en holders wordt gebruikt en
deze communiceren altijd via een peer-to-peer connectie tussen twee mobiele
apparaten.

##### 4.2.3.1 Database diagram

<img src="http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/SelSolvID/docs/master/diagrams/database.puml"/>

### 4.3(Her)gebruik van componenten en frameworks

Het meest algemene component is de API, deze kan via het internet door iedereen
met de juiste toegang benaderd worden. Deze is nodig om interne applicaties
boven op te bouwen maar ook derden zouden toegang kunnen krijgen tot deze api.
Op deze manier kunnen derden hun eigen toepassingen bouwen die hun eigen doelen
beter bereiken dan een algemene applicatie.

Bij het bouwen van deze api wordt code geschreven die direct herbruikt kan
worden bij de bouw van de web applicatie. Verder kan deze code gemakkelijk
vertaald worden naar code voor de mobiele applicatie.

Dit komt omdat voor zowel de API als de webapplicatie javascript wordt gebruikt.
Boven op javascript wordt dan typescript gebruikt. Bij typescript worden
modellen van data geschreven die overal in het systeem gebruikt moeten worden.

## 5. Deployment View

### 5.1 Deployment en infrastructuur van API, database en web applicatie

Voor de deployment is het belangrijk dat de API en de webapplicatie via een http
proxy op hetzelfde domein draaien. Dit zorgt ervoor dat de webapplicatie veilig
en zonder moeilijkheden met de API mag praten.

Om dit te realiseren worden er een aantal docker containers gebouwd. Namelijk
voor:

- Het serveren van de webapplicatie
- Het draaien van de API
- Het draaien van de database.
- Het proxy-en van requests naar de web-applicatie of de API

Deze vier containers zitten allemaal in een zelfde priv?? netwerk. Alleen de
proxy kan van buitenaf benaderd worden. De proxy zet https requests om naar http
requests, zodat de individuele servers geen verantwoordelijkheid dragen voor
https, dit zorgt voor veel minder complexiteit in de code. De proxy zorgt er ook
voor dat elk request bij de juiste service aankomt. Het is alleen natuurlijk
niet mogelijk direct de database te benaderen, dat kan alleen via de API.

De proxy weet welk request voor welke service is aan de hand van een url.

Requests met een url zoals `domein.nl/api/**/*` gaan allemaal naar de api. Alle
andere requests worden naar de server van de webapplicatie gestuurd.

Al deze infrastructuur kan, middels docker, op 1 server draaien. Omdat
scalability niet van groot belang is in dit project is 1 cloud server voldoende
voor het demonstreren van het proof of concept.

### 5.2 Deployment en infrastructuur voor de mobiele applicatie

Voor het downloaden van de mobiele applicatie wordt een continuous integration
pipeline ingesteld die automatisch de mobiele applicatie bouwt en beschikbaar
stelt voor downloaden. Dit wordt gerealiseerd in github actions. Binnen github
actions worden een aantal tools gebruikt om de applicaties te bouwen. Voor
Kotlin is dat de kotlin compiler, voor svelte is dat de svelte compiler. Voor de
api wordt Typescript gebruikt om de code te transpilen.

Tijdens het gebruik van de mobiele applicatie praat de applicatie met de
bovengenoemde API, op de beschreven manier. Dit gebeurt met http(s). Naast het
praten met de API moet de mobiele applicatie ook peer-to-peer connecties kunnen
opzetten met andere instanties van de mobiele applicatie op andere mobiele
apparaten. Het plan was eerst om dit te doen via Bluetooth. Maar vanwege
problemen verder uitgelegd in het volgende kopje, is dit niet gelukt. De nieuw
bedachte optie om dit te implementeren is door gebruik te maken van de API om
beide personen aan elkaar te verbinden. Hier wordt dieper op ingegaan in 5.2.2

#### 5.2.1 Waarom de implementatie van bluetooth niet is gelukt

Voor de bluetooth zijn meerdere implementaties uitgeprobeerd; voornamelijk de
officiele
[android bluetooth documentatie](https://developer.android.com/guide/topics/connectivity/bluetooth)
op het eind. Met deze implementatie van bluetooth kwam de app het verste. De
setup was gelukt en je kon in de app je device discoverable maken (zodat andere
telefoons je kunnen vinden bij het zoeken naar bluetooth connecties).

Het eerste waar we hier tegenaan liepen was het automatisch verbinden met
elkaar. Wanneer de gebruiker de qr-code heeft gescanned van een ander device,
zouden deze twee apparaten automatisch met elkaar via bluetooth moeten
verbinden. Dit kan je doen door lokale hardware identifiers op te halen. Voor de
automatische verbinding kon je het lokale bluetooth MAC address gebruiken en
deze zou je moeten kunnen ophalen met
[BluetoothAdapter.getAddress()](<https://developer.android.com/reference/android/bluetooth/BluetoothAdapter#getAddress()>).
Jammer genoeg kan dit niet meer sinds Android 6 gezien de getAdress methode nu
altijd een waarde van `02:00:00:00:00:00` terug geeft (zie
[deze](https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-hardware-id)
documentatie). Voor dit probleem habben we wel een
[mogelijke oplossing](https://developer.android.com/guide/topics/connectivity/companion-device-pairing)
voor gevonden maar hier waren we helaas niet aan toen gekomen gezien er meerdere
andere problemen waren waar we tegenaan liepen. Hoewel het automatisch pairen
via bluetooth niet mogelijk was, zou dit nog wel handmatig mogelijk zijn als de
gebruiker dit zelf doet in de bluetooth settings van zijn/haar telefoon.

Waar we uiteindelijk vast zijn gelopen is het versturen van data via bluetooth
(bij
[deze](https://developer.android.com/guide/topics/connectivity/bluetooth/connect-bluetooth-devices)
en
[deze](https://developer.android.com/guide/topics/connectivity/bluetooth/transfer-data)
stap). Dit was ingewikkeld om te maken omdat de documentatie veel belangrijke
stappen oversloeg waardoor het stappenplan niet te volgen was. Ook was een
redelijk groot deel van de code outdated en waren meerdere gebruikte methodes
deprecated. Hierdoor was het een hele puzzel om uit te vinden wat er miste en
hoe dit toch kon werken. Dit zou uiteindelijk te veel tijd kosten om te maken
binnen de resterende tijd.

Overige pogingen waren: _AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA_

#### 5.2.2 Nieuwe oplossing

### 5.3 Teststraat

Als onderdeel van het project wordt een teststraat ingericht. Dit houdt in dat
er van het systeem drie versies beschikbaar komen. Hiervoor worden de api en
webapp drievoudig gedeployed. De mobile applicatie krijgt drie verschillende
downloads die overeenkomen met de drie gedeployde versies van de webapp en api.

### 5.4 Deployment diagram

<img src="http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/SelSolvID/docs/master/diagrams/deployment.puml">
```
