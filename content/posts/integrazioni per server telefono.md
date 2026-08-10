+++
title = "Trasformare un telefono Android in un piccolo server"
date = "2026-08-10T14:22:00+02:00"
lastmod = "2026-08-10T18:00:00+02:00"
draft = false
description = "Termux, Tailscale, Cloudflare Tunnel e avvio automatico: una configurazione progressiva per ospitare servizi su Android."
tags = ["Termux", "Android", "Cloudflare Tunnel", "Tailscale"]
categories = ["Tecnologia"]
+++

Un telefono Android inutilizzato può ospitare piccoli servizi personali senza imitare subito tutta la complessità di una VPS. La configurazione più comprensibile procede **a strati**:

```text
                    INTERNET
                       │
                 Cloudflare
                       │
                Cloudflare Tunnel
                       │
                  TELEFONO
                 ┌─────┴─────┐
                 │           │
             app :3000    app :8000
                 
PC ── Tailscale ──► telefono ──► SSH/Termux
```

Cloudflare Tunnel è adatto proprio a questo: `cloudflared` crea una connessione **in uscita** dal telefono verso Cloudflare, quindi non serve IP statico né port forwarding sul router. ([Cloudflare Docs](https://developers.cloudflare.com/tunnel/ "Cloudflare Tunnel"))

## 1. Preparare Termux

Se lo hai già puoi saltare l'installazione. Il progetto Termux raccomanda F-Droid o le release ufficiali GitHub; inoltre esistono direttamente nei repository Termux i pacchetti `openssh`, `cloudflared` e `caddy`. ([GitHub](https://github.com/termux/termux-app/blob/master/README.md "Termux app README"))
    

Dentro Termux:

```bash
pkg update
pkg upgrade

pkg install openssh cloudflared curl git
```

Per ora **non installerei nemmeno Caddy**.

Avvia SSH:

```bash
sshd
```

Controlla quale porta sta usando:

```bash
sshd -T | grep '^port '
```

e il tuo username Termux:

```bash
whoami
```

Per esempio potresti ottenere:

```text
port 8022
u0_a123
```

## 2. Collegarsi in privato con Tailscale

Installa Tailscale sul telefono e sul PC, quindi accedi con lo stesso account su entrambi. Tailscale supporta Android 8 o successivo. ([Tailscale](https://tailscale.com/docs/install/android "Install Tailscale on Android"))
    

Nell'app Tailscale del telefono vedrai un IP simile a:

```text
100.90.40.20
```

Dal computer potrai quindi entrare nel telefono:

```bash
ssh -p 8022 u0_a123@100.90.40.20
```

sostituendo porta, username e IP con i tuoi.

Questa è una distinzione importante: su Android userei **il normale `sshd` di Termux attraverso la rete Tailscale**, non la funzione "Tailscale SSH". Il server integrato Tailscale SSH non è disponibile nell'app Android; è disponibile su piattaforme server supportate come Linux e macOS CLI. ([Tailscale](https://tailscale.com/docs/features/tailscale-ssh "Tailscale SSH"))

Quindi hai già ottenuto:

```text
PC
 ↓
Tailscale
 ↓
Telefono
 ↓
Termux / SSH
```

senza aprire SSH su Internet.

## 3. Pubblicare un'app con Cloudflare Tunnel

Immaginiamo che sul telefono ci sia una app Node, FastAPI o Deno in ascolto su:
    

```text
127.0.0.1:3000
```

Per esempio:

```bash
curl http://127.0.0.1:3000
```

deve rispondere.

Poi:

```bash
cloudflared tunnel login
```

crea il tunnel:

```bash
cloudflared tunnel create telefono
```

e guarda l'ID:

```bash
cloudflared tunnel list
```

Supponiamo che il dominio sia:

```text
server.tuodominio.it
```

associalo al tunnel:

```bash
cloudflared tunnel route dns telefono server.tuodominio.it
```

Cloudflare supporta proprio la mappatura di un hostname pubblico verso un servizio locale come `localhost:3000`. ([Cloudflare Docs](https://developers.cloudflare.com/tunnel/ "Cloudflare Tunnel"))

Crea:

```bash
mkdir -p ~/.cloudflared
nano ~/.cloudflared/config.yml
```

con:

```yaml
tunnel: ID-DEL-TUNNEL
credentials-file: /data/data/com.termux/files/home/.cloudflared/ID-DEL-TUNNEL.json

ingress:
  - hostname: server.tuodominio.it
    service: http://127.0.0.1:3000

  - service: http_status:404
```

Poi:

```bash
cloudflared tunnel run telefono
```

Adesso:

```text
https://server.tuodominio.it
```

arriva a:

```text
Internet
 ↓
Cloudflare
 ↓
Tunnel
 ↓
telefono
 ↓
127.0.0.1:3000
```

e **non devi aprire la porta 3000 sul router**. ([Cloudflare Docs](https://developers.cloudflare.com/tunnel/ "Cloudflare Tunnel"))

## 4. Gestire più applicazioni

Con più applicazioni non serve ancora Caddy. Cloudflare Tunnel può già smistarle:
    

```yaml
ingress:
  - hostname: sito.tuodominio.it
    service: http://127.0.0.1:3000

  - hostname: api.tuodominio.it
    service: http://127.0.0.1:8000

  - hostname: admin.tuodominio.it
    service: http://127.0.0.1:5000

  - service: http_status:404
```

Quindi:

```text
sito.tuodominio.it
        ↓
     :3000

api.tuodominio.it
        ↓
     :8000

admin.tuodominio.it
        ↓
     :5000
```

Per il tuo caso questa è probabilmente la soluzione **più semplice**.

Caddy lo aggiungerei solo quando vuoi qualcosa di più strutturato. Caddy può fare da reverse proxy verso diversi backend locali. ([Caddy Web Server](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy "Caddy reverse proxy"))

In quel caso diventerebbe:

```text
Cloudflare
    ↓
cloudflared
    ↓
Caddy
    ├── :3000
    ├── :8000
    └── :5000
```

Termux ha già un pacchetto Caddy, quindi:

```bash
pkg install caddy
```

([GitHub](https://github.com/termux/termux-packages/blob/master/packages/caddy/build.sh "Caddy package for Termux"))

## 5. Avviare i servizi al riavvio

**Termux:Boot** è progettato per eseguire script all'avvio di Android. La documentazione ufficiale mostra anche l'avvio automatico di `sshd` e l'uso di `termux-wake-lock`. ([GitHub](https://github.com/termux/termux-boot/blob/master/README.md "Termux:Boot README"))
    

Dopo aver installato Termux:Boot, aprilo almeno una volta.

Poi:

```bash
mkdir -p ~/.termux/boot
nano ~/.termux/boot/server
```

puoi iniziare con:

```bash
#!/data/data/com.termux/files/usr/bin/sh

termux-wake-lock

sshd

cloudflared tunnel run telefono &
```

Rendilo eseguibile:

```bash
chmod +x ~/.termux/boot/server
```

E nelle impostazioni Android togli l'ottimizzazione batteria per Termux/Tailscale/Termux:Boot, perché Android può altrimenti terminare processi in background; anche il progetto Termux richiama esplicitamente questo problema. ([GitHub](https://github.com/termux/termux-app/blob/master/README.md "Termux app README"))

A questo punto hai già una macchina piuttosto seria:

```text
Accendo telefono
      ↓
Termux:Boot
      ↓
sshd + cloudflared
      ↓
┌──────────────────────┐
│ SSH tramite Tailscale│
│ Web tramite Cloudflare│
└──────────────────────┘
```

## 6. Rendere la configurazione ripetibile

Solo a questo punto aggiungerei Ansible. Sul telefono:
    

```bash
pkg install python
```

Sul tuo PC installi Ansible e crei per esempio:

```text
phone-server/
├── inventory.ini
├── phone.yml
├── files/
└── templates/
```

`inventory.ini`:

```ini
[phone]
android ansible_host=100.90.40.20 ansible_port=8022 ansible_user=u0_a123 ansible_python_interpreter=/data/data/com.termux/files/usr/bin/python
```

Poi:

```bash
ansible phone -i inventory.ini -m ping
```

Ansible utilizza normalmente SSH per gestire host remoti ed è proprio pensato per portare una macchina verso uno stato dichiarato nel playbook. ([Documentazione Ansible](https://docs.ansible.com/projects/ansible/latest/inventory_guide/connection_details.html "Connection methods and details"))

Dopo puoi creare:

```bash
ansible-playbook -i inventory.ini phone.yml
```

e far sì che `phone.yml` automaticamente crei directory, copi configurazioni, installi app, aggiorni file e riavvii solo ciò che è necessario.

**Io quindi farei questa progressione:**

```text
FASE 1
Termux
+ SSH
+ Tailscale
+ Cloudflare Tunnel

       ↓

FASE 2
Termux:Boot
+ avvio automatico
+ health check

       ↓

FASE 3
Caddy
solo se hai molti servizi

       ↓

FASE 4
Ansible
configurazione riproducibile

       ↓

FASE 5
release versionate
current -> release
rollback
secrets con Ansible Vault
```

Per il tuo telefono, **partirei concretamente da Fase 1 e 2**. È già sufficiente per avere qualcosa di molto simile a un VPS: puoi spostare il telefono da un Wi-Fi all'altro, Tailscale continua a darti accesso amministrativo e `cloudflared`, una volta ristabilita la connessione Internet, può ristabilire il tunnel verso Cloudflare. ([Cloudflare Docs](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/ "Cloudflare Tunnel"))

Il passo successivo è raccogliere configurazione e controlli in una cartella versionata con `Makefile`, Ansible, health check e script di avvio, così da ottenere comandi ripetibili come:

```bash
make phone
make phone-status
make phone-edge-check
```

In questo modo il telefono resta un piccolo server sperimentale, ma la sua configurazione non dipende più dalla memoria di chi lo ha preparato.
