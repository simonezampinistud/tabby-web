# Revisioni della sincronizzazione

Il campo `content` viene conservato integralmente, compresi gli spazi e le
righe finali significativi nei blocchi YAML. Non viene interpretato dal server:
restano ammessi documenti cifrati e campi sconosciuti.

PUT/PATCH possono includere `X-Tabby-Expected-Revision` con il valore esatto
di `modified_at` restituito da GET. La revisione viene confrontata nella stessa
transazione che blocca e aggiorna la riga. Una revisione diversa restituisce
HTTP 412 con `{"code": "revision_changed"}`, senza applicare la richiesta.
Il controllo di proprietà precede il confronto della revisione.

La garanzia di esclusione tra aggiornamenti richiede un database con supporto
effettivo a `SELECT FOR UPDATE`, per esempio PostgreSQL. SQLite non fornisce
la stessa garanzia. Il test concorrente osserva effettivamente l'attesa del
secondo scrittore in PostgreSQL; viene saltato sugli altri backend.

Senza l'header rimane disponibile il comportamento dei client esistenti.
Questi client possono ancora sovrascrivere modifiche successive: il nuovo
header non converte automaticamente il loro protocollo in un aggiornamento
condizionale. Non viene aggiunta alcuna fusione delle chiavi host SSH.

Ricaricare lo stesso contenuto e lo stesso nome non cambia `modified_at`.
Un eventuale `last_used_with_version` viene comunque aggiornato come metadato.
Il confronto è testuale: formattazioni YAML diverse possono produrre revisioni
diverse. Cambiare contenuto o nome genera una nuova revisione.

Prove: `cd backend && python manage.py test tabby.app.test_config_sync`.
Per includere la prova concorrente impostare `DATABASE_URL` verso un database
PostgreSQL isolato, con credenziali abilitate a creare il database di test.
Le prove non devono essere eseguite contro un database di produzione.
