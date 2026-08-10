+++
title = "Dal sito statico al backend: serverless, code e autoscaling"
date = "2026-06-16T09:36:00+02:00"
lastmod = "2026-08-10T18:00:00+02:00"
draft = false
description = "Come scegliere tra funzioni serverless, Cloud Run, gruppi di VM e Kubernetes senza complicare l'architettura prima del necessario."
tags = ["serverless", "autoscaling", "Cloud Run", "Google Cloud"]
categories = ["Tecnologia"]
+++

Un sito statico è semplice da pubblicare, veloce e poco costoso. Appena deve ricevere dati, proteggere una chiave o svolgere un lavoro in background, però, serve un componente eseguito sul server.

La domanda utile non è “qual è l'architettura più potente?”, ma **qual è il gradino più piccolo che risolve il problema attuale?**

## Che cosa aggiunge una funzione serverless

Una funzione serverless esegue codice in risposta a una richiesta o a un evento. È adatta a operazioni brevi e indipendenti:

- salvare la risposta di un modulo in un database;
- creare una sessione di pagamento senza esporre chiavi segrete nel browser;
- inviare un'email o ricevere un webhook;
- generare un URL temporaneo per caricare un file;
- chiamare un'API esterna con credenziali protette;
- avviare una piccola trasformazione di testo o immagini.

Il frontend continua a essere statico. La funzione diventa un confine controllato tra il browser e i servizi che non devono essere accessibili direttamente.

Una funzione non è però la scelta ideale per un processo molto lungo, un'applicazione che mantiene connessioni persistenti o un carico che richiede strumenti di sistema specifici. In quei casi conviene passare a un container o a un worker dedicato.

## Quando usare un container gestito

Se l'applicazione è già contenuta in un'immagine Docker, un servizio come [Cloud Run](https://cloud.google.com/run/docs/overview/what-is-cloud-run) offre un passaggio naturale: riceve traffico HTTP, avvia le istanze necessarie e può ridurle quando non servono.

È una buona soluzione per API, applicazioni web e processi che entrano nei limiti del servizio. Rispetto a una VM elimina gran parte della gestione del sistema operativo; rispetto a una singola funzione permette di distribuire un'applicazione completa con le sue dipendenze.

Il container deve comunque essere progettato per essere sostituibile. I dati persistenti vanno salvati in un database o in uno storage esterno, non nel filesystem locale dell'istanza.

## I lavori lunghi hanno bisogno di una coda

Per convertire molti file, generare PDF o elaborare documenti, inviare ogni richiesta direttamente a una VM è fragile. Se il processo si interrompe, non è chiaro quali lavori siano stati completati.

Una coda separa la ricezione dall'elaborazione:

```text
utente -> API -> coda -> worker -> storage/database
```

L'API registra il lavoro e risponde rapidamente. Uno o più worker prelevano i messaggi, elaborano i file e salvano il risultato. La coda può ritentare gli errori e mostrare quante attività sono ancora in attesa.

Su Google Cloud questo ruolo può essere svolto da [Pub/Sub](https://cloud.google.com/pubsub/docs/overview) o, per chiamate HTTP pianificate e controllate, da Cloud Tasks. La metrica più utile per scalare non è sempre la CPU: per un sistema a worker spesso conta di più il numero o l'età dei messaggi in coda.

## Quando servono vere macchine virtuali

Una VM rimane utile quando il carico richiede accesso completo al sistema operativo, software difficili da containerizzare, acceleratori particolari o processi molto lunghi.

Per replicare più VM si parte da un'immagine o, meglio, da uno script di provisioning ripetibile. Un [Managed Instance Group](https://cloud.google.com/compute/docs/instance-groups) crea istanze omogenee da un template e può aumentare o ridurre il loro numero in base a metriche definite.

Il processo eseguito sulla VM dovrebbe partire come servizio di sistema o container con una politica di riavvio. `tmux` è ottimo per una sessione diagnostica, ma non è un supervisore di produzione: dopo un riavvio o un errore serve qualcosa che riporti automaticamente il worker nello stato desiderato.

Un esempio minimale con `systemd`:

```ini
[Unit]
Description=Document worker
After=network-online.target

[Service]
User=worker
WorkingDirectory=/opt/document-worker
ExecStart=/opt/document-worker/.venv/bin/python worker.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## Kubernetes arriva dopo

Kubernetes diventa interessante quando bisogna coordinare molti servizi, distribuire versioni, gestire risorse diverse e scalare più componenti insieme. Su Google Cloud, GKE Autopilot riduce il lavoro sui nodi, ma non elimina la complessità del modello Kubernetes.

Per un'applicazione singola, Cloud Run è spesso sufficiente. Per un gruppo omogeneo di worker con esigenze di sistema, un Managed Instance Group può essere più diretto. Kubernetes ha senso quando il problema organizzativo esiste già, non come primo passo per anticiparlo.

## E un server in casa?

Proxmox risponde a una domanda diversa: come usare un computer fisico per ospitare più macchine virtuali o container nella propria rete. È utile per laboratorio, apprendimento e servizi personali, ma non sostituisce automaticamente un'infrastruttura cloud accessibile e ridondata.

Il percorso essenziale è:

1. installare Proxmox VE su una macchina dedicata;
2. assegnare al server un indirizzo stabile nella rete locale;
3. caricare l'immagine del sistema operativo;
4. creare una VM scegliendo CPU, memoria, disco e rete;
5. predisporre backup prima di affidarle dati importanti.

La virtualizzazione hardware deve essere abilitata nel firmware. Se il servizio deve essere raggiungibile da Internet, occorre poi progettare con attenzione accesso remoto, aggiornamenti, firewall e continuità elettrica.

## Una progressione ragionevole

```text
sito statico
    -> funzione serverless per una singola operazione
    -> container gestito per un'applicazione completa
    -> coda e worker per elaborazioni asincrone
    -> gruppo di VM per requisiti di sistema specifici
    -> Kubernetes quando più servizi richiedono orchestrazione
```

Ogni passaggio aggiunge capacità, ma anche costi operativi. L'architettura migliore non è quella con più componenti: è quella che rende il lavoro affidabile restando comprensibile a chi dovrà mantenerlo.
