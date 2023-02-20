**state**: inWorking


[AUDIT_REST_01] Inoltro dati tracciati nel dominio del Fruitore REST
====================================================================

Il presente pattern aggiunge alla comunicazione tra fruitore ed erogatore 
a livello di messaggio:

-  la capacità del fruitore di inoltrare i dati tracciati nel proprio dominio richiesti dall'erogatore.

Si adottano le indicazione riportate in :rfc:`7231`. Considereremo sempre
richieste e risposte complete, con i metodi standard definiti in RFC
7231#section-4.


Descrizione
-----------

Il presente pattern declina l’utilizzo di:

-  JSON Web Token (JWT) definita dall’ :rfc:`7519`;

-  JSON Web Signature (JWS) definita dall’ :rfc:`7515`.

L'erogatore e il fruitore DEVONO concordare i dati tracciati dal fruitore nel proprio dominio richiesti dall'erogatore, individuando i claim da includere nel JWT che DEVONO essere debitamente descritti dall'erogatore nella documentazione allegata al relativo e-service pubblicato nel Catalogo API della Piattaforma Digitale Nazionale Dati.

Esempi di claim che POSSONO essere inclusi nel JWT sono:

- userID, un identificativo univoco dell'utente interno al dominio del fruitore che ha determinato l'esigenza della request di accesso all'e-service dell'erogatore;

- userLocation, un identificativo univoco della postazione interna al dominio del fruitore da cui è avviata l'esigenza della request di accesso all'e-service dell'erogatore;

- LoA, livello di sicurezza o di garanzia adottato nel processo di autenticazione informatica.


L'erogatore e il fruitore DEVONO utilizzare la Piattaforma Digitale Nazionale Dati per 
l’interoperabilità di cui al comma 2 dell'articolo 50-ter del CAD per la costituzione del trust, 
nello specifico ai profili di emissione dei Voucher previsti per la Piattaforma Digitale Nazionale 
Dati per l’interoperabilità sono aggiunti i seguenti passi per garantire la non ripudiabilità del contenuto del JWT: 

- il fruitore DEVE predisporre la rappresentazione opaca dei dati tracciati (hash del JWT) ed inserirli nella Access Token Request alla Piattaforma Digitale Nazionale Dati per l’interoperabilità;

- la Piattaforma Digitale Nazionale Dati per l’interoperabilità DEVE inserire la rappresentazione opaca dei dati tracciati nell'Access Token prodotto;

- il fruitore DEVE calcolare la hash del JWT ricevuto nell’header Agid-JWT-TrackingEvidence e verificarne la corrispondenza con quanto presente nell'Access Token.


La costituzione del trust tra fruitore ed erogatore PUÒ essere realizzata
al di fuori della Piattaforma Digitale Nazionale Dati per l’interoperabilità
nel solo caso in cui il fruitore non possa accreditarsi alla stessa e comunque 
entro 12 mesi dal superamento di tale impedimento l'erogatore e fruitore devono
aggiornare l'e-service assicurando la costituzione del trust tramite la Piattaforma 
Digitale Nazionale Dati per l’interoperabilità.

In quanto segue si declina il presente pattern, ricordando che in assenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità, per garantire la non ripudiabilità del contenuto del JWT, il fruitore deve applicare la specifica JWS allo stesso.


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

1. Il fruitore predispone il JWT con i dati tracciati nel proprio dominio, ovvero:

   a. il JOSE Header con almeno i parameter:

      i.   alg con l’algoritmo di firma, vedi :rfc:`8725` (solo in assenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità)

      ii.  typ uguale a JWT

      iii. una o più delle seguenti opzioni per referenziare il
           certificato X.509 (solo in assenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità):

           -  :code:`x5u` (X.509 URL)

           -  :code:`x5c` (X.509 Certificate Chain)

           -  :code:`x5t#S256` (X.509 Certificate SHA-256 Thumbprint)

   b. i seguenti claim obbligatori:

      iv. i riferimenti temporali di emissione e scadenza: :code:`iat` , :code:`exp`. Se
          il flusso richiede di verificare l’istante di prima validità
          del token, si può usare il claim :code:`nbf`.

      v.  il riferimento dell’erogatore in :code:`aud`;

   c. i seguenti claim, secondo la logica del servizio:

      vi.   :code:`sub`: oggetto (principal see :rfc:`3744#section-2`) dei claim
            contenuti nel jwt

      vii.  :code:`iss`: identificativo del mittente

      viii. :code:`jti`: identificativo del JWT, per evitare replay attack

   d. il claim concordati con l'erogatore

2. il fruitore firma il token adottando la JWS Compact Serialization (solo in assenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità)

3. il fruitore posiziona il token nell’header Agid-JWT-TrackingEvidence

4. Il fruitore spedisce il messaggio all’erogatore.

**B: Risultato**

5.  L’erogatore decodifica il token presente in Agid-JWT-TrackingEvidence header
    secondo le indicazioni contenute in :rfc:`7515#section-5.2`,
    le buone prassi indicate in :rfc:`8725`
    e valida i claim contenuti nel Jose Header, in particolare verifica:

    e. il contenuto dei claim :code:`iat` , :code:`exp`;

    f. la corrispondenza tra se stesso e il claim :code:`aud`;

    g. l’univocità del claim :code:`jti` se presente.

6.  In presenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità, l’erogatore verifica la corrispondenza dell’hash contenuto nel voucher PDND è l’hash del token nell’header Agid-JWT-TrackingEvidence 

7. In assenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità, l’erogatore:
    a.	recupera il certificato X.509 referenziato nel JOSE Header facendo attenzione alle indicazioni contenute in :rfc:`8725#section-3.10`
    
    b. verifica il certificato secondo i criteri del trust
    
    c. valida la firma verificando l’elemento Signature del JWS
    
8.  Se le azioni da 6 o 7 ha avuto esito positivo, il messaggio viene elaborato e viene restituito il risultato del servizio richiamato

Note:

-  Per gli algoritmi da utilizzare in alg e Digest si vedano
   le Linee Guida sulla sicurezza, emanate dall'Agenzia per l'Italia Digitale 
   ai sensi dell'articolo 71 del decreto legislativo 7 marzo 2005, n. 82 (Codice dell'Amministrazione Digitale).

Esempio
-------

Di seguito è riportato un tracciato del messaggio inoltrato dal fruitore all’interfaccia di servizio dell’erogatore, in assenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità.

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
     "iat": 1516239022,
     "nbf": 1516239022,
     "exp": 1516239024,
     "userID": "user293",
     "userLocation": "station012"
   }

Le parti, in base alle proprie esigenze, individuano gli specifici algoritmi
secondo quanto indicato nelle Linee Guida sulla sicurezza,
emanate dall'Agenzia per l'Italia Digitale ai sensi dell'articolo 71
del decreto legislativo 7 marzo 2005, n. 82 (Codice dell'Amministrazione Digitale).

