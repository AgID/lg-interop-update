**state**: inWorking


[INTEGRITY_REST_02] Integrità del payload delle request REST in PDND
====================================================================

Il presente pattern, nel contesto di Piattaforma Digitale Nazionale Dati per l’interoperabilità (di cui al comma 2 dell'articolo 50-ter del CAD), aggiunge alla comunicazione tra fruitore ed erogatore a livello di messaggio:

- integrità del payload della request del fruitore.

Si adottano le indicazione riportate in RFC 7231. Considereremo sempre richieste e risposte complete, con i metodi standard definiti in RFC 7231#section-4.

Questo pattern non copre quindi Range Requests RFC 7233 o HTTP method PATCH che trasmette una rappresentazione parziale.

Il presente pattern DEVE essere applicato nel caso in cui fruitore ed erogatore risultano aderenti della Piattaforma Digitale Nazionale Dati per l’interoperabilità.

L'integrità del payload della request del fruitore garantita attraverso l'applicazione del presente pattern è riconosciuta nel perimentro degli aderenti alla Piattaforma Digitale Nazionale Dati per l’interoperabilità.

Descrizione
-----------

Il presente profilo propone l’utilizzo di:

-  semantica HTTP :rfc:`7231`;

-  Digest HTTP header :rfc:`3230` per l’integrità della rappresentazione
   della risorsa;

-  JSON Web Token (JWT) definita dall’ :rfc:`7519`;

-  JSON Web Signature (JWS) definita dall’ :rfc:`7515`.

Si assume che il trust tra fruitore ed erogatore è costruito per il tramite di Piattaforma Digitale Nazionale Dati per l’interoperabilità che rende disponibile il materiale crittografico, nello specifico la/e chiave/i pubblica dichiarata dal fruitore relativamente al client da esso utilizzato per invocare l'e-service dell'erogatore.

.. mermaid::

  sequenceDiagram

    activate Fruitore
	activate Erogatore
    Fruitore->>+Erogatore: 1. Request()
	Erogatore-->>Fruitore: 2. Response
    deactivate Erogatore
    deactivate Fruitore


*Figura 8 - Integrità del payload della request del fruitore*

Regole di processamento
-----------------------

La creazione ed il processamento dei JWT DEVE rispettare
le buone prassi di sicurezza indicate in :rfc:`8725`.

**A: Richiesta**

1. Il fruitore predispone il body del messaggio (ad esempio un oggetto
   JSON)

2. Il fruitore calcola il valore del Digest header dei representation
   data secondo le indicazioni in :rfc:`3230`

3. Il fruitore individua l’elenco degli HTTP Header da firmare, incluso
   Digest e se presenti :httpheader:`Content-Type` e HTTP header
   Content-Encoding

4. Il fruitore crea la struttura o la stringa da firmare in modo che
   includa gli http header da proteggere, i riferimenti temporali di
   validità della firma e degli estremi della comunicazione, ovvero:

   a. il JOSE Header con almeno i parameter:

      i.   alg con l’algoritmo di firma, vedi :rfc:`8725`

      ii.  typ uguale a JWT

      iii. kid uguale alla chiave pubblica, registrata su Piattaforma Digitale Nazionale Dati per l’interoperabilità, associata alla chiave privata utilizzata per la firma della request 

   b. i seguenti claim obbligatori:

      iv. i riferimenti temporali di emissione e scadenza: :code:`iat` , :code:`exp`. Se
          il flusso richiede di verificare l’istante di prima validità
          del token, si può usare il claim :code:`nbf`.

      v.  il riferimento all'e-service dell’erogatore in :code:`aud`;

   c. i seguenti claim, secondo la logica del servizio:

      vi.   :code:`sub`: oggetto (principal see :rfc:`3744#section-2`) dei claim
            contenuti nel jwt

      vii.  :code:`iss`: l'id del client utilizzato dal fruitore

      viii. :code:`jti`: identificativo del JWT, per evitare replay attack

   d. il claim signed_headers con gli header http da proteggere ed i
      rispettivi valori, ovvero:

      ix. :httpheader:`Digest`

      x.  :httpheader:`Content-Type`

      xi. :httpheader:`Content-Encoding`

5. il fruitore firma il token adottando la JWS Compact Serialization

6. il fruitore posiziona il JWS nell’header Agid-JWT-Signature

7. Il fruitore spedisce il messaggio all’erogatore.

**B: Risultato**

8.  L’erogatore decodifica il JWS presente in Agid-JWT-Signature header
    secondo le indicazioni contenute in :rfc:`7515#section-5.2`,
    le buone prassi indicate in :rfc:`8725`
    e valida i claim contenuti nel Jose Header, in particolare verifica:

    e. il contenuto dei claim :code:`iat` , :code:`exp`;

    f. la corrispondenza tra se stesso e il claim :code:`aud`;

    g. l’univocità del claim :code:`jti` se presente.

9.  L’erogatore recupera da Piattaforma Digitale Nazionale Dati per l’interoperabilità la chiave pubblica indicata dal fruitore nel claim kid dell'JOSE Header  

10. L’erogatore valida la firma verificando l’elemento Signature del JWS

11. L’erogatore verifica la corrispondenza tra i valori degli header
    passati nel messaggio e quelli presenti nel claim signed-header,
    Content-Type e Content-Encoding se presenti devono essere sempre
    firmati, come indicato nel punto A3

12. L’erogatore quindi verifica la corrispondenza tra Digest ed il
    payload body ricevuto

13. Se le azioni da 8 a 12 hanno avuto esito positivo, il messaggio
    viene elaborato e viene restituito il risultato del servizio
    richiamato.
	
Note:

-  Per gli algoritmi da utilizzare in alg e Digest si vedano
   le Linee Guida sulla sicurezza, emanate dall'Agenzia per l'Italia Digitale 
   ai sensi dell'articolo 71 del decreto legislativo 7 marzo 2005, n. 82 (Codice dell'Amministrazione Digitale).


Esempio
-------

Di seguito è riportato un tracciato del messaggio inoltrato dal fruitore
all’interfaccia di servizio dell’erogatore.

Richiesta HTTP con Digest e representation metadata

.. code-block:: http

   POST https://api.erogatore.example/rest/service/v1/hello/echo/ HTTP/1.1
   Accept: application/json
   Agid-JWT-Signature: eyJhbGciOiJSUzI1NiIsInR5c.vz8...
   Digest: SHA-256=cFfTOCesrWTLVzxn8fmHl4AcrUs40Lv5D275FmAZ96E=
   Content-Type: application/json
   
   {"testo": "Ciao mondo"}

Porzione JWS con campi protetti dalla firma

.. code-block:: python

   # *header*
   {
     "alg": "RS256",
     "typ": "JWT",
     "kid": "199d08d2-9971-4979-a78d-e6f7a544f296"
   }
   # *payload*
   
   {
     "aud": "https://api.erogatore.example/rest/service/v1/hello/echo"
     "iat": 1516239022,
     "nbf": 1516239022,
     "exp": 1516239024,
     "signed_headers": [
       {"digest": "SHA-256=cFfTOCesrWTLVzxn8fmHl4AcrUs40Lv5D275FmAZ96E="},
       {"content-type": "application/json"}
     ],
   }
