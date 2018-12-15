# Domácí úkol na BI-BIG
Vypracoval Matyáš Skalický

# Dataset
Dataset byl pro účel zadání upraven a rozdělen na 3 datasety za pomocí frameworku Pandas. JupyterNotebook obsahující tuto transformaci je popsán v souboru [DatasetTransform.ipynb](DatasetTransform.ipynb).

### Zdroj datasetu:
[https://www.kaggle.com/mehdidag/black-friday](https://www.kaggle.com/mehdidag/black-friday)

### Popis datasetu
Vzhledem k faktu, že pro agregace používám jeden a ten samý dataset rozdělený na 3 části, popíšu nejprve všechny sloupce datasetu a následně jejich rozdělení do nových menších datasetů.

Dataset obsahuje 550 000 záznamů z Black Friday z amerického obchodního domu. Obsahuje jak numerické, tak kategorické proměnné. Obsahuje chybějící hodnoty. Bylo dogenerován sloupec Refferal a upraveno Product_ID na numerickou hodnotu (odstraněním předpony P a úvodních nul).

### Sloupce
Popis sloupců původního datasetu:

- **User_ID - 537577 non-null int64** - Unikátní identifikátor uživatele.
- **Product_ID - 537577 non-null int64** - Unikátní identifikátor produktu.
- **Gender - 537577 non-null object** - Pohlaví zákazníka (M/F).
- **Age - 537577 non-null object** - Věková skupina zákazníka.
- **Occupation - 537577 non-null int64** - Kategorie dle typu zaměstnání (maskovaná, číslo).
- **City_Category - 537577 non-null object** - Kategorie města (maskovaná - hodnoty A, B, C).
- **Stay_In_Current_City_Years - 537577 non-null object** - Délka pobytu v současném městě v letech.
- **Marital_Status - 537577 non-null int64** - Manželský status (1=ženatý, 0=jinak).
- **Product_Category_1 - 537577 non-null int64** - Primární kategorie produktu. (maskovaná, číslo)
- **Product_Category_2 - 370591 non-null float64** - Sekundární kategorie produktu. (maskovaná, číslo)
- **Product_Category_3 - 164278 non-null float64** - Tercérní kategorie produktu. (maskovaná, číslo)
- **Purchase - 537577 non-null int64** - Částka, kterou uživatel utratil v USD za celou dobu.
- **Refferal - 537577 non-null object** - Stránka, odkud uživatel přišel, když se registroval (náhodně generováno).

### Ukázka datasetu (10 řádků)
TODO

### Rozdělení datasetu na 3 nové datasety.
TODO

***
# Spouštění databázového clusteru
Následující postup vychází z [návodu prezentovaného na 5. cvičení](https://courses.fit.cvut.cz/BI-BIG/tutorials/05/index.html). Postup vyžaduje nainstalovaný a plně funkční [docker-compose](https://docs.docker.com/compose/). 

## Spark
### Build image pro spark
Bude vytvořen image *spark* který bude využit dle parametrů při spuštění jak pro worker, tak pro master node.

```bash
docker build -f spark.df -t spark .
```

### Spuštění master a worker node
```bash
docker-compose up
```

### spark-shell
#### spuštění spark-shell
Následující příkaz spustí kontejner s přístupem do bashe na popředí.
```bash
docker run -it -p 8088:8088 -p 8042:8042 -p 4041:4040 --name driver -h driver spark:latest bash
```

### Připojení spark-shell na master
V běžícím bashi kontejneru se připojíme na mster node spark clusteru. 
```bash
spark-shell --master spark://<IP adresa mastera>:7077
```

## HDFS kontejner
Pro spuštění kontejneru s Hadoop HDFS použijeme následující příkaz:
```bash
docker run --name hadoop -t -i sequenceiq/hadoop-docker /etc/bootstrap.sh -bash
```

Pro zjednodušení práce si přidáme do cesty image odkaz na hadoop.
```bash
export PATH=$PATH:/usr/local/hadoop/bin/
```

***
# Import dat do databázového clusteru
Vytvoříme složku *data* v rootu image.
```bash
mkdir /data
hdfs dfs -mkdir /data
```

Kopie dat do kontejneru na kterém běží HFDS.
```bash
docker cp data/BlackFriday.csv 62c4cd83e124:/data/BlackFriday.csv
```

Vložení dat z filesystému image do HDFS.
```bash
hdfs dfs -put /data/BlackFriday.csv /data/BlackFriday.csv
```

Načtení csv souboru z HDFS do sparku
```scala
val blackFriday = spark.sqlContext.read.format("csv").option("header", "true").option("inferSchema", "true").load("hdfs://172.17.0.5:9000/data/BlackFriday.csv")
```

# Agregace
### 1. vytvořit nový dataset, který bude agregovat data z jednoho původního datasetu

### 2. vytvořit nový dataset, který bude agregovat data ze dvou původních datasetů najednou

### 3. vytvořit nový dataset, který bude agregovat data ze dvou datasetů najednou, z čehož jeden bude výsledkem předchozí agregace a uložit ho zpět do databáze/na file systém

# Export dat ze spark clusteru
Na HDFS se vytvoří složka /data/BlackFridayResult s výsledným csv.
```bash
blackFriday.coalesce(1).write.format("com.databricks.spark.csv").option("header","true").save("hdfs://172.17.0.5:9000/data/BlackFridayResult")
```

Kopie dat z HDFS do filesystému kontejneru na kterém běží HDFS.
```bash
hdfs dfs -get /data/BlackFridayResult /data/BlackFridayResult
```

Kopie dat z filesystému kontejneru.
```bash
docker cp 62c4cd83e124:/data/BlackFridayResult .
```

***
***
# Vyhladávací index
## Nahrání dat a tvorba indexu
Použijeme kontejner s technologií ElasticSearch pro indexaci csv souboru. Zároveň spustíme i kontejner Kibana, jterý slouží pro vizualizaci a dotazování nad daty. Data do ElasticSearch dostaneme za pomocí docker kontejneru LogStash. [Postup vychází ze cvičení číslo 9](https://courses.fit.cvut.cz/BI-BIG/tutorials/09/index.html).

Jdeme do složky logstash. Kontejnery spustíme na pozadí s použitím příkazu:
```bash
docker-compose up -d
```

Data se načtou dle definice konfigurace v souboru *logstash.conf*:
```json
input {
	file {
		path => "/datasets/BlackFridayEdit.csv"
		start_position => "beginning"
	}
}

filter {
	csv {
		separator => ","
		columns => ["User_ID", "Product_ID", "Gender", "Age", "Occupation", "City_Category", "Stay_In_Current_City_Years", "Marital_Status", "Product_Category_1", "Product_Category_2", "Product_Category_3", "Purchase", "Refferal"]
	}
	mutate {convert => ["User_ID", "integer"]}
	mutate {convert => ["Product_ID", "integer"]}
	mutate {convert => ["Occupation", "integer"]}
	mutate {convert => ["Product_Category_1", "integer"]}
	mutate {convert => ["Product_Category_2", "integer"]}
	mutate {convert => ["Product_Category_3", "integer"]}
	mutate {convert => ["Purchase", "integer"]}
	}

output {
	elasticsearch {
		hosts => "http://elasticsearch:9200"
		index => "blackfriday"
	}
}

```

Uživatelské rozhraní Kibana je na adrese: [http://127.0.0.1:5601](http://127.0.0.1:5601). Pro vytvoření indexu zvolíme *Management -> Index Patterns* a do pole "Index pattern" okopírujeme název indexu, tedy "blackfriday". Následně klikneme next a v time filter zvolíme možnost "I don't want to use Time Filter" jelikož náš dataset neobsahuje časové značky. Klikneme na Create index pattern a index je tak vytvořen.

## Dotazy na index
Dotazy v Kibaně lze provádět v záložce *Discover*. Konzole se nachází na *Dev Tools -> Console*. 

### Filtrování - vyhledání všech mužů:
Dev Tools:
 ```json
GET /blackfriday/doc/_search
{
    "query": {
        "match": {
        "Gender": "M"
        } 
    }
}
 ```

 Kibana: 
 ```Gender:M```

### Třídění - seřadit zákazníky podle útraty
Dev Tools:
```json
GET /blackfriday/doc/_search
{
    "sort": {
        "Purchase": {
            "order" : "desc"
        }
    }
}
```

### Wildcard hledání - všichni zákazníci co přišli z Google
Dev Tools:
```json
GET /blackfriday/doc/_search
{
    "query": {
        "wildcard": {
            "Refferal": "www.google.*"
        } 
    }
}
```

Kibana:
``` Refferal:www.google.* ```

## Dashboard
TODO
