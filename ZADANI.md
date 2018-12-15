# Zadání semestrální práce

## Požadavky
- vybrat si min. 3 různé datasety (možno i vygenerovat smysluplná data náhodně)
    - každý vstupní dataset bude mít nejméně 50.000 záznamů (pokud budete předvádět v učebně, zvolte data tak, aby bylo časově reálné úlohu ukázat)
- tyto (min.) 3 datasety naimportovat do databáze či distribuovaného filesystému (např. HDFS)
- vytvořit nový dataset, který bude agregovat data z jednoho původního datasetu
- vytvořit nový dataset, který bude agregovat data ze dvou původních datasetů najednou
- vytvořit nový dataset, který bude agregovat data ze dvou datasetů najednou, z čehož jeden bude výsledkem předchozí agregace a uložit ho zpět do databáze/na file systém
- vytvořit nad kterýmkoliv datasetem index v ElasticSearch (či podobném enginu) a připravit 3 různé dotazy do tohoto indexu (nestačí index databáze, je potřeba použít indexovací engine jako je ElasticSearch)
    - využit filtrování
    - využít třídění
    - použít wildcard hledání (www.-soft-.com)
- k indexu připojit vizualizační nástroj (např. Kibanu) a udělat dashboard s 4 smysluplnými pohledy na vaše data


## Forma
- k semestrální práci je potřeba zpracovat kompletní dokumentaci (odevzdat ve formátu PDF), která bude obsahovat minimálně:
    - klasickou strukturu včetně hlavičky, rejstříku, úvodu, hlavní části, závěru
    - businessový pohled na to, jaká data se budou používat
    - popis a ukázku dat z použitých datasetů (ukázka = několik jednotek, maximálně desítek řádků z každého datasetu)
    - kompletní příkazy použité pro jednotlivé transformace
    - v příloze použité konfigurační soubory (např. pro search engine)
- na základě dokumentace by mělo být možné se vstupními datasety kompletně replikovat vaší práci!

## Obhajoba semestrálky
- dokumentaci k semestrálce (PFD) je potřeba poslat -dopředu e-mailem- a to -do 17.12.2018 BOD- (tzn. 16.12.2018 23:59)
- na posledním cvičení v tomto roce (-20.12.2018-) je potřeba -semestrálku předvést-
    - možno na vašem notebooku nebo na počítači v učebně
    - budu chtít vidět 
    -- že vám procházejí transformace
    -- že vám indexovací engine správně zpracovává data
    -- váš dashboard