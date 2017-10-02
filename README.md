# nplso-umbau-modell

Im Nutzungsplanungsmodell `SO_Nutzungsplanung_20170901` hat sich ein kleiner Fehler eingeschlichen: die UUID als OID gingen verloren. Aus diesem Grund wurde das Modell `SO_Nutzungsplanung_20170901` erstellt, welches diesen Umstand korrigiert.

Das Pilotprojekt _Wisen_ war zum Zeitpunkt der Entdeckung bereits am Laufen. Sämtliche Prozesse (Import, VeriSO, ...) wurden jedoch auf das korrigierte Modell abgestützt. Aus diesem Grund musste eine Möglichkeit des Modellumbaues gefunden werden:

## Modellumbau

### Datensatz "alt" importieren

`java -jar ~/apps/ili2pg-3.9.1/ili2pg.jar --dbhost geodb-dev.cgjofbdf5rqg.eu-central-1.rds.amazonaws.com --dbdatabase xanadu2 --dbusr stefan --dbpwd XXXXXXXX --nameByTopic --disableValidation --defaultSrsCode 2056 --sqlEnableNull --createGeomIdx --models SO_Nutzungsplanung_20170901 --dbschema rohr_alt --importTid --import rohr_20170901.xtf`

Die `--importTid` ist entscheident, da damit eine zusätzliche Spalte `t_ili_tid` mit den Transfer-Id aus dem XTF in der Datenbank angelegt wird.

### Update t_ili_tid

Mit dem SQL-Update-Befehlen aus der Datei `sql/update_tid.sql` werden die originalen Transfer-Id zu UUID gemacht. In den SQL-Befehlen muss vorher der Schemanamen angepasst werden.

### Datensatz "alt" exportieren

`java -jar ~/apps/ili2pg-3.9.1/ili2pg.jar --dbhost geodb-dev.cgjofbdf5rqg.eu-central-1.rds.amazonaws.com --dbdatabase xanadu2 --dbusr stefan --dbpwd XXXXXXXX --nameByTopic --disableValidation --defaultSrsCode 2056 --sqlEnableNull --createGeomIdx --models SO_Nutzungsplanung_20170901 --dbschema rohr_alt --importTid --export rohr_20170901_export.xtf`

Der Datensatz wird - immer noch im alten, fehlerhaften Modell - exportiert. Das exportierte XTF muss mit `ilivalidator` geprüft werden und kann mit `xmllint` schöner formatiert werden.

`xmllint --format rohr_20170901_export.xtf -o rohr_20170901_export.xtf`

`java -jar ~/apps/ilivalidator-1.4.0/ilivalidator.jar rohr_20170901_export.xtf`

Die visuelle Kontrolle zeigt, dass im XTF die TID neu einer UUID entsprechen.

### Kopieren und umbenennen der Datei

Die exportierte Datei wird kopiert:

`cp rohr_20170901_export.xtf rohr_20170915_umbau.xtf`

### Modellanpassung

Als letzter Schritt muss im XTF der Modellnamen mittels search/replace in einem Texteditor (Achtung: Encoding beachten!) angepasst werden. Dies ist einfach möglich, da einzig die TID eine UUID ist und sonst nichts (v.a.) strukturell geändert hat.

