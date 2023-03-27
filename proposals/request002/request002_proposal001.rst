**state**: inWorking


[AUDIT_REST_01] Inoltro dati tracciati nel dominio del Fruitore REST 
=====================================================================

Il presente pattern aggiunge alla comunicazione tra fruitore ed erogatore 
a livello di messaggio:

-  la capacità del fruitore di inoltrare i dati tracciati nel proprio dominio richiesti dall'erogatore.

Si adottano le indicazione riportate in :rfc:`7231`. Considereremo sempre
richieste e risposte complete, con i metodi standard definiti in RFC
7231#section-4.

L'erogatore e il fruitore DEVONO utilizzare la Piattaforma Digitale 
Nazionale Dati per l’interoperabilità di cui al comma 2 dell'articolo 
50-ter del CAD per la costituzione del trust, tramite il materiale crittografico 
depositato applicando i profili di emissione dei voucher previsti dalla stessa.

La costituzione del trust tra fruitore ed erogatore PUÒ essere realizzata
al di fuori della Piattaforma Digitale Nazionale Dati per l’interoperabilità, attraverso l'utilizzo di materiale critografico basato su certificati X.509,
solo nel caso in cui il fruitore non possa accreditarsi alla stessa e comunque 
entro 12 mesi dal superamento di tale impedimento l'erogatore e fruitore devono aggiornare le modalità di costituzione del trust assicurando lo stesso per il tramite della Piattaforma Digitale Nazionale Dati per l’interoperabilità.


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

- "aud" il riferimento all'e-service dell’erogatore;

- "iss" il riferimento del fruitore o del client utilizzato per la richiesta dell'e-service;

e, nel caso di utilizzo Piattaforma Digitale Nazionale Dati interoperabilità per la costruzione del trust, DEVE assicurare il popolamaneto del seguente claim del JWT di audit:  

- "purposeId" l'id della finalità registrata dal fruitore sulla Piattaforma Digitale Nazionale Dati interoperabilità in relazione alla richiesta di fruizione dell'e-service.


Di seguito è descritta l'applicazione del presente pattern nei due scenari in cui il trust tra fruitore ed erogato è realizzato:

- per il tramite della Piattaforma Digitale Nazionale Dati interoperabilità (TRUST GESTITO DA PDND);

- al di fuori della Piattaforma Digitale Nazionale Dati interoperabilità (TRUST DIRETTO FRUITORE - EROGATORE).


TRUST GESTITO DA PDND
---------------------

L'erogatore e il fruitore DEVONO utilizzare la Piattaforma Digitale Nazionale Dati per 
l’interoperabilità di cui al comma 2 dell'articolo 50-ter del CAD per la costituzione del trust, 
nello specifico utilizzando i profili di emissione dei Voucher previsti per la Piattaforma Digitale Nazionale 
Dati per l’interoperabilità.

Per assicurare dell'inoltro dei dati tracciati nel dominio del fruitore all'erogatore:

- il fruitore DEVE predisporre la rappresentazione dei dati tracciati e firmare la stessa utilizzando la chiave privata associata alla chiave pubblica registrata sulla Piattaforma Digitale Nazionale Dati per l’interoperabilità per il client utilizzato (JWS di audit);

- il fruitore nella request all'erogatore deve includere nell'header Agid-JWT-TrackingEvidence la rappresentazione dei dati tracciati e firmati (JWS di audit);

- l'erogatore DEVE verificare la firma del JWS di audit ricevuto nell'header Agid-JWT-TrackingEvidence, utilizzando la chiave pubblica recuperata dalla Piattaforma Digitale Nazionale Dati per l’interoperabilità associata alla chiave privata utilizzata dal fruitore per la firma del JWS di audit.


Nell'attuazione dei precedenti passi il fruitore è responsabile della valorizzazione dei claim inclusi nel JWS di audit.


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

      iii. kid uguale all'identificativo della chiave pubblica, registrata su Piattaforma Digitale Nazionale Dati per l’interoperabilità, associata alla chiave privata utilizzata per la firma

   b. i seguenti claim obbligatori:

      iv. i riferimenti temporali di emissione e scadenza: :code:`iat` , :code:`exp`. Se il flusso richiede di verificare l’istante di prima validità del token, si può usare il claim :code:`nbf`.

      v.  il riferimento dell’erogatore in :code:`aud`;
      
      vi. l'id della finalità registrata dal fruitore su Piattaforma Digitale Nazionale Dati interoperabilità in :code:`purposeId`;
      
      vii. l'id del client utilizzato dal fruitore in :code:`iss`;
      
      viii. identificativo del JWS in :code:`jti`;
      
      ix. id della finalità registrata dal fruitore in :code:`purposeId`;
	   
   c. il claim concordati con l'erogatore;

2. il fruitore firma il token adottando la JWS Compact Serialization utilizzando la chiave privata associata alla chiave pubblica registrata sulla Piattaforma Digitale Nazionale Dati per l'interoperabilità per il client utilizzato per la richiesta;

3. il fruitore posiziona il JWS di audit nell’header Agid-JWT-TrackingEvidence. 

4. Il fruitore spedisce il messaggio all’erogatore.

**B: Risultato**

5.  L’erogatore decodifica il JWS di audit presente in Agid-JWT-TrackingEvidence header
    secondo le indicazioni contenute in :rfc:`7515#section-5.2`,
    le buone prassi indicate in :rfc:`8725`
    e valida i claim contenuti nel Jose Header, in particolare verifica:

      i. il contenuto dei claim :code:`iat` , :code:`exp`;
      
      ii. la corrispondenza tra se stesso e il claim :code:`aud`; 
          
 6. l’erogatore recupera la chiave pubblica del client del fruitore dalla Piattaforma Digitale Nazionale Dati per l'interoperabilità e valida la firma verificando il JWS di audit
    
7.  Se l'azioni 5 e 6 hanno avuto esito positivo, il messaggio viene elaborato e viene restituito il risultato dell'e-service richiamato

Note:

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
     "iat": 1516239022,     
     "nbf": 1516239022,
     "exp": 1516239024,
     "userID": "user293",
     "userLocation": "station012"     
   }

TRUST DIRETTO FRUITORE - EROGATORE
----------------------------------

L'erogatore e il fruitore DEVONO definire il trust per consentire lo scambio del materiale crittografico necessario per assicurare la firma del JSW di audit.

Per dare seguito all'inoltro dei dati tracciati nel dominio del fruitore all'erogatore:

- il fruitore DEVE predisporre la rappresentazione dei dati tracciati e firmare la stessa utilizzando il materiale crittografico scambiato nel trust definito (JWS di audit);

- il fruitore nella request all'erogatore deve includere nell'header Agid-JWT-TrackingEvidence la rappresentazione dei dati tracciati e firmati (JWS di audit);

- l'erogatore DEVE verificare la firma del JWS di audit ricevuto nell'header Agid-JWT-TrackingEvidence, utilizzando il materiale crittografico scambiato nel trust definito.


Nell'attuazione dei precedenti passi il fruitore è responsabile della valorizzazione dei claim inclusi nel JWS di audit.


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

      iii. una o più delle seguenti opzioni per referenziare il certificato X.509:
      
	   -  :code:`x5u` (X.509 URL)

           -  :code:`x5c` (X.509 Certificate Chain)

           -  :code:`x5t#S256` (X.509 Certificate SHA-256 Thumbprint)

   b. i seguenti claim obbligatori:

      iv. i riferimenti temporali di emissione e scadenza: :code:`iat` , :code:`exp`. Se
          il flusso richiede di verificare l’istante di prima validità
          del token, si può usare il claim :code:`nbf`.

      v.  il riferimento dell’erogatore in :code:`aud`;
    
      vi. il riferimento del fruitore in :code:`iss`;
      
      vii. identificativo del JWS in :code:`jti`;	  

   c. il claim concordati con l'erogatore;

2. il fruitore firma il token adottando la JWS Compact Serialization utilizzando il materiale crittografico scambiato nel trust definito;

3. il fruitore posiziona il JWS di audit nell’header Agid-JWT-TrackingEvidence. 

4. Il fruitore spedisce il messaggio all’erogatore.

**B: Risultato**

5.  L’erogatore decodifica il JWS di audit presente in Agid-JWT-TrackingEvidence header
    secondo le indicazioni contenute in :rfc:`7515#section-5.2`,
    le buone prassi indicate in :rfc:`8725`
    e valida i claim contenuti nel Jose Header, in particolare verifica:

    e. il contenuto dei claim :code:`iat` , :code:`exp`;

    f. la corrispondenza tra se stesso e il claim :code:`aud`;    
          
6. l’erogatore valida la firma verificando il JWS di audit con il materiale crittografico scambiato nel trust definito;
    
7.  Se l'azioni 5 e 6 hanno avuto esito positivo, il messaggio viene elaborato e viene restituito il risultato dell'e-service richiamato

Note:

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
     "x5c": [
        "MIICyzCCAbOgAwIBAgIEC..."
  ]
   }
   # *payload*
   
   {
     "aud": "https://api.erogatore.example/rest/service/v1/hello/echo"
     "iss": "be54418b-fa38-4060-bf11-eac2cc1a48ca",
     "iat": 1516239022,     
     "nbf": 1516239022,
     "exp": 1516239024,
     "userID": "user293",
     "userLocation": "station012"
   }

