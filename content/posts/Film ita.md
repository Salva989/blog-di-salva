+++
title = "Film ita"
date = "2026-4-25T09:00:00+02:00"
draft = false
description = "
Come ho trasformato il mio AI Agent in un cercatore di film su YouTube (usando Playwright)."
tags = ["Agenti ai", "scraping, "youtube", "film completi ita"]
categories = ["Technology", "Culture"]
+++


Come ho trasformato il mio AI Agent in un cercatore di film su YouTube (usando Playwright)

Avete presente quando cercate qualcosa su YouTube e vorreste solo una bella lista pulita, magari in un file Markdown, senza dover scrollare all'infinito? Beh, ho deciso di farlo fare al mio agente, **Alfredo** (basato su OpenClaw), e non è stato tutto rose e fiori, ma il risultato è pazzesco.

Ecco come abbiamo fatto.

1. Il Kit di Sopravvivenza

Per prima cosa, ho fatto installare ad Alfredo una serie di "skill" fondamentali. Tra queste, la più importante è stata **playwright-scraper-skill**. Playwright è una libreria potentissima che permette all'agente di aprire un vero browser (Chromium), navigare come farebbe un umano e leggere il contenuto delle pagine.

2. L’ostacolo tecnico (e come l'abbiamo superato)

Appena abbiamo provato a lanciare lo scraping, ci siamo scontrati con la realtà: mancavano delle librerie di sistema sul server (cose come `libnspr4` e `libnss3`).  
Alfredo non poteva installarle da solo tramite Telegram per motivi di sicurezza, quindi mi ha passato il comando `sudo apt-get`. Un colpo di terminale, un "fatto" in chat, e siamo ripartiti.

3. Lo Script: dalla "pigrizia" ai 300 risultati

Inizialmente, lo script leggeva solo i primi risultati visibili. Troppo poco.  
Ho chiesto ad Alfredo di modificare lo script JavaScript (usando `playwright-simple.js`) per implementare lo **scroll automatico**.

L'obiettivo? **300 risultati**.  
Anche se YouTube a volte fa il timido e sembra volerne mostrare meno, abbiamo forzato la mano con un ciclo di scroll (`MAX_SCROLLS=50`) e siamo riusciti a estrarre una lista massiccia di film completi disponibili sulla piattaforma.

4. Il risultato finale: Markdown + Git

La cosa più bella? Non ho dovuto copiare e incollare nulla. Alfredo ha:

1. Effettuato lo scraping di `https://www.youtube.com/results?search_query=film+ita`.
2. Estratto titoli e URL.
3. Generato un file `youtube-film-ita-scraping.md` direttamente nel workspace.
4. **Fatto il commit su Git** in automatico (anche se abbiamo dovuto gestire un commit un po' "pesante" con i node_modules, ma ci lavoreremo!).

Conclusione

Avere un agente che non solo scrive codice, ma interagisce con il file system e naviga il web per te, cambia totalmente il modo di lavorare. Ora ho la mia lista di 300 film pronta in Markdown, ordinata e pulita.

**Prossimo step?** Pulire la cronologia Git e magari aggiungere un filtro per la durata dei video, così da scartare i trailer e tenere solo i film veri!