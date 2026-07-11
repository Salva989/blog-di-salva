L’articolo cerca di rispondere a una domanda: **dove bisogna conservare il token di autenticazione di un’applicazione web?**

Pensa al token come a una **chiave digitale** che dimostra al server che hai già effettuato l’accesso.

## 1. Perché `localStorage` è rischioso

Molti tutorial fanno così:

```javascript
localStorage.setItem("token", token);
```

È comodo perché il token rimane anche dopo aver ricaricato la pagina. Il problema è che **qualsiasi JavaScript eseguito nella pagina può leggerlo**.

Se l’applicazione subisce un attacco XSS, il codice dell’attaccante può copiare il token e inviarlo altrove. A quel punto l’attaccante può usare quella chiave dal proprio computer, anche dopo che l’utente ha chiuso il sito. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))

## 2. Conservarlo solo in memoria non risolve tutto

Potresti conservare il token dentro una variabile JavaScript:

```javascript
let accessToken = "...";
```

È leggermente più difficile da rubare rispetto a `localStorage`, ma:

- sparisce quando ricarichi la pagina;
    
- ogni scheda del browser ha una memoria diversa;
    
- un attaccante che controlla JavaScript potrebbe intercettarlo quando viene aggiunto alle richieste.
    

Quindi è utile soprattutto per token che durano pochissimo, non per credenziali permanenti. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))

## 3. La soluzione consigliata: cookie `HttpOnly`

Il server può consegnare al browser un cookie simile a questo:

```http
Set-Cookie: __Host-session=abc123;
HttpOnly;
Secure;
SameSite=Lax;
Path=/
```

Le proprietà significano:

- **HttpOnly:** JavaScript non può leggere il cookie;
    
- **Secure:** viene inviato solamente tramite HTTPS;
    
- **SameSite:** limita l’invio del cookie da siti esterni;
    
- **`__Host-`:** impedisce configurazioni meno sicure legate a domini e sottodomini.
    

Il browser invia automaticamente il cookie al server, ma il codice JavaScript non può copiarlo. Un attacco XSS potrebbe ancora effettuare operazioni mentre la pagina è aperta, ma non può rubare facilmente la chiave e portarsela via. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))

## 4. Il nuovo problema: CSRF

Poiché i cookie vengono inviati automaticamente, un sito malevolo potrebbe tentare di convincere il browser a mandare una richiesta al tuo server.

Per questo l’articolo consiglia di aggiungere:

1. un **token CSRF** alle operazioni che modificano dati;
    
2. `SameSite=Lax` o `Strict`;
    
3. controlli server sugli header `Origin` e `Sec-Fetch-Site`;
    
4. nessuna operazione che modifica dati tramite richieste `GET`. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))
    

## 5. Per una normale applicazione, meglio sessioni server-side

Per un’applicazione con un frontend e un backend, l’autore preferisce questo sistema:

```text
Browser:
cookie con ID casuale → abc123

Server:
abc123 → utente Salvatore
```

Il cookie non contiene direttamente identità, ruolo o autorizzazioni. Contiene soltanto un numero casuale. Le informazioni vere restano sul server.

Questo rende semplice:

- disconnettere immediatamente l’utente;
    
- bloccare una sessione rubata;
    
- aggiornare ruoli e permessi;
    
- invalidare tutte le sessioni dopo un cambio password.
    

Con un JWT completamente autonomo, invece, il token potrebbe continuare a funzionare fino alla sua scadenza, a meno di introdurre comunque una lista server-side di token revocati. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))

## 6. Quando entrano in gioco OAuth e servizi esterni

Quando utilizzi Google, Microsoft, Auth0 o altri servizi OAuth, la soluzione più sicura presentata è il **Backend for Frontend**, abbreviato **BFF**.

Il funzionamento è:

```text
Browser → cookie HttpOnly → tuo backend BFF → token OAuth → servizio esterno
```

Il browser non vede mai il vero access token o refresh token. I token rimangono sul server, mentre il browser possiede solamente un cookie di sessione. Il BFF riceve le richieste dal frontend, aggiunge il token corretto e le inoltra alle API. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))

## In pratica

La conclusione dell’articolo è:

```text
Applicazione normale:
sessione server-side + cookie HttpOnly, Secure e SameSite

Applicazione con OAuth o API esterne:
Backend for Frontend, con i token conservati esclusivamente sul server
```

Quindi **non conservare token importanti e longevi nel `localStorage`**. La soluzione più sicura non consiste nel trovare il posto migliore nel browser: consiste, quando possibile, nel **non mettere affatto il vero token nel browser**. ([Neciu Dan](https://neciudan.dev/most-secure-way-to-store-auth-token "What's the best way to do authentication in modern applications — Neciu Dan"))

Questo non rende l’applicazione invulnerabile: bisogna comunque proteggersi da XSS, CSRF e malware presenti sul computer dell’utente. L’obiettivo è ridurre ciò che un attaccante può fare quando una singola difesa fallisce.