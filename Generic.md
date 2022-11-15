# Bestialità da evitare in ogni progetto software

- [Bestialità da evitare in ogni progetto software](#bestialità-da-evitare-in-ogni-progetto-software)
  - [1. Lavorare senza VCS](#1-lavorare-senza-vcs)
  - [2. Non avere un ambiente di test](#2-non-avere-un-ambiente-di-test)
  - [3. Usare il VCS in maniera impropria](#3-usare-il-vcs-in-maniera-impropria)
    - [3a. Fare i controdeploy da produzione a sviluppo](#3a-fare-i-controdeploy-da-produzione-a-sviluppo)
    - [3b. Usare repo diversi per test e produzione](#3b-usare-repo-diversi-per-test-e-produzione)
  - [4. Non avere un sistema di deploy](#4-non-avere-un-sistema-di-deploy)

## 1. Lavorare senza VCS

Iniziamo con l'ovvio: usate un Version Control System. Anzi, usate git.

Vi piace Mercurial? Usatelo. Vi piace SVN? Usate git. 😄

## 2. Non avere un ambiente di test

A meno che non stiate scrivendo un programma minimale, cercate di avere almeno questi tre ambienti:

- **Produzione**: il sistema consolidato e accessibile ai vostri utenti.
- **Test**: le modifiche al codice che vorremmo portare in produzione; la base dati non è necessariamente allineata con la produzione.
- **Sviluppo**: tutte le modifiche anche distruttive al codice e alla base dati.

Se siete diligenti potete aggiungere anche un quarto ambiente, ossia:

- **Staging**: una copia carbone della produzione (codice e base dati), tranne le ultimissime modifiche al codice e alla base dati. Serve per avere una copia fedele della produzione in modo da scovare anche i bug più insidiosi.

## 3. Usare il VCS in maniera impropria

### 3a. Fare i controdeploy da produzione a sviluppo

Continuiamo con le ovvietà: il percorso corretto (o meno contorto) per apportare modifiche è: ambiente di sviluppo => test => produzione. Modificare file direttamente in produzione è garanzia di rotture, prima o poi.

Git serve per salvare le modifiche rilevanti della codebase in un *ambiente di sviluppo*; non dovrebbe servire a portare sui PC degli sviluppatori le modifiche realizzate in produzione.

### 3b. Usare repo diversi per test e produzione

I branch servono per avere più versioni della **stessa** codebase, solitamente la versione di sviluppo e quella produzione. Non dovrebbero esistere **due** repository ad es. per la versione di sviluppo e quella di produzione.

Le modifiche alla codebase possono incasinarla, e anche per questo esistono i branch.

Se avete dubbi adottate il git-flow: <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>

## 4. Non avere un sistema di deploy

Fare i deploy copiando a mano i file modificati è un ottimo modo per causare problemi all'ambiente di produzione.

Nel caso tipo,  al deploy ci si dimentica di uploadare un file modificato tre settimane fa e dopo un paio d'ore il cliente ci telefona indispettito.
