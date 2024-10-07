# Documentazione delle API del sito principale di Trenitalia
Per ora tutti hanno documentato solo le (strane) api di Viaggiatreno, quindi oggi condividerò quello che fino ad ora sono riuscito a capire riguardo le API del sito ufficiale.
> [!WARNING]
> Questa documentazione risale al giorno `07/10/2024`, il sito potrebbe cambiare i suoi endpoint e/o parametri per la richiesta in un qualsiasi momento!

## Elenco delle stazioni
L'elenco delle stazioni è un semplice array in un file JSON, niente di che.\
Da notare che a differenza di Viaggiatreno questo sito non richiede (almeno per ora) i codici (e quando li richiede sono diversi da quelli di viaggiatreno, ovviamente) ma il tutto funziona con il nome della stazione.
```
GET https://www.trenitalia.com/content/dam/tcom/config/stationList.json
```
Esempio della risposta (ovviamente ridotto):
```json
[
  {
    "Abano": {
      "isF": 0,
      "FB": 0,
      "FA": 0,
      "isE": 0
    }
  },
  {
    "Abbadia Lariana": {
      "isF": 0,
      "FB": 0,
      "FA": 0,
      "isE": 0
    }
  },
  // altre stazioni con lo stesso formato
]
```

## Recuperare i codici delle stazioni
Siccome la ricerca per nome non è tanto efficiente e Trenitalia vuole apparentemente far fare più richieste possibili ai client siccome hanno i server di dio la richiesta per il codice di una fermata viene eseguita nella pagina delle soluzioni.\
Il payload è veramente semplice in quanto serve solo il nome (anche parziale) ed eventualmente il limite delle risposte che si vuole ricevere.
```
GET https://www.lefrecce.it/Channels.Website.BFF.WEB/website/locations/search
    Params:
      - name=Genova Brignole
      - limit=1
    > https://www.lefrecce.it/Channels.Website.BFF.WEB/website/locations/search?name=Genova Brignole&limit=1
```
La risposta è molto semplice:
```json
[
	{
		"id": 830004702,
		"name": "Genova Brignole",
		"displayName": "Genova Brignole",
		"timezone": "",
		"multistation": false,
		"centroidId": null
	}
]
```
E' possibile anche usare questo endpoint come autocompletamento:
```
GET https://www.lefrecce.it/Channels.Website.BFF.WEB/website/locations/search?name=Genova&limit=5
```
```json
[
	{
		"id": 830004999,
		"name": "Genova ( Tutte Le Stazioni )",
		"displayName": "Genova ( Tutte Le Stazioni )",
		"timezone": "",
		"multistation": true,
		"centroidId": null
	},
	{
		"id": 830004702,
		"name": "Genova Brignole",
		"displayName": "Genova Brignole",
		"timezone": "",
		"multistation": false,
		"centroidId": null
	},
	{
		"id": 830004700,
		"name": "Genova Piazza Principe",
		"displayName": "Genova Piazza Principe",
		"timezone": "",
		"multistation": false,
		"centroidId": null
	},
	{
		"id": 830004107,
		"name": "Genova Acquasanta",
		"displayName": "Genova Acquasanta",
		"timezone": "",
		"multistation": false,
		"centroidId": null
	},
	{
		"id": 830013919,
		"name": "GENOVA AEROPORTO C. COLOMBO",
		"displayName": "GENOVA AEROPORTO C. COLOMBO",
		"timezone": "",
		"multistation": false,
		"centroidId": null
	}
]
```

## Ricerca delle soluzioni
Ottenuti i codici della stazione di partenza e di arrivo è necessario richiedere tutte le soluzioni disponibili tramite questo endpoint.\
Il metodo di richiesta è `POST` e richiede un body in JSON che verrà documentato sotto.
```
POST https://www.lefrecce.it/Channels.Website.BFF.WEB/website/ticket/solutions
```
Il payload, in questo caso un JSON, è così composto:
```
{
   "departureLocationId":ID della stazione di partenza,
   "arrivalLocationId":ID della stazione di arrivo,
   "departureTime":timestamp ISO 8601,
   "adults":numero degli adulti,
   "children":numero dei ragazzi,
   "criteria":{
      "frecceOnly":se cercare solo freccie (FA, FB, FR),
      "regionalOnly":se cercare solo regionali (R, RV),
      "intercityOnly":se cercare solo intercity (IC, ICN),
      "tourismOnly":se cercare solo treni TTI (Treni Turistici Italiani) oserei dire,
      "noChanges":se non si vogliono i cambi,
      "order":indica l'ordine delle soluzioni,
      "offset":non ne ho idea,
      "limit":direi che limita quante vengono caricate e la richiesta viene ripetuta quando si oltrepassa il limite con lo scrolling
   },
   "advancedSearchRequest":{
      "bestFare":direi che è per trovare la tariffa migliore, in disuso,
      "bikeFilter":per trovare treni che permettono le biciclette a bordo
   }
}
```
Ecco un esempio:
```
{
   "departureLocationId":830004702,
   "arrivalLocationId":830000219,
   "departureTime":"2024-10-18T18:00:00.000",
   "adults":1,
   "children":0,
   "criteria":{
      "frecceOnly":false,
      "regionalOnly":false,
      "intercityOnly":false,
      "tourismOnly":false,
      "noChanges":false,
      "order":"DEPARTURE_DATE",
      "offset":0,
      "limit":10
   },
   "advancedSearchRequest":{
      "bestFare":false,
      "bikeFilter":false
   }
}
```
Gli `order` disponibili sono i seguenti: `DEPARTURE_TIME`, `ARRIVAL_TIME`, `FASTEST` e `CHEAPEST`.\
Credono siano autoesplicativi.\
\
