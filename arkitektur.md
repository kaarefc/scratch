# Foreslået arkitektur

## Opgaver der skal løses
* Ingest i arkivet
* Automatisk QA af et batch (gyldige data og metadata)
* Manuel QA af et batch
* Generering af bevaringsmetadata
* Generering af formidlingskopi

## Principper:
* Tidlig ingest
* QA, bevarings- og tilgængeliggørelsesopgaver foregår i arkivet
* Autonome komponenter over forkromet workflow
* Automatisk QA skal desuden kunne køres af Ninestars

## Elementer i løsningen:

### Eksisterende elementer:
* Bitmagasin
* DOMS
* Hadoop-platform
* DOMS-GUI

### Nye elementer
* Fremvisningsinterface af sider (Prototype fra Toke)
* Early Ingest-platform
* Platform til validering af metadata i DOMS
* Udtræk af filer/metadata til manuel QA

## Ingest i arkivet
Overordnet beskrivelse:
  Givet et batch, læg alle jp2-filer i bitmagasinet og alle xml-filer (inklusive ALTO) i Fedora

Foreslået løsning:
* Hotfolder overvåges for nye batches
* Alle filer med extension *.jp2 lægges i bitmagasinet (.md5-fil bruges til checksumscheck) (løsningen bør være konfigurerbar til forskellige extensions)
* Alle andre filer lægges i Fedora (.md5-fil bruges til checksumscheck)
* Directorystruktur bevares i bitmagasinet i FilID
* Directorystruktur bevares i DOMS ved et objekt pr. directory, en datastream pr. fil. Directorynavne bevares i object labels, filnavne bevares i datastream labels.

## Bitarkivet og bånd
Overordnet beskrivelse:
  Filer skal gemmes i bitarkivet på ten ben, et nearline og et offline. Begge de involverede ben vil have en anseelig mændge cache, men vil være tape-backed.

Foreslået løsning:
* Filerne lægges på de to ben ved modtagelse,
* Filer i cache vil blive rullet på bånd når et bestemt, men pt. udefineret (tidsbestemt, eller eksplicit ved godkendelse af batch), signal sendes. 
* Filer fra batches der fejler validering vil blive slettet igen.
* Vi antager at vi kan validere og afvise et batch før det bliver nødvendigt at rulle det på bånd, men i værste fald er der spildt noget plads på båndet
* Nøglerne der giver adgang til at slette fra bitmagasinet skal eksplicit skaffes hvis der skal fjernes et ekstra batch. Som udgangspunkt kan nøglerne også slette allerede godkendte batches.

## Opgaver på data (jp2-filer)
Overordnet beskrivelse:
  Alle operationer på data køres som hadoop-jobs på filer i bitmagasinet. Herunder karakterisering og generering af formidlingskopier.

Foreslået løsning:
* jpylizer køres i map-skridt af map/reduce-job. Resultatet lægges i DOMS (som datastream) i reduce-skridt.
* Udtræk af histogram køres i map-skridt af map/reduce-job. Resultatet lægges i DOMS i reduce-skridt.
* Generering af formidlingskopi køres i map-skridt af map/reduce-job. Filen skrives direkte til formidlingsstorage. Eventuelle fejl skrives tilbage til DOMS i reduce-skridt.
* Der laves ingen validering i map/reduce-job. Dette foretages som efterbehandling af metadata.

## Validering af metadata (i DOMS)
Overordnet beskrivelse:
  Alle jobs der validerer metadata opfattes som jobs på et helt batch. Metadatavalidering kan være lokalt for én xml-fil eller kræve kendskab til sammenhæng mellem flere xml-filer i et batch - f.eks. validering af samme batchnummer i alt metadata eller validering af fortløbende sidenumre

Foreslået løsning:
 * Validering implementeres som gennemløb af en træstruktur (svarende til filstruktur-hierarki), med mulighed for validering i hver knude
 * Validering af en enkelt XML-fil kan foretages med XML-schema og schematron
 * Træet kan gennemløbes med et filsystem eller DOMS som nederste niveau
 * Resultatet af en validering skrives tilbage til DOMS
 * En fejlet validering skal medføre en notifikation til Supervisor
 * Hellere små autonome skridt end ét stort workflow

## Manuel QA
Overordnet beskrivelse:
  Der skal foretages manuel QA på filer og metadata udvalgt efter statistisk princip. Derudover skal der evt. foretages manuel QA på filer vi kan identificere som mistænkelige (f.eks. et helt mørkt batch). Manuel QA kræver adgang til et system til at inspicere jp2-filer og metadata. Vi fravælger i første omgang at lave et egentligt workflowstyringsprogram.

Foreslået løsning:
* Der etableres en fremvisningsløsning til avissider i bitmagasinet. Toke har lavet en første udgave.
* Der etableres mulighed for direkte links til objekter i DOMS GUI.
* Der udtrækkes hvilke sider der skal checkes udfra den statistiske visning.
* Ud fra dette udtræk laves et regneark (csv-fil) med link til fremvisning og DOMS GUI.
* Til dette ark tilføjes mistænkelige objekter med tilsvarende links og en begrundelse for hvorfor de er mistænkelige
* Vi laver ingen support af hvordan regnearkene benyttes til at godkende batches

## Fejlede batches
Overordnet beskrivelse:
  Batches fejles eller godkendes som et hele. Man kan ikke afvise dele af et batch, og leverandøren vil aflevere et nyt batch.

Foreslået løsning
* Hvis et batch fejler, skal hele batches slettes fra Bitarkivet
* Slette fra bitmagasinet involverer anskaffelse af slette nøglen
* Hvis et batch fejler, skal hele batches slettes fra DOMS
* Doms kan slette uden en bestemt slette nøgle, da alt bliver bevaret alligevel, og kan genskabes.
* Et genafleveret batch skal behandles på samme måde som et nyt batch
