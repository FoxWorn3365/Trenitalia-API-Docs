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
```json
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
Credono siano autoesplicativi.
> [!NOTE]
> Qualora si disponesse già di un `cartId` è possibile inserirlo nel payload sopra indicato come `"cartId":"L'id del cart"`

Quando si esegue la richiesta, se tutto va bene, la risposta sarà simile alla seguente:
```json
{
	"searchId": "830004702-830000219-one_way-2024-10-18T18:00:00.000---rewq-fdsa-iokm-ttno-vcxz-FBN-S",
	"cartId": "e76a2c37-e641-4358-aa8f-995257bceb3f",
	"highlightedMessages": [],
	"solutions": [ // un array di soluzioni che analizzeremo sotto // ]
	"minimumPrices": []
}
```
E' molto, molto importante salvarsi il `cartId` in quanto ci servirà per eseguire le prossime richieste.\
Tenete conto che quell'Id rappresenta una sessione, quindi non durerà all'infinito ergo non mettetelo hard-coded!\

### Soluzione
Una soluzione viene descritta da un object abbastanza complesso ricevuto in JSON.\
Ecco un esempio:
```json
{
   "solution":{
      "_type":"TICKET",
      "id":"x6ccf7020-90ba-4d53-b224-b4e877bc6705",
      "origin":"Genova Brignole",
      "destination":"Torino Porta Nuova",
      "departureTime":"2024-10-18T18:13:00.000+02:00",
      "arrivalTime":"2024-10-18T20:33:00.000+02:00",
      "duration":"2h 20min",
      "status":"SALEABLE",
      "trains":[
         {
            "description":"12138",
            "trainCategory":"Regionale",
            "acronym":"RE",
            "denomination":"label.train.info.denomination.re",
            "name":"12138",
            "logoId":"RE",
            "urban":false
         },
         {
            "description":"2140",
            "trainCategory":"Regionale Veloce",
            "acronym":"RV",
            "denomination":"label.train.info.denomination.rv",
            "name":"2140",
            "logoId":"RV",
            "urban":false
         }
      ],
      "price":{
         "currency":"€",
         "amount":14.45,
         "originalAmount":null,
         "indicative":false
      },
      "discounts":[
         
      ],
      "nodes":[
         {
            "id":"x886a62ff-c717-472d-bd4e-fabced92d8f2",
            "origin":"Genova Brignole",
            "bdoOrigin":"S04702",
            "destination":"Genova Piazza Principe",
            "departureTime":"2024-10-18T18:13:00.000+02:00",
            "arrivalTime":"2024-10-18T18:19:00.000+02:00",
            "salable":false,
            "train":{
               "description":"12138",
               "trainCategory":"Regionale",
               "acronym":"RE",
               "denomination":"label.train.info.denomination.re",
               "name":"12138",
               "logoId":"RE",
               "urban":false
            }
         },
         {
            "id":"x4e1783d9-f4d3-4922-b148-4bed2f76ca18",
            "origin":"Genova Piazza Principe",
            "bdoOrigin":"S04700",
            "destination":"Torino Porta Nuova",
            "departureTime":"2024-10-18T18:27:00.000+02:00",
            "arrivalTime":"2024-10-18T20:33:00.000+02:00",
            "salable":false,
            "train":{
               "description":"2140",
               "trainCategory":"Regionale Veloce",
               "acronym":"RV",
               "denomination":"label.train.info.denomination.rv",
               "name":"2140",
               "logoId":"RV",
               "urban":false
            }
         }
      ],
      "additionalPrice":null,
      "description":null,
      "validities":null
   },
   "grids":[
      {
         "id":"xa3cf14a3-37a1-41c0-8806-b11ee7b6321e",
         "summaries":[
            {
               "name":"Regionale 12138",
               "description":"Genova Brignole (18:13) - Genova Piazza Principe (18:19)",
               "duration":"6min",
               "highlightedMessage":null,
               "urban":false,
               "vehicleInfo":null,
               "departureLocationName":"Genova Brignole",
               "arrivalLocationName":"Genova Piazza Principe",
               "departureTime":"2024-10-18T18:13:00.000+02:00",
               "arrivalTime":"2024-10-18T18:19:00.000+02:00",
               "trainInfo":{
                  "description":"12138",
                  "trainCategory":"Regionale",
                  "acronym":"RE",
                  "denomination":"label.train.info.denomination.re",
                  "name":"12138",
                  "logoId":"RE",
                  "urban":false
               },
               "bdoOrigin":"S04702",
               "showInfomobilityLink":false
            },
            {
               "name":"Regionale Veloce 2140",
               "description":"Genova Piazza Principe (18:27) - Torino Porta Nuova (20:33)",
               "duration":"2h 06min",
               "highlightedMessage":null,
               "urban":false,
               "vehicleInfo":null,
               "departureLocationName":"Genova Piazza Principe",
               "arrivalLocationName":"Torino Porta Nuova",
               "departureTime":"2024-10-18T18:27:00.000+02:00",
               "arrivalTime":"2024-10-18T20:33:00.000+02:00",
               "trainInfo":{
                  "description":"2140",
                  "trainCategory":"Regionale Veloce",
                  "acronym":"RV",
                  "denomination":"label.train.info.denomination.rv",
                  "name":"2140",
                  "logoId":"RV",
                  "urban":false
               },
               "bdoOrigin":"S04700",
               "showInfomobilityLink":false
            }
         ],
         "selectedOfferId":69,
         "selectedServiceId":201,
         "services":[
            {
               "id":201,
               "name":"2ª CLASSE",
               "shortName":"2ª CLASSE",
               "groupName":"LABEL.SERVICE.GROUP.2",
               "description":null,
               "descriptionStandard":"",
               "offers":[
                  {
                     "offerId":69,
                     "serviceId":201,
                     "availableServiceId":null,
                     "serviceName":"2ª CLASSE",
                     "name":"ORDINARIA",
                     "description":"Your ticket will be validated automatically on the scheduled departure of your train. Have you changed your mind? No problem! By 11.59 p.m. on the day before your trip, you can change the date and time, and on the day of travel you can change the departure time before the ticket is automatically validated. Once the ticket is validated, it can no longer be modified.",
                     "price":{
                        "currency":"€",
                        "amount":14.45,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":32767,
                     "canModify":null,
                     "canChange":null,
                     "canRefund":null,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"x9811820f-22bc-46d8-ba6f-97910b0eced9",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "xfc9ede7c-445f-42b6-9665-134e831e1ad5"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":true,
                     "selecteable":true
                  }
               ],
               "minPrice":{
                  "currency":"€",
                  "amount":14.45,
                  "originalAmount":null,
                  "indicative":false
               },
               "bestOfferId":69,
               "extendedName":null,
               "descriptionKey":"201"
            }
         ],
         "infoMessages":[
            
         ],
         "canShowSeatMap":false,
         "collapsedVisualization":true,
         "regional":true
      }
   ],
   "customizes":[
      
   ],
   "stopList":[
      
   ],
   "messages":[
      null
   ],
   "availabilities":[
      
   ],
   "nextDaySolution":false,
   "tooltip":{
      "nodeServices":[
         {
            "denomination":"label.train.info.denomination.re",
            "name":"12138",
            "offer":"Ordinaria",
            "service":"2ª Classe"
         },
         {
            "denomination":"label.train.info.denomination.rv",
            "name":"2140",
            "offer":"Ordinaria",
            "service":"2ª Classe"
         }
      ],
      "loyaltyPoints":0.00,
      "regionalLoyaltyPoints":2
   },
   "canShowSeatMap":false,
   "notAllOfferGroupStandard":false,
   "silenceArea":false,
   "fastPurchase":false,
   "canShowGrid":false,
   "co2Emission":null
}
```
E' molto lungo e complesso, quindi lascio a voi la comprensione e mi limiterò a spiegare alcuni tratti interessanti.\
`nodes` è l'elenco di treni che servono per raggiungere la destinazione dal luogo iniziale (ovvero i treni da cambiare).\
Qualora si abbia indicato nelle preferenze "soluzioni senza cambi" oppure sia una soluzione di per sè senza cambi allora ci sarà un solo elemento nell'array `nodes`.
L'`id` è molto importante per ottenere varie informazioni (verrà indicato come `solutionId`)\
Ogni elemento dell'array `nodes` contiene a sua volta un `id` (verrà indicato come `nodeId`)

## Informazioni sul carrello attuale
Per ottenere le informazioni sul carrello attuale è possibile, avendo il `cartId`, usare questo endpoint: 
```
GET https://www.lefrecce.it/Channels.Website.BFF.WEB/website/cart
    Params:
      - cartId=c88f13ca-64c3-4b7e-a963-8982a2549c25
      - currentSolutionId=x4fb9e730-b02f-4be8-b605-30b6fa7d8ff5
    > https://www.lefrecce.it/Channels.Website.BFF.WEB/website/cart?cartId=c88f13ca-64c3-4b7e-a963-8982a2549c25&currentSolutionId=x4fb9e730-b02f-4be8-b605-30b6fa7d8ff5
```
Il parametro `currentSolutionId`, se presente, aggiungerà temporaneamente (solo per la richiesta) la soluzione indicata.
Ecco un esempio delle risposte, con e senza `currentSolutionId`:
```json
{
   "solutionContainers":[ // ripetizione inutile dell'object Soluzione // ]
   "solutionToChange":null,
   "solutionPreviews":[
      {
         "id":"x4fb9e730-b02f-4be8-b605-30b6fa7d8ff5",
         "header":"VIAGGIO",
         "subheader":null,
         "title":"Genova Brignole - Torino Porta Nuova",
         "subtitle":"Viaggio 19.10.2024 (06:48)",
         "totalPrice":{
            "currency":"€",
            "amount":11.90,
            "originalAmount":null,
            "indicative":false
         }
      }
   ],
   "status":"OPENED",
   "expirationDate":null,
   "latestSolutionIds":[
      
   ],
   "temporarySolutionIds":[
      "x4fb9e730-b02f-4be8-b605-30b6fa7d8ff5"
   ],
   "totalPrice":{
      "currency":"€",
      "amount":11.90,
      "originalAmount":null,
      "indicative":false
   },
   "totalPriceCrypted":"fVfQ8qEbN7U0Hf+0aDeFPw==",
   "additionalFees":[
      
   ]
}
```
```json
{
   "solutionContainers":[
      
   ],
   "solutionToChange":null,
   "solutionPreviews":[
      
   ],
   "status":"OPENED",
   "expirationDate":null,
   "latestSolutionIds":[
      
   ],
   "temporarySolutionIds":[
      
   ],
   "totalPrice":null,
   "totalPriceCrypted":null,
   "additionalFees":[
      
   ]
}
```

## Aggiornare il carrello
E' anche possibile aggiornare il carrello.\
Questa funzione in realtà aggiorna solo l'offerta selezionata (come 1°/2° classe ed il tipo come Super Economy, Economy ecc)
Essendo questa funzione poco pratica come API verrà documentata al minimo.
```
POST https://www.lefrecce.it/Channels.Website.BFF.WEB/website/grid/<cartId>/update
```
Payload (JSON):
```json
{
   "solutionId":"x4fb9e730-b02f-4be8-b605-30b6fa7d8ff5",
   "offeredServicesUpdate":[
      {
         "offerKeys":[
            "x931ef873-266d-4d45-b7cd-80aa3d55e225"
         ],
         "travellers":[
            {
               "id":"xe872c499-819c-4492-ba51-86c368368e30",
               "firstName":null,
               "lastName":null,
               "loyaltyCode":null,
               "customerKey":null,
               "parameters":[
                  
               ],
               "loyaltyPoints":null,
               "regionalLoyaltyPoints":null,
               "showCFBalanceLink":false,
               "showREGBalanceLink":false,
               "title":null,
               "tags":[
                  
               ],
               "adult":true,
               "loyaltyGiftCardBeneficiary":null
            }
         ]
      }
   ]
}
```
Risposta (JSON):
```json
{
   "grids":[
      {
         "id":"x994e26a5-76a4-4100-83d9-210239ff6e13",
         "summaries":[
            {
               "name":"Intercity 500",
               "description":"Genova Brignole (06:48) - Torino Porta Nuova (08:45)",
               "duration":"1h 57min",
               "highlightedMessage":null,
               "urban":false,
               "vehicleInfo":null,
               "departureLocationName":"Genova Brignole",
               "arrivalLocationName":"Torino Porta Nuova",
               "departureTime":"2024-10-19T06:48:00.000+02:00",
               "arrivalTime":"2024-10-19T08:45:00.000+02:00",
               "trainInfo":{
                  "description":"500",
                  "trainCategory":"Intercity",
                  "acronym":"IC",
                  "denomination":"Intercity",
                  "name":"500",
                  "logoId":"IC",
                  "urban":false
               },
               "bdoOrigin":"S04702",
               "showInfomobilityLink":false
            }
         ],
         "selectedOfferId":1,
         "selectedServiceId":158,
         "services":[
            {
               "id":170,
               "name":"2ª CLASSE EASY",
               "shortName":"EASY",
               "groupName":"2ª CLASSE",
               "description":"Poltrona spaziosa (larghezza sedile 51cm, spazio tra poltrone 95cm), ampi spazi per i bagagli, tavolini apribili al posto, prese elettriche, luce di cortesia, ganci appendiabiti, pulitore viaggiante presente durante l’intero viaggio.",
               "descriptionStandard":"",
               "offers":[
                  {
                     "offerId":1,
                     "serviceId":170,
                     "availableServiceId":null,
                     "serviceName":"2ª CLASSE EASY",
                     "name":"BASE",
                     "description":"Biglietto con cambi prenotazione illimitati prima della partenza. Per le Frecce qualora  il  livello di prezzo  Base originariamente pagato non sia disponibile, è necessario richiedere il cambio biglietto e corrispondere la differenza di prezzo. Rimborso con trattenuta.",
                     "price":{
                        "currency":"€",
                        "amount":20.50,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":374,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":true,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "xb08f37a2-51de-4dfe-95f5-aee9954bbd40"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1100,
                     "serviceId":170,
                     "availableServiceId":null,
                     "serviceName":"2ª CLASSE EASY",
                     "name":"Economy",
                     "description":"Biglietto modificabile prima della partenza con integrazione e non rimborsabile. Soggetto a disponibilita' posti",
                     "price":{
                        "currency":"€",
                        "amount":18.90,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":74,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x708cb789-1b7d-4616-9821-03b8e3787e96"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1101,
                     "serviceId":170,
                     "availableServiceId":null,
                     "serviceName":"2ª CLASSE EASY",
                     "name":"Super Economy",
                     "description":"Biglietto non modificabile e non rimborsabile. Soggetto a disponibilita' posti",
                     "price":{
                        "currency":"€",
                        "amount":11.90,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":56,
                     "canModify":false,
                     "canChange":false,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x2effcc44-f741-4666-bf1d-fa2ca5232d52"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1814,
                     "serviceId":170,
                     "availableServiceId":null,
                     "serviceName":"2ª CLASSE EASY",
                     "name":"YOUNG",
                     "description":"Offerta riservata ai titolari CartaFreccia con età inferiore a 30 anni (il viaggiatore deve corrispondere all'intestatario della carta). Il cambio della sola data/ora è consentito con integrazione a prezzo Base. No rimborso. Soggetta a disponibilità posti.",
                     "price":{
                        "currency":"€",
                        "amount":8.50,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":63,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":false,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":{
                        "imageId":"info",
                        "message":"Per l'offerta selezionata è necessario inserire un codice Cartafreccia.",
                        "status":"WARNING"
                     },
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              {
                                 "typeId":13,
                                 "name":"Cartafreccia",
                                 "displayName":"Carta<i>FRECCIA</i>/X-GO",
                                 "value":null,
                                 "allowedValues":[
                                    
                                 ],
                                 "required":true,
                                 "readOnly":false,
                                 "visible":true,
                                 "validationPattern":"[0-9]{9,9}",
                                 "inputPattern":"[0-9]",
                                 "minLength":9,
                                 "maxLength":9,
                                 "maxValue":null,
                                 "minValue":null,
                                 "type":"TRAVELLER",
                                 "flowType":"SELECTION",
                                 "identity":"INT",
                                 "errorMessage":null,
                                 "info":null,
                                 "valid":null
                              }
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x4b92c261-083f-443e-973e-85d1e6fbb7a3"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1549,
                     "serviceId":170,
                     "availableServiceId":null,
                     "serviceName":"2ª CLASSE EASY",
                     "name":"SENIOR",
                     "description":"Offerta riservata ai titolari CartaFreccia con età pari o superiore a 60 anni (il viaggiatore deve corrispondere all'intestatario della carta). Il cambio della sola data/ora è consentito con integrazione a prezzo Base. No rimborso. Soggetta a disponibilità posti.",
                     "price":{
                        "currency":"€",
                        "amount":10.30,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":11,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":false,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":{
                        "imageId":"info",
                        "message":"Per l'offerta selezionata è necessario inserire un codice Cartafreccia.",
                        "status":"WARNING"
                     },
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              {
                                 "typeId":13,
                                 "name":"Cartafreccia",
                                 "displayName":"Carta<i>FRECCIA</i>/X-GO",
                                 "value":null,
                                 "allowedValues":[
                                    
                                 ],
                                 "required":true,
                                 "readOnly":false,
                                 "visible":true,
                                 "validationPattern":"[0-9]{9,9}",
                                 "inputPattern":"[0-9]",
                                 "minLength":9,
                                 "maxLength":9,
                                 "maxValue":null,
                                 "minValue":null,
                                 "type":"TRAVELLER",
                                 "flowType":"SELECTION",
                                 "identity":"INT",
                                 "errorMessage":null,
                                 "info":null,
                                 "valid":null
                              }
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x4ee7e7d9-9654-4165-9292-2dedee3caa32"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  }
               ],
               "minPrice":{
                  "currency":"€",
                  "amount":11.90,
                  "originalAmount":null,
                  "indicative":false
               },
               "bestOfferId":1101,
               "extendedName":null,
               "descriptionKey":"IC.170"
            },
            {
               "id":158,
               "name":"1ª CLASSE PLUS",
               "shortName":"PLUS",
               "groupName":"1ª CLASSE",
               "description":"Poltrona spaziosa (larghezza sedile 63 cm, spazio tra poltrone 103cm), ampi spazi per i bagagli, tavolini apribili al posto, prese elettriche, luce di cortesia, ganci appendiabiti, pulitore viaggiante presente durante l’intero viaggio.",
               "descriptionStandard":"",
               "offers":[
                  {
                     "offerId":1,
                     "serviceId":158,
                     "availableServiceId":null,
                     "serviceName":"1ª CLASSE PLUS",
                     "name":"BASE",
                     "description":"Biglietto con cambi prenotazione illimitati prima della partenza. Per le Frecce qualora  il  livello di prezzo  Base originariamente pagato non sia disponibile, è necessario richiedere il cambio biglietto e corrispondere la differenza di prezzo. Rimborso con trattenuta.",
                     "price":{
                        "currency":"€",
                        "amount":28.00,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":94,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":true,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x931ef873-266d-4d45-b7cd-80aa3d55e225"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":true,
                     "selecteable":true
                  },
                  {
                     "offerId":1100,
                     "serviceId":158,
                     "availableServiceId":null,
                     "serviceName":"1ª CLASSE PLUS",
                     "name":"Economy",
                     "description":"Biglietto modificabile prima della partenza con integrazione e non rimborsabile. Soggetto a disponibilita' posti",
                     "price":{
                        "currency":"€",
                        "amount":24.90,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":18,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "xf60a866d-4167-470f-9ce5-77902842d7b3"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1101,
                     "serviceId":158,
                     "availableServiceId":null,
                     "serviceName":"1ª CLASSE PLUS",
                     "name":"Super Economy",
                     "description":"Biglietto non modificabile e non rimborsabile. Soggetto a disponibilita' posti",
                     "price":{
                        "currency":"€",
                        "amount":16.90,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":12,
                     "canModify":false,
                     "canChange":false,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":true,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":null,
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x2ab5c6ce-0c29-4c49-a65f-34ef5f5f404c"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1814,
                     "serviceId":158,
                     "availableServiceId":null,
                     "serviceName":"1ª CLASSE PLUS",
                     "name":"YOUNG",
                     "description":"Offerta riservata ai titolari CartaFreccia con età inferiore a 30 anni (il viaggiatore deve corrispondere all'intestatario della carta). Il cambio della sola data/ora è consentito con integrazione a prezzo Base. No rimborso. Soggetta a disponibilità posti.",
                     "price":{
                        "currency":"€",
                        "amount":14.00,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":8,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":false,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":{
                        "imageId":"info",
                        "message":"Per l'offerta selezionata è necessario inserire un codice Cartafreccia.",
                        "status":"WARNING"
                     },
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              {
                                 "typeId":13,
                                 "name":"Cartafreccia",
                                 "displayName":"Carta<i>FRECCIA</i>/X-GO",
                                 "value":null,
                                 "allowedValues":[
                                    
                                 ],
                                 "required":true,
                                 "readOnly":false,
                                 "visible":true,
                                 "validationPattern":"[0-9]{9,9}",
                                 "inputPattern":"[0-9]",
                                 "minLength":9,
                                 "maxLength":9,
                                 "maxValue":null,
                                 "minValue":null,
                                 "type":"TRAVELLER",
                                 "flowType":"SELECTION",
                                 "identity":"INT",
                                 "errorMessage":null,
                                 "info":null,
                                 "valid":null
                              }
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "xf784a896-0a22-424f-ab0f-873d093715c9"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  },
                  {
                     "offerId":1549,
                     "serviceId":158,
                     "availableServiceId":null,
                     "serviceName":"1ª CLASSE PLUS",
                     "name":"SENIOR",
                     "description":"Offerta riservata ai titolari CartaFreccia con età pari o superiore a 60 anni (il viaggiatore deve corrispondere all'intestatario della carta). Il cambio della sola data/ora è consentito con integrazione a prezzo Base. No rimborso. Soggetta a disponibilità posti.",
                     "price":{
                        "currency":"€",
                        "amount":16.80,
                        "originalAmount":null,
                        "indicative":false
                     },
                     "discounts":[
                        
                     ],
                     "availableAmount":3,
                     "canModify":true,
                     "canChange":true,
                     "canRefund":false,
                     "isCartaFrecciaProgram":false,
                     "isXgoProgram":false,
                     "isInhibited":false,
                     "status":"SALEABLE",
                     "infoMessages":[
                        
                     ],
                     "reportItemMessage":{
                        "imageId":"info",
                        "message":"Per l'offerta selezionata è necessario inserire un codice Cartafreccia.",
                        "status":"WARNING"
                     },
                     "travellers":[
                        {
                           "id":"xe872c499-819c-4492-ba51-86c368368e30",
                           "firstName":null,
                           "lastName":null,
                           "loyaltyCode":null,
                           "customerKey":null,
                           "parameters":[
                              {
                                 "typeId":13,
                                 "name":"Cartafreccia",
                                 "displayName":"Carta<i>FRECCIA</i>/X-GO",
                                 "value":null,
                                 "allowedValues":[
                                    
                                 ],
                                 "required":true,
                                 "readOnly":false,
                                 "visible":true,
                                 "validationPattern":"[0-9]{9,9}",
                                 "inputPattern":"[0-9]",
                                 "minLength":9,
                                 "maxLength":9,
                                 "maxValue":null,
                                 "minValue":null,
                                 "type":"TRAVELLER",
                                 "flowType":"SELECTION",
                                 "identity":"INT",
                                 "errorMessage":null,
                                 "info":null,
                                 "valid":null
                              }
                           ],
                           "loyaltyPoints":null,
                           "regionalLoyaltyPoints":null,
                           "showCFBalanceLink":false,
                           "showREGBalanceLink":false,
                           "title":null,
                           "tags":[
                              
                           ],
                           "adult":true,
                           "loyaltyGiftCardBeneficiary":null
                        }
                     ],
                     "offerKeys":[
                        "x5048ed3e-0b37-4375-ac02-4c56a085957c"
                     ],
                     "forceEvaluation":false,
                     "paidSeatSelection":false,
                     "seatNotAssigned":false,
                     "additionalPrice":null,
                     "selected":false,
                     "selecteable":true
                  }
               ],
               "minPrice":{
                  "currency":"€",
                  "amount":16.90,
                  "originalAmount":null,
                  "indicative":false
               },
               "bestOfferId":1101,
               "extendedName":null,
               "descriptionKey":"IC.158"
            }
         ],
         "infoMessages":[
            
         ],
         "canShowSeatMap":true,
         "collapsedVisualization":true,
         "regional":false
      }
   ],
   "collapsedVisualization":false,
   "canShowSeatMap":true
}
```

## Maggiori informazioni riguardo una soluzione
Nel qual caso si volessero ottenere più informazioni sulla soluzione è possibile chiamare l'endpoint `stops` che non solo dice tutte le fermate (con orari annessi) ma anche i servizi a bordo ed eventuali messaggi.
```
GET https://www.lefrecce.it/Channels.Website.BFF.WEB/website/stops?cartId=<cartId>&solutionId=<solutionId>
```
Risposta (JSON):
```json
[
   {
      "summary":{
         "name":"Intercity 500",
         "description":"Genova Brignole (06:48) - Torino Porta Nuova (08:45)",
         "duration":"1h 57min",
         "highlightedMessage":null,
         "urban":false,
         "vehicleInfo":null,
         "departureLocationName":"Genova Brignole",
         "arrivalLocationName":"Torino Porta Nuova",
         "departureTime":"2024-10-19T06:48:00.000+02:00",
         "arrivalTime":"2024-10-19T08:45:00.000+02:00",
         "trainInfo":{
            "description":"500",
            "trainCategory":"Intercity",
            "acronym":"IC",
            "denomination":"Intercity",
            "name":"500",
            "logoId":"IC",
            "urban":false
         },
         "bdoOrigin":"S04702",
         "showInfomobilityLink":false
      },
      "stops":[
         {
            "location":{
               "id":830004702,
               "name":"Genova Brignole",
               "displayName":"Genova Brignole",
               "timezone":"",
               "multistation":false,
               "centroidId":null
            },
            "departureTime":"2024-10-19T06:48:00.000+02:00",
            "arrivalTime":null,
            "trainNumber":"500"
         },
         {
            "location":{
               "id":830004700,
               "name":"Genova Piazza Principe",
               "displayName":"Genova Piazza Principe",
               "timezone":"",
               "multistation":false,
               "centroidId":null
            },
            "departureTime":"2024-10-19T06:56:00.000+02:00",
            "arrivalTime":null,
            "trainNumber":"500"
         },
         {
            "location":{
               "id":830004203,
               "name":"Novi Ligure",
               "displayName":"Novi Ligure",
               "timezone":"",
               "multistation":false,
               "centroidId":null
            },
            "departureTime":"2024-10-19T07:34:00.000+02:00",
            "arrivalTime":"2024-10-19T07:32:00.000+02:00",
            "trainNumber":"500"
         },
         {
            "location":{
               "id":830000470,
               "name":"Alessandria",
               "displayName":"Alessandria",
               "timezone":"",
               "multistation":false,
               "centroidId":null
            },
            "departureTime":"2024-10-19T07:48:00.000+02:00",
            "arrivalTime":"2024-10-19T07:46:00.000+02:00",
            "trainNumber":"500"
         },
         {
            "location":{
               "id":830000462,
               "name":"Asti",
               "displayName":"Asti",
               "timezone":"",
               "multistation":false,
               "centroidId":null
            },
            "departureTime":"2024-10-19T08:07:00.000+02:00",
            "arrivalTime":"2024-10-19T08:05:00.000+02:00",
            "trainNumber":"500"
         },
         {
            "location":{
               "id":830000452,
               "name":"Torino Lingotto",
               "displayName":"Torino Lingotto",
               "timezone":"",
               "multistation":false,
               "centroidId":830000996
            },
            "departureTime":null,
            "arrivalTime":"2024-10-19T08:37:00.000+02:00",
            "trainNumber":"500"
         },
         {
            "location":{
               "id":830000219,
               "name":"Torino Porta Nuova",
               "displayName":"Torino Porta Nuova",
               "timezone":"",
               "multistation":false,
               "centroidId":830000996
            },
            "departureTime":null,
            "arrivalTime":"2024-10-19T08:45:00.000+02:00",
            "trainNumber":"500"
         }
      ],
      "services":[
         {
            "imageData":"*icona codificata in base64*",
            "description":"Servizio di 1ª e 2ª classe prenotabile"
         },
         {
            "imageData":"*icona codificata in base64*",
            "description":"Treno con carrozza dotata di posto attrezzato e bagno accessibile per passeggeri su sedia a ruote. Per le località abilitate al servizio consultare l'elenco riportato sul sito www.rfi.it"
         },
         {
            "imageData":"*icona codificata in base64*",
            "description":"Il servizio di trasporto biciclette è soggetto a limitazione. Per i dettagli consulta le Condizioni generali di Trasporto. Treno con servizio di trasporto biciclette al seguito del viaggiatore."
         },
         {
            "imageData":null,
            "description":""
         },
         {
            "imageData":null,
            "description":"Per viaggiare con treni Intercity e` previsto il pagamento di prezzi globali comprensivi dell`assegnazione del posto."
         },
         {
            "imageData":"*immagine codificata in base64, image/png o image/jpeg presumo*",
            "description":"Treno con distributore automatico di bevande e snack"
         }
      ]
   }
]
```

