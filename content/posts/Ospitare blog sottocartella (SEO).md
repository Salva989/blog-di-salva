

+++
title = "Matcha Tea: Why I Love It"
date = "2025-10-18T16:20:00+02:00"
draft = false
description = "A short personal note on why matcha tea is special to me — flavor, ritual, and benefits."
tags = ["matcha", "tea", "personal"]
categories = ["Food & Drink"]
+++


````
# Come ospitare un blog su una sottodirectory invece di un sottodominio con Cloudflare Workers

Ospitare il proprio blog su una sottodirectory (es. `esempio.it/blog`) invece che su un sottodominio (es. `blog.esempio.it`) può migliorare significativamente la SEO e l'esperienza utente. Questa guida spiega passo dopo passo come configurare questa struttura utilizzando Cloudflare Workers.

## Introduzione

### Perché scegliere una sottodirectory?
I vantaggi sono principalmente legati alla SEO:
* **Autorità consolidata:** Ospitare il blog in una sottodirectory aiuta a consolidare l'autorità del dominio principale.
* **Prestazioni nei motori di ricerca:** Nonostante Google affermi di trattare i sottodomini allo stesso modo, i dati empirici suggeriscono che le sottodirectory tendono a performare meglio nelle classifiche.
* **Aumento del traffico:** L'autore riporta un aumento del traffico organico poche settimane dopo il passaggio da sottodominio a sottodirectory.

### Svantaggi
* **Complessità tecnica:** Molte piattaforme CMS sono progettate per i sottodomini. Configurare una sottodirectory richiede tempo e precisione.

---

## Passaggi per la configurazione

Immaginiamo di voler spostare un blog da `blog.esempio.it` a `esempio.it/blog`.

### 1. Configurazione DNS per il sito principale
Nel pannello Cloudflare:
1. Vai su **SSL/TLS** > **Panoramica** e imposta la modalità su **"Completa" (Full)**.
2. Vai in **DNS** > **Record** e aggiungi i record CNAME per il tuo sito principale (es. puntando a Render o Vercel).
3. Assicurati che lo stato del Proxy sia impostato su **"Proxied"** (nuvoletta arancione).

### 2. Configurazione DNS per il Blog
Aggiungi un record CNAME per il blog nel pannello DNS di Cloudflare:
* **Tipo:** CNAME
* **Nome:** blog
* **Target:** L'indirizzo fornito dal tuo hosting (es. `://vercel-dns.com`)
* **Stato Proxy:** Proxied

### 3. Configura il Blog Next.js
Modifica il file `next.config.js` (o `.mjs`) aggiungendo il `basePath`:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  basePath: "/blog",
  // ...altre configurazioni
};
export default nextConfig;
````