# NLP-Projekt: Klassifikation von Unternehmenssnippets
## Datenquellen
Im Rahmen eines Praktikums wurden die Unternehmensdaten zunächst mithilfe einer Web-Scaper Extension von relevanten Webseiten gezogen, um eine Datenbasis für Sales-Aktivitäten zu haben. Mithilfe dieser Anwendung kann in kurzer Zeit eine hohe Anzahl an Unternehmensdaten angereichert werden, indem beispielweise Ausstellerlisten von Messen oder öffentliche Mitgliederdaten von Verbänden heruntergeladen werden. Zur Klassifizierung wurden den Unternehmenseinträgen zunächst händisch Segmente und Subsegmente zugeordnet.

| Unternehmensname  | Straße | PLZ | Land | TelefonNr. | Webseite | E-Mail | Segment   | Subsegment |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
| XY AG  | Musterstr. 69  | 65283  | DE  | 0443 582  | www.xyag.de  | info@xyag.de | Industrial & Services | Service Bureau |  

~~Im Prinzip wurde eine manuelle Textklassifikation vorgenommen, um die Unternehmen in vorgegebene Kategorien einzuordnen. Im Zuge des Praktikums sind knapp 2000 klssifizierte Datenpunkte entstanden, die von der Salesabteilung und dem Marketing für die Neukundenakquise verwendet werden. Dieser Datensatz kann in der [list.csv](/list.csv) eingesehen werden.~~

Im Verlauf des Projektes stellte sich heraus, dass der geplante Datensatz zu unausgeglichen ist. Deshalb wechselten wir auf den Forbes Top 2000 Companies Datensatz. Dieser enthält in ähnlicher Form wie der angedachte Datensatz Firmen und deren entsprechenden Sector.

Für das Projekt können die gelabelten Datensätze für das Training des Klassifizierungsalgorithmus verwendet werden. 

Für das Natural Language Processing nutzen wir sogenannte Snippets, das ist die Kurzbeschreibung der Suchmaschine (in dem Fall [Bing](www.bing.com). Diese enthalten meist alle relevnaten Informationen und Begriffe, da der Suchmaschinennutzer darüber auf die Webseite gelockt werden soll.

Die Snippets werden über einen [Crawler](/crawler.py) , welcher den Python Web-Scraper [Silenium](https://selenium-python.readthedocs.io/) verwendet. Hier wird die Website jedes in der Forbes Top2000.csv befindlichen Unternehmens bei Bing gesucht und das entsprechende Snippet dem Datenpunkt hinzugefügt und in dem [output_forbes_dif_final.csv](/output_forbes_dif_final.csv) gespeichert.

Da durch den Crawler nicht alle Snippets gezogen werden konnten und auch trotz Anpassung der Sucheinstellungen bei Bing auf nur deutsche Websiten teilweise englische Snippets gezogen wurden, überarbeiteten wir den Datensatz händisch noch einmal und besserten Snippets aus, sodass wir am Ende auf eine gesammt Anzahl von __1971__ Datenpunkten kamen.

## Datentransformation 
Die Daten, die man über den Web-Scraper erhält, müssen zunächst vorverarbeitet werden, um einheitliche, durch Semikolon getrennte Datensätze zu erhalten. Für diese vorgeschobene Bearbeitung wurde die search and replace Funktion im Text Editor verwendet.

In der [model.py](/model.py) umfasst die Methode ```prepare_snippets```die Vorverarbeitung jedes einzelnen Snippets. Diese Methode verfügt über 9 de-/aktivierbare Vorverarbeitungsschritte. Diese umfassen:

* raw_string_return = False #verarbeitete Snippet als String zurückgegeben
* remove_int = False #Integer entfernen
* lowercase = True #Alles in Kleinbuchstaben
* stopwords = True #Stopwords entfernen
* punctuations = True #Satzzeichen entfernen
* only_nouns_n_adjs = True #Nur Nomen und Adjektive verwenden
* lammatize = True #Lammatizierung
* reduce=True #Nur einzigartige Worte
* word_embeddings = True #Snippet in Word-Embedding transformieren

## Model

Das Model selbst wurde in einem [Google Colab](https://colab.research.google.com/drive/1zX34Y4aOSBLFZ4gA3lCsfgEPTk0MsmxM?usp=sharing) trainiert und evaluiert. Für das Hyperparametertuning wurden alle drei verwendeten Modelle einer RandomGridSearch unterzogen und jeweils Crossvalidated auf den gesamten Datensatz. Final wurde ein KNN, Random Forest und SVC mit einer Convolution von 10 über den Trainingsdatensatz gestackt, sodass folgende Evaluation bei einem Test-Train-Val Split von 80% rauskam:

```
Test Set Accuracy Score :
 0.37142857142857144

Train Set Accuracy Score :
 0.7049808429118773

Classification Report :
                             precision    recall  f1-score   support

    Consumer Discretionary       0.22      0.36      0.28        83
          Consumer Staples       0.00      0.00      0.00        34
                    Energy       0.00      0.00      0.00        32
                Financials       0.44      0.86      0.58       229
               Health Care       1.00      0.03      0.06        33
               Industrials       0.00      0.00      0.00        84
    Information Technology       0.25      0.09      0.13        34
                 Materials       0.10      0.05      0.07        60
Telecommunication Services       0.00      0.00      0.00        11
                 Utilities       0.00      0.00      0.00        30

                  accuracy                           0.37       630
                 macro avg       0.20      0.14      0.11       630
              weighted avg       0.26      0.37      0.26       630
```

Für die finale Implementierung wurde das Model auf den gesamten Datensatz gefittet.

## Deployment

Der Crawler und das Model wurden in einer Flask Application zusammengeführt, sodass über ein GUI eine Suchanfrage möglich ist, welche live das Snippet der eingeggebenen Firma sucht und klassifiziert. 

Zum lokalen Deployment muss folgendes ausgeführt werden:

```
git clone https://github.com/peterbehrens/NLP-project.git
cd NLP-project
pip install -r requirements.txt
python app.py
```

Jetzt sollte die Flask-App laufen und kann über [http://localhost:5000](http://localhost:5000) aufgerufen werden.
