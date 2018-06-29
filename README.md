# Rappresentazione JSON per le fatture elettroniche

**Proof-of-concept** per una rappresentazione in formato JSON di fatture elettroniche.

## Rationale

Le fatture elettroniche vengono trasmesse ed archiviate in formato **XML**.

La documentazione tecnica del **formato XML** delle fatture elettroniche è disponibile al seguente url: http://www.fatturapa.gov.it/export/fatturazione/it/normativa/f-2.htm

Tuttavia per lo sviluppo di front-end per la creazione di fatture è più comodo manipolare i dati in formato **JSON**, e convertire ad XML al termine delle fasi che richiedono l'interazione con l'utente.

## Proof-of-concept

Inizialmente si è tentato di convertire lo schema XSD in uno [schema JSON](http://json-schema.org/) usando la libreria [jgeXml](https://github.com/Mermade/jgeXml):
```
nodejs ./node_modules/jgexml/testxsd2j.js Schema_del_file_xml_FatturaPA_versione_1.2.xsd  > fatturaPA_1.2_schema.json
```
ottenendo però uno schema non utilizzabile.

Successivamente i files XML di esempio sono stati convertiti a JSON per mezzo della libreria [node-xml2json](https://github.com/buglabs/node-xml2json) con i comandi:
```
./bin/xml2json.js IT01234567890_FPR01.xml | grep -v 'xmlns:' | grep -v 'xsi:' | sed 's/p:FatturaElettronica/FatturaElettronica/g' > IT01234567890_FPR01.json
./bin/xml2json.js IT01234567890_FPR02.xml | grep -v 'xmlns:' | grep -v 'xsi:' | sed 's/p:FatturaElettronica/FatturaElettronica/g' > IT01234567890_FPR02.json
./bin/xml2json.js IT01234567890_FPR03.xml | grep -v 'xmlns:' | grep -v 'xsi:' | sed 's/p:FatturaElettronica/FatturaElettronica/g' > IT01234567890_FPR03.json
./bin/xml2json.js IT01234567890_FPA01.xml | grep -v 'xmlns:' | grep -v 'xsi:' | sed 's/p:FatturaElettronica/FatturaElettronica/g' > IT01234567890_FPA01.json
./bin/xml2json.js IT01234567890_FPA02.xml | grep -v 'xmlns:' | grep -v 'xsi:' | sed 's/p:FatturaElettronica/FatturaElettronica/g' > IT01234567890_FPA02.json
./bin/xml2json.js IT01234567890_FPA03.xml | grep -v 'xmlns:' | grep -v 'xsi:' | sed 's/p:FatturaElettronica/FatturaElettronica/g' > IT01234567890_FPA03.json
```

Questa conversione automatica funziona più o meno, ma c'è un intoppo per quegli elementi che possono ripetersi (es. `FatturaElettronica.FatturaElettronicaBody` e `FatturaElettronica.FatturaElettronicaBody[1].DatiBeniServizi.DettaglioLinee`), nel caso ve ne sia uno solo la conversione genera un JSON dove invece di un `array` ci è un `object` vedi:
- https://social.msdn.microsoft.com/Forums/windowsapps/en-US/03fe9c57-b7a8-41aa-ade4-652b6b5eafc3/single-element-xml-to-json-array
- http://jersey.576304.n2.nabble.com/Single-Element-Arrays-and-JSON-td5532105.html
Per questi casi occorre wrappare manualmente l'`object` con `[]`.

Si è quindi generato uno schema a partire dai files fattura JSON di esempio, per mezzo del servizio on-line [jsonschema](https://www.jsonschema.net/), che dopo semplificazione (`grep -v '$id'`) e aggiunta di campi `title` e `description` desunti dalle SPECIFICHE TECNICHE OPERATIVE DEL FORMATO DELLA FATTURA DEL SISTEMA DI INTERSCAMBIO ha dato origine allo schema `www/fatturaPA_1.2_schema_semplificato.json`.

Tutti i files di esempio JSON sono validati dallo schema come si può verificare con:
```
bin/validate.js
```

## Demo

Con lo schema semplificato così ottenuto e [JSON Editor](https://github.com/json-editor/json-editor) è possibile generare automaticamente un editor:

```
yarn install
make
python -m SimpleHTTPServer
```
visitare http://localhost:8000/www/index.html

## TODO

- completare lo schema

- aggiungere funzione per caricare file JSON esistente

- aggiungere funzioni all'editor usando [le estensioni allo schema JSON specifiche di JSON Editor](https://github.com/json-editor/json-editor#json-schema-support)

- aggiungere funzione di export a XML (idea: usare un template client-side tipo moustache che infila i campi presenti nel JSON in uno schema di file XML)

- aggiungere funzione di import di un file XML esistente

- unit tests:
  - verificare che lo schema JSON valida tutti i files fattura JSON di esempio
  - verificare che i files XML generati sono compliant con lo schema XSD

## Riferimenti

- https://jsonlint.com/

- http://json-schema.org/

- https://spacetelescope.github.io/understanding-json-schema/reference/index.html

- https://jsonschemalint.com/#/version/draft-06/markup/json

- https://github.com/json-editor/json-editor