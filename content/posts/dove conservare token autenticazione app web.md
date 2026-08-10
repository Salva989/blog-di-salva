+++
title = "Dove conservare i token di autenticazione in un'app web"
date = "2026-07-11T09:43:00+02:00"
lastmod = "2026-08-10T18:00:00+02:00"
draft = false
description = "localStorage, memoria, cookie HttpOnly e Backend for Frontend: rischi e scelte pratiche per gestire una sessione web."
tags = ["autenticazione", "sicurezza web", "cookie", "OAuth"]
categories = ["Tecnologia"]
+++

La domanda sembra semplice: **dove bisogna conservare il token di autenticazione di un'applicazione web?** La risposta dipende dall'architettura, ma una regola aiuta subito: il browser non dovrebbe ricevere credenziali longeve quando può lavorare con un normale cookie di sessione.

Pensa al token come a una **chiave digitale** che dimostra al server che hai già effettuato l’accesso.

## 1. Perché `localStorage` è rischioso

Molti tutorial fanno così:

```javascript
localStorage.setItem("token", token);
```

È comodo perché il token rimane anche dopo aver ricaricato la pagina. Il problema è che **qualsiasi JavaScript eseguito nella pagina può leggerlo**.

Se l’applicazione subisce un attacco XSS, il codice dell’attaccante può copiare il token e inviarlo altrove. A quel punto l’attaccante può usare quella chiave dal proprio computer, anche dopo che l’utente ha chiuso il sito. Per questo [OWASP sconsiglia di salvare identificatori di sessione nel Web Storage](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#storage-apis).

## 2. Conservarlo solo in memoria non risolve tutto

Potresti conservare il token dentro una variabile JavaScript:

```javascript
let accessToken = "...";
```

Non rimane sul disco come un valore in `localStorage`, ma non è una difesa contro JavaScript ostile eseguito nella pagina:

- sparisce quando ricarichi la pagina;
    
- ogni scheda del browser ha una memoria diversa;
    
- un attaccante che controlla JavaScript potrebbe intercettarlo quando viene aggiunto alle richieste.
    

Quindi la memoria è utile soprattutto per access token brevi in architetture che li richiedono, non per credenziali permanenti.

## 3. La soluzione consigliata: cookie `HttpOnly`

Il server può consegnare al browser un cookie simile a questo:

```http
Set-Cookie: __Host-session=abc123; Path=/; Secure; HttpOnly; SameSite=Lax
```

Le proprietà significano:

- **HttpOnly:** JavaScript non può leggere il cookie;
    
- **Secure:** viene inviato solamente tramite HTTPS;
    
- **SameSite:** limita l’invio del cookie da siti esterni;
    
- **`__Host-`:** impedisce configurazioni meno sicure legate a domini e sottodomini.
    

Il browser invia automaticamente il cookie al server, ma il codice JavaScript non può leggerlo. Un attacco XSS potrebbe ancora effettuare operazioni mentre la pagina è aperta, ma non può estrarre direttamente il valore del cookie. La [Session Management Cheat Sheet di OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) approfondisce attributi e ciclo di vita della sessione.

## 4. Il nuovo problema: CSRF

Poiché i cookie vengono inviati automaticamente, un sito malevolo potrebbe tentare di convincere il browser a mandare una richiesta al tuo server.

Per questo l’articolo consiglia di aggiungere:

1. un **token CSRF** alle operazioni che modificano dati;
    
2. `SameSite=Lax` o `Strict`;
    
3. controlli server sugli header `Origin` e `Sec-Fetch-Site`;
    
4. nessuna operazione che modifica dati tramite richieste `GET`.
    

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
    

Con un JWT completamente autonomo, invece, il token potrebbe continuare a funzionare fino alla sua scadenza, a meno di introdurre comunque un controllo server-side per revocarlo.

## 6. Quando entrano in gioco OAuth e servizi esterni

Quando utilizzi Google, Microsoft, Auth0 o altri servizi OAuth, la soluzione più sicura presentata è il **Backend for Frontend**, abbreviato **BFF**.

Il funzionamento è:

```text
Browser → cookie HttpOnly → tuo backend BFF → token OAuth → servizio esterno
```

Il browser non vede mai il vero access token o refresh token. I token rimangono sul server, mentre il browser possiede solamente un cookie di sessione. Il BFF riceve le richieste dal frontend, aggiunge il token corretto e le inoltra alle API.

## In pratica

La regola pratica è:

```text
Applicazione normale:
sessione server-side + cookie HttpOnly, Secure e SameSite

Applicazione con OAuth o API esterne:
Backend for Frontend, con i token conservati esclusivamente sul server
```

Quindi **non conservare token importanti e longevi nel `localStorage`**. La soluzione più robusta non consiste nel trovare il posto perfetto nel browser: consiste, quando possibile, nel **non mettere affatto il vero token nel browser**.

Questo non rende l’applicazione invulnerabile: bisogna comunque proteggersi da XSS, CSRF e malware presenti sul computer dell’utente. L’obiettivo è ridurre ciò che un attaccante può fare quando una singola difesa fallisce.
