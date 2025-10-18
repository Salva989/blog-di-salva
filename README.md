# Blog Facile

Un blog moderno e veloce creato con Hugo e il tema PaperMod.

## 🚀 Funzionalità

- **Veloce**: Generato staticamente con Hugo
- **Responsive**: Design che si adatta a tutti i dispositivi
- **SEO Ottimizzato**: Configurato per i motori di ricerca
- **Dark Mode**: Supporto per modalità scura
- **Social Sharing**: Pulsanti di condivisione social
- **Tempo di Lettura**: Stima del tempo di lettura per ogni post
- **Breadcrumbs**: Navigazione migliorata
- **Syntax Highlighting**: Evidenziazione della sintassi del codice

## 🛠️ Tecnologie

- [Hugo](https://gohugo.io/) - Generatore di siti statici
- [PaperMod](https://github.com/adityatelange/hugo-PaperMod) - Tema Hugo

## 🏃‍♂️ Avvio Rapido

### Prerequisiti

- Hugo Extended installato sul tuo sistema

### Installazione

1. Clona la repository:
```bash
git clone https://github.com/Salva989/blog-facile.git
cd blog-facile
```

2. Inizializza i submodule per il tema:
```bash
git submodule update --init --recursive
```

3. Avvia il server di sviluppo:
```bash
hugo server -D
```

4. Apri il browser su `http://localhost:1313`

## 📝 Creazione Contenuti

### Nuovo Post

```bash
hugo new content posts/nome-del-post.md
```

### Nuova Pagina

```bash
hugo new content nome-pagina.md
```

## 🔧 Configurazione

Le impostazioni principali si trovano nel file `hugo.toml`. Puoi modificare:

- Titolo del sito
- Descrizione
- Link social
- Menu di navigazione
- Impostazioni del tema

## 📦 Build di Produzione

```bash
hugo --minify
```

I file generati si troveranno nella cartella `public/`.

## 🤝 Contribuire

Le pull request sono benvenute! Per modifiche importanti, apri prima una issue per discutere cosa vorresti cambiare.

## 📄 Licenza

Questo progetto è sotto licenza MIT. Vedi il file `LICENSE` per i dettagli.