**state**: inWorking


[AUDIT_REST_02] Inoltro dati tracciati nel dominio del Fruitore REST con correlazione
=====================================================================================

Il presente pattern aggiunge alla comunicazione tra fruitore ed erogatore 
a livello di messaggio:

-  la capacità del fruitore di inoltrare i dati tracciati nel proprio dominio richiesti dall'erogatore;
- la correlazione tra gli strumenti di autenticazione e i dati tracciati nel proprio dominio e inoltrati dal fruitore.

Si adottano le indicazione riportate in :rfc:`7231`. Considereremo sempre
richieste e risposte complete, con i metodi standard definiti in RFC
7231#section-4.

L'erogatore e il fruitore DEVONO utilizzare la Piattaforma Digitale 
Nazionale Dati per l’interoperabilità di cui al comma 2 dell'articolo 
50-ter del CAD per la costituzione del trust, tramite il materiale crittografico 
depositato applicando i profili di emissione dei voucher previsti dalla stessa.


Descrizione
-----------

Il presente pattern declina l’utilizzo di:

-  JSON Web Token (JWT) definita dall’ :rfc:`7519`;

-  JSON Web Signature (JWS) definita dall’ :rfc:`7515`.

L'erogatore e il fruitore DEVONO concordare i dati tracciati dal fruitore nel proprio dominio richiesti dall'erogatore, individuando i claim da includere nel JWT di audit che, nel caso di utilizzo Piattaforma Digitale Nazionale Dati interoperabilità per la costruzione del trust, DEVONO essere debitamente descritti dall'erogatore nella documentazione allegata al relativo e-service pubblicato nel Catalogo API della Piattaforma Digitale Nazionale Dati interoperabilità.

Esempi di claim che POSSONO essere inclusi nel JWT di audit sono:

- userID, un identificativo univoco dell'utente interno al dominio del fruitore che ha determinato l'esigenza della request di accesso all'e-service dell'erogatore;

- userLocation, un identificativo univoco della postazione interna al dominio del fruitore da cui è avviata l'esigenza della request di accesso all'e-service dell'erogatore;

- LoA, livello di sicurezza o di garanzia adottato nel processo di autenticazione informatica nel dominio del fruitore.

Il fruitore DEVE sempre assicurare il popolamento dei seguenti claim del JWT di audit: 

- "dnonce" un numero casuale costituito da 13 cifre, al fine di aumentare l'entropia dello stesso;

- "aud" il riferimento all'e-service dell’erogatore;

- "iss" il riferimento del fruitore o del client utilizzato per la richiesta dell'e-service;

- "purposeId" l'id della finalità registrata dal fruitore sulla Piattaforma Digitale Nazionale Dati interoperabilità in relazione alla richiesta di fruizione dell'e-service.

L'erogatore e il fruitore DEVONO utilizzare la Piattaforma Digitale Nazionale Dati per 
l’interoperabilità di cui al comma 2 dell'articolo 50-ter del CAD per la costituzione del trust, 
nello specifico ai profili di emissione dei Voucher previsti per la Piattaforma Digitale Nazionale 
Dati per l’interoperabilità sono aggiunti i seguenti passi per garantire la non ripudiabilità del contenuto del JWT di audit: 

- il fruitore, applicando quanto indicato nelle specifiche tecniche della Piattaforma Digitale Nazionale Dati per l’interoperabilità, DEVE predisporre la rappresentazione opaca dei dati tracciati e firmati (digest del JWS di audit), o utilizzare una rappresentazione opaca dei dati tracciati e firmati ancora valida nel proprio dominio, ed inserirla nella Access Token Request alla Piattaforma Digitale Nazionale Dati per l’interoperabilità;

- la Piattaforma Digitale Nazionale Dati per l’interoperabilità DEVE inserire la rappresentazione opaca dei dati tracciati(digest del JWS di audit) nell'Access Token, ovvero il Voucher rilasciato al fruitore;

- il fruitore nella request all'erogatore deve includere nell'header Agid-JWT-TrackingEvidence la rappresentazione dei dati tracciati e firmati (JWS di audit);

- l'ergatore DEVE verificare la firma del JWS di audit ricevuto nell'header Agid-JWT-TrackingEvidence, utilizzando la chiave pubblica recuperata dalla Piattaforma Digitale Nazionale Dati per l’interoperabilità;

- l'erogatore DEVE calcolare il digest della rappresentazione dei dati tracciati e firmati (JWS di Audit) ricevuti nell’header Agid-JWT-TrackingEvidence e verificarne la corrispondenza con quanto presente nell'Access Token (digest JWS di Audit).


Nell'attuazione dei precedenti passi il fruitore è responsabile della:

- valorizzazione dei claim inclusi nel JWS di audit;

- opacizzazione dei dati tracciati inoltrata alla Piattaforma Digitale Nazionale Dati per l’interoperabilità.


.. mermaid::

  sequenceDiagram

    activate Fruitore
	activate Erogatore
    Fruitore->>+Erogatore: 1. Request()
	Erogatore-->>Fruitore: 2. Response
    deactivate Erogatore
    deactivate Fruitore


*Figura XX - Audit dati tracciati nel dominio del fruitore*

Regole di processamento
-----------------------

La creazione ed il processamento dei JWT DEVE rispettare
le buone prassi di sicurezza indicate in :rfc:`8725`.

**A: Richiesta**

1. Il fruitore predispone il JWS con i dati tracciati nel proprio dominio, ovvero:

   a. il JOSE Header con almeno i parameter:

      i.   alg con l’algoritmo di firma, vedi :rfc:`8725`

      ii.  typ uguale a JWT

      iii. kid uguale all'identificativo della chiave pubblica, registrata su Piattaforma Digitale Nazionale Dati per l’interoperabilità, associata alla chiave privata utilizzata per la firma della request

   b. i seguenti claim obbligatori:

      iv. i riferimenti temporali di emissione e scadenza: :code:`iat` , :code:`exp`. Se
          il flusso richiede di verificare l’istante di prima validità
          del token, si può usare il claim :code:`nbf`.

      v.  il riferimento dell’erogatore in :code:`aud`;
	  
	  vi. l'id della finalità registrata dal fruitore su Piattaforma Digitale Nazionale Dati interoperabilità in :code:`purposeId`;
	  
	  vii. l'id del client utilizzato dal fruitore in :code:`iss`;
	  
	  viii. identificativo del JWS in :code:`jti`;	  	  
	  
	  ix. id della finalità registrata dal fruitore in :code:`purposeId`;
	  
	  x. un numero casuale costituito da 13 cifre in :code:`dnonce`;
      

   c. il claim concordati con l'erogatore;

2. il fruitore firma il token adottando la JWS Compact Serialization utilizzando la chiave privata associta alla chiave pubblica registrata sulla Piattaforma Digitale Nazionale Dati per l'interoperabilità al client utilizzato per la richiesta

3. il fruitore calcola il digest del JWS di audit e lo aggiunge alla richiesta del Voucher secondo le modalità indicate nelle specifiche tecniche della Piattaforma Digitale Nazionale Dati per l’interoperabilità.

4. il fruitore posiziona il Voucher nell'header Autorization e il JWS di audit nell’header Agid-JWT-TrackingEvidence. 

5. Il fruitore spedisce il messaggio all’erogatore.

**B: Risultato**

6. L'erogatore verifica il Voucher secondo le modalità indicate nelle specifiche tecniche della Piattaforma Digitale Nazionale Dati per l’interoperabilità.

7.  L’erogatore decodifica il JWS di audit presente in Agid-JWT-TrackingEvidence header
    secondo le indicazioni contenute in :rfc:`7515#section-5.2`,
    le buone prassi indicate in :rfc:`8725`
    e valida i claim contenuti nel Jose Header, in particolare verifica:

    e. il contenuto dei claim :code:`iat` , :code:`exp`;

    f. la corrispondenza tra se stesso e il claim :code:`aud`;  
      
8.  l’erogatore verifica la corrispondenza del digest contenuto nel Voucher della Piattaforma Digitale Nazionale Dati per l'interoperabilità è il digest calcolato dal JWS di audit presente nell’header Agid-JWT-TrackingEvidence 

9. l’erogatore recupera la chiave pubblica del client del fruitore dalla Piattaforma Digitale Nazionale Dati per l'interoperabilità e valida la firma verificando l’elemento Signature del JWS di audit
    
10.  Se l'azioni 6 o 9 ha avuto esito positivo, il messaggio viene elaborato e viene restituito il risultato dell'e-service richiamato

Note:

-  I predenti passi 1, 2 e 3 sono realizzati dal fruitore nella solo nel caso in cui non disponga di un digest del JWS di audit ancora valido nel proprio dominio;
-  Per gli algoritmi da utilizzare in alg e Digest si vedano
   le Linee Guida sulla sicurezza, emanate dall'Agenzia per l'Italia Digitale 
   ai sensi dell'articolo 71 del decreto legislativo 7 marzo 2005, n. 82 (Codice dell'Amministrazione Digitale).

Esempio
-------

Di seguito è riportato un tracciato del messaggio inoltrato dal fruitore all’interfaccia di servizio dell’erogatore.
Richiesta HTTP con Digest e representation metadata

.. code-block:: http

   POST https://api.erogatore.example/rest/service/v1/hello/echo/ HTTP/1.1
   Accept: application/json
   Autorization: Bearer AftgSSDGciFEEOiJfsI1NfsdfsdfiIsInR5c.vfd5...
   Agid-JWT-TrackingEvidence: eyJhbGciOiJSUzI1NiIsInR5c.vz8...
   Digest: SHA-256=cFfTOCesrWTLVzxn8fmHl4AcrUs40Lv5D275FmAZ96E=
   Content-Type: application/json
   
   {"testo": "Ciao mondo"}

Porzione JWS con campi protetti dalla firma

.. code-block:: python

   # *header*
   {
     "alg": "ES256",
     "typ": "JWT",
     "kid": "199d08d2-9971-4979-a78d-e6f7a544f296"
   }
   # *payload*
   
   {
     "aud": "https://api.erogatore.example/rest/service/v1/hello/echo"
     "iss": "be54418b-fa38-4060-bf11-eac2cc1a48ca",
     "purposeId": "4a153b51-5d47-4db9-be7e-e73dbcae4bb9",
     "dnonce": 1234567890123,
     "iat": 1516239022,     
     "nbf": 1516239022,
     "exp": 1516239024,
     "userID": "user293",
     "userLocation": "station012"   
   }


