# MKInfernalGate - Analisi Tecnica Completa

## Introduzione

MKInfernalGate è un generatore di contenitori HTML auto-decrittanti progettato per consentire la distribuzione sicura di contenuti protetti tramite password all'interno di un singolo file HTML autonomo.

L'obiettivo del progetto è combinare:

- Cifratura crittografica moderna lato client
- Portabilità assoluta (single-file deployment)
- Assenza di dipendenze esterne
- Compatibilità con browser moderni
- Interfaccia grafica immersiva
- Offuscamento del payload generato

Il sistema opera interamente nel browser utilizzando esclusivamente le API crittografiche native del Web Crypto Standard.

---

# Architettura Generale

L'applicazione è composta da due componenti principali:

## 1. Forge Engine

Responsabile della generazione del contenitore protetto.

Funzioni principali:

- acquisizione file originale
- derivazione della chiave crittografica
- cifratura AES-256-GCM
- costruzione del portale HTML finale
- applicazione del layer di offuscamento

---

## 2. Infernal Gate Runtime

Responsabile della decrittazione.

Funzioni principali:

- richiesta password
- derivazione della chiave
- validazione autenticità tramite GCM Tag
- decrittazione del payload
- rendering del contenuto originale

---

# Processo di Cifratura

## Acquisizione del contenuto

Il file selezionato viene letto integralmente in memoria:

```javascript
const plaintext = new Uint8Array(await file.arrayBuffer());
```

Tutti i byte originali vengono mantenuti senza alterazioni.

---

## Generazione Salt

Per ogni nuova cifratura viene creato un salt casuale da 128 bit:

```javascript
const salt = crypto.getRandomValues(new Uint8Array(16));
```

Scopo:

- prevenire rainbow table attacks
- garantire chiavi differenti anche con password identiche
- rendere ogni output univoco

---

## Generazione Nonce

Viene generato un IV (nonce) casuale da 96 bit:

```javascript
const nonce = crypto.getRandomValues(new Uint8Array(12));
```

AES-GCM richiede nonce univoci per mantenere le proprie proprietà di sicurezza.

---

# Derivazione della Chiave

La password non viene mai utilizzata direttamente.

Viene elaborata mediante:

```javascript
PBKDF2
```

con:

```javascript
SHA-256
```

e:

```javascript
1.000.000 iterazioni
```

Configurazione:

```javascript
{
  hash: "SHA-256",
  iterations: 1000000,
  salt: salt
}
```

Risultato:

```javascript
AES-256 Key
```

Questa fase aumenta significativamente il costo computazionale di eventuali tentativi di brute force.

---

# Cifratura

La cifratura utilizza:

```javascript
AES-GCM 256 bit
```

Modalità:

```javascript
AES-GCM
```

L'algoritmo produce:

- ciphertext
- authentication tag

Entrambi vengono incorporati nel portale finale.

---

# Proprietà di Sicurezza di AES-GCM

AES-GCM fornisce simultaneamente:

## Confidenzialità

Impedisce la lettura dei dati senza la chiave corretta.

## Integrità

Rileva modifiche accidentali o intenzionali.

## Autenticità

Garantisce che il contenuto non sia stato alterato.

Qualsiasi modifica a:

- ciphertext
- nonce
- tag
- salt

provoca il fallimento della decrittazione.

---

# Struttura del Portale Generato

Il file finale contiene:

```text
Salt
Nonce
Ciphertext
Authentication Tag
Motore di Decrittazione
Interfaccia Grafica
```

Tutto all'interno di un unico documento HTML.

Non sono richiesti:

- server
- backend
- database
- API esterne
- librerie di terze parti

---

# Tactical Camouflage Layer

Dopo la generazione del portale viene applicato un secondo livello di protezione.

---

## Obfuscation

L'intero documento HTML viene:

1. convertito in byte
2. sottoposto a XOR
3. convertito in Base64

Schema:

```text
HTML
 ↓
XOR
 ↓
Base64
 ↓
Loader HTML
```

Chiave XOR:

```javascript
0x13
```

---

## Obiettivo

Questo meccanismo NON costituisce una misura crittografica.

Serve invece a:

- nascondere il codice sorgente immediatamente leggibile
- ridurre l'ispezionabilità casuale
- ostacolare analisi superficiali
- mimetizzare il payload finale

Un analista esperto può comunque ricostruire il contenuto.

La protezione reale rimane AES-256-GCM.

---

# Runtime di Decrittazione

Quando il file viene aperto:

1. viene richiesta la password
2. PBKDF2 rigenera la chiave
3. viene tentata la decrittazione
4. il contenuto viene caricato in memoria

Solo una password corretta produce un payload valido.

---

# Sistema Anti-Bruteforce

Il runtime implementa:

```javascript
MAX_ATTEMPTS = 7
```

Dopo vari errori:

- vengono introdotti cooldown temporanei
- l'interfaccia viene bloccata
- viene impedita la prosecuzione dei tentativi

Limitazione importante:

Essendo una webapp client-side, questo sistema è efficace principalmente contro l'utilizzo normale dell'interfaccia.

Non impedisce analisi offline del file da parte di un attaccante motivato.

La vera resistenza al brute force deriva dalla robustezza della password e dal costo di PBKDF2.

---

# Contromisure Anti-Tampering

Il runtime verifica:

## Disponibilità WebCrypto

```javascript
crypto.subtle
```

## Integrità delle API native

```javascript
[native code]
```

## Condizioni di debug basilari

Vengono effettuati controlli temporali per rilevare rallentamenti anomali associabili a debugging.

In caso di anomalia:

```javascript
sd()
```

provvede alla terminazione immediata della sessione.

---

# Sicurezza Reale

Le principali garanzie offerte dal sistema derivano da:

- AES-256-GCM
- PBKDF2-SHA256
- Salt casuale
- Nonce casuale
- Authentication Tag
- Web Crypto API native

L'offuscamento XOR e le protezioni anti-debug costituiscono invece meccanismi di deterrenza e complicazione dell'analisi.

---

# Modello di Minaccia

MKInfernalGate protegge efficacemente contro:

- accessi casuali
- divulgazione involontaria del contenuto
- manomissioni del payload
- ispezioni superficiali
- tentativi manuali di accesso

Non è progettato per contrastare:

- reverse engineering avanzato
- analisi dinamica professionale
- attacchi mirati su endpoint compromessi
- acquisizione del contenuto dopo la decrittazione

Come qualsiasi sistema client-side, una volta che il contenuto viene decrittato e visualizzato, esso è necessariamente disponibile all'utente autorizzato.

---

# Conclusione

MKInfernalGate implementa un modello di distribuzione sicura basato su crittografia moderna lato browser, racchiudendo dati cifrati e motore di decrittazione in un singolo file HTML autosufficiente.

L'utilizzo di AES-256-GCM combinato con PBKDF2-SHA256 a 1.000.000 iterazioni fornisce un livello di protezione elevato contro tentativi di accesso non autorizzati, mentre il sistema di offuscamento e le contromisure anti-tampering aggiungono ulteriori livelli di complessità all'analisi del contenitore generato.
