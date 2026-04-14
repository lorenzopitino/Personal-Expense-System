# Personal-Expense-System
Elaborato fondamenti di informatica

 1. Descrizione del Progetto

Il software è un sistema di gestione delle spese personali che permette di:

- Gestire categorie di spesa personalizzate.

- Registrare transazioni con data, importo e descrizione.

- Impostare un budget mensile per ogni categoria.

- Visualizzare report dettagliati (spese per categoria, analisi budget vs spese, elenco cronologico).

Il sistema è composto da un database relazionale (SQL) e un'interfaccia a riga di comando (Python).

 2. Requisiti per l'esecuzione
 
Software Necessario:
- Interprete: Python 3.8 o superiore.

- Database: PostgreSQL (attivo e funzionante).

Il programma utilizza le seguenti librerie:

psycopg2: Libreria esterna necessaria per la connessione tra Python e PostgreSQL.

datetime: Libreria standard di Python per la gestione delle date.

 3. Istruzioni per l'esecuzione

Prima di avviare il programma Python, è necessario preparare il database:

Accedere a PostgreSQL (es. tramite pgAdmin o terminale psql).

Creare un database chiamato postgres.

Eseguire il codice contenuto nel file (database.sql.txt) per creare le tabelle Categorie.

Apri il terminale in Python e installa la libreria per la connessione al database psycopg2

Per avviare l'applicazione, posizionati nella cartella del progetto ed esegui main.py dal terminale

Una volta avviato, segui le istruzioni a schermo del menu interattivo.
