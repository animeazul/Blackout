
# ¿Cuándo volveran los asteroides asesinos?

Queria hacer algo divertido con powerbi y python solamente por diversion y encontre estas dos Apis
- ***Asteroid and comet close approaches to the planets in the past and future*** https://ssd-api.jpl.nasa.gov/doc/cad.html
- **Asteroids - NeoWs**   https://api.nasa.gov/neo/rest/v1/neo/browse/
Y pense en realizar un pequeño reporte de los asteoroides que pasan cerca de la tierra.

<html>
<iframe width="800" height="600" src="https://app.powerbi.com/view?r=eyJrIjoiMWU5NmU1MDctZGY4NC00NTQyLTgzZTctMzc0MGQ0NjBhODI0IiwidCI6ImE1YmRlMjRmLTdlODgtNGE0ZC04ZTg4LWY3YTFmYWY5ZGE5NCIsImMiOjR9" frameborder="0" allowFullScreen="true"></iframe>
</html>

## Primero API KEY
Si buscan las llaves para Apis pueden registrarse en https://api.nasa.gov/ y tambien encontrar mas APIS.

## Datasource power bi
Para esto use dos tipos de fuente: 
1.- Una web para traer los asteoroides que pasan por la tierra en estos ultimos 7 dias
2.- Python script para del punto uno buscar cuando volveran o pasaron esos asteroides a la tierra

### 1.- Fuente Web
Agrego el codigo en power query M , recuerden usar su Key en la peticion pyhton (para las consultas induviduales) ,y transformandolas 
para usar los datos (recuerden que por lo general la respuesta de un api se entrega en formato .json)

```M
let
    Source = Json.Document(Web.Contents("https://api.nasa.gov/neo/rest/v1/feed?api_key=XXXXXXXXXX")),
    near_earth_objects = Source[near_earth_objects],
    #"Converted to Table" = Record.ToTable(near_earth_objects),
    #"Expanded Value" = Table.ExpandListColumn(#"Converted to Table", "Value"),
    #"Expanded Value1" = Table.ExpandRecordColumn(#"Expanded Value", "Value", {"links", "id", "neo_reference_id", "name", "nasa_jpl_url", "absolute_magnitude_h", "estimated_diameter", "is_potentially_hazardous_asteroid", "close_approach_data", "is_sentry_object"}, {"Value.links", "Value.id", "Value.neo_reference_id", "Value.name", "Value.nasa_jpl_url", "Value.absolute_magnitude_h", "Value.estimated_diameter", "Value.is_potentially_hazardous_asteroid", "Value.close_approach_data", "Value.is_sentry_object"}),
    #"Expanded Value.links" = Table.ExpandRecordColumn(#"Expanded Value1", "Value.links", {"self"}, {"Value.links.self"}),
    #"Expanded Value.estimated_diameter" = Table.ExpandRecordColumn(#"Expanded Value.links", "Value.estimated_diameter", {"kilometers"}, {"Value.estimated_diameter.kilometers"}),
    #"Expanded Value.estimated_diameter.kilometers" = Table.ExpandRecordColumn(#"Expanded Value.estimated_diameter", "Value.estimated_diameter.kilometers", {"estimated_diameter_min", "estimated_diameter_max"}, {"Value.estimated_diameter.kilometers.estimated_diameter_min", "Value.estimated_diameter.kilometers.estimated_diameter_max"}),
    #"Expanded Value.close_approach_data" = Table.ExpandListColumn(#"Expanded Value.estimated_diameter.kilometers", "Value.close_approach_data"),
    #"Expanded Value.close_approach_data1" = Table.ExpandRecordColumn(#"Expanded Value.close_approach_data", "Value.close_approach_data", {"close_approach_date", "close_approach_date_full", "epoch_date_close_approach", "relative_velocity", "miss_distance", "orbiting_body"}, {"Value.close_approach_data.close_approach_date", "Value.close_approach_data.close_approach_date_full", "Value.close_approach_data.epoch_date_close_approach", "Value.close_approach_data.relative_velocity", "Value.close_approach_data.miss_distance", "Value.close_approach_data.orbiting_body"}),
    #"Expanded Value.close_approach_data.relative_velocity" = Table.ExpandRecordColumn(#"Expanded Value.close_approach_data1", "Value.close_approach_data.relative_velocity", {"kilometers_per_hour"}, {"Value.close_approach_data.relative_velocity.kilometers_per_hour"}),
    #"Expanded Value.close_approach_data.miss_distance" = Table.ExpandRecordColumn(#"Expanded Value.close_approach_data.relative_velocity", "Value.close_approach_data.miss_distance", {"kilometers"}, {"Value.close_approach_data.miss_distance.kilometers"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Value.close_approach_data.miss_distance",{{"Value.absolute_magnitude_h", type number}, {"Value.estimated_diameter.kilometers.estimated_diameter_max", type number}, {"Value.is_potentially_hazardous_asteroid", type logical}, {"Value.close_approach_data.relative_velocity.kilometers_per_hour", type number}, {"Value.close_approach_data.miss_distance.kilometers", type number}, {"Value.is_sentry_object", type logical}, {"Value.estimated_diameter.kilometers.estimated_diameter_min", type number}, {"Value.close_approach_data.epoch_date_close_approach", type number}})
in
    #"Changed Type"
```
### 2 .- Python scrypt 
El codigo M para este es el siguiente pero desglozo el codigo python despues.
Si lo quieren copiar recuerden usar su propia Key en **api_key=XXXXXXXX**

```M
let
    Source = Python.Execute("# -*- coding: utf-8 -*-#(lf)""""""#(lf)Created on Sat Nov  9 23:17:16 2019#(lf)#(lf)@author: jjdiaz#(lf)""""""#(lf)#(lf)#(lf)import requests#(lf)import json#(lf)import pandas as pd#(lf)import numpy as np#(lf)response = requests.get(""https://api.nasa.gov/neo/rest/v1/feed?api_key=XXXXXX"")#(lf)datastore =response.json()#(lf)#(lf)lista = []#(lf)for x in datastore[""near_earth_objects""]:#(lf)  #print(datastore[""near_earth_objects""][x]) #(lf)  for y in datastore[""near_earth_objects""][x]:#(lf)     lista.append(y[""id""])#(lf)      #(lf)u =np.unique(np.array(lista))#(lf)df2=pd.DataFrame()#(lf)#(lf)errornum=0#(lf)for w in u:#(lf)  # print( w)#(lf)   response2 = requests.get(""https://ssd-api.jpl.nasa.gov/cad.api?des=""+w+""&date-min=1900-01-01&date-max=2100-01-01&dist-max=0.2"")#(lf)   json_string =response2.json()#(lf)   #print(json_string[""data""])#(lf)   try:#(lf)       df = json_string[""data""]#(lf)   except:#(lf)       errornum=1+errornum#(lf)   else:#(lf)    a=0    #(lf)    for i in df:#(lf)       #(lf)       df[a].append(np.array2string(w))#(lf)       a=a+1#(lf)       #(lf)    df= pd.DataFrame(df)#(lf)    df2=df2.append(df, ignore_index=True)#(lf)print(df2)#(lf)      #(lf)#(lf)   #(lf)      "),
    df1 = Source{[Name="df2"]}[Value],
    #"Changed Type" = Table.TransformColumnTypes(df1,{{"0", type text}, {"1", Int64.Type}, {"2", type number}, {"3", type datetime}, {"4", type number}, {"5", type number}, {"6", type number}, {"7", type number}, {"8", type number}, {"9", type text}, {"10", type number}, {"11", type text}}),
    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"0", "des"}, {"1", "Orbit Id"}, {"2", "jd"}, {"3", "time of close-approeach"}, {"4", "dist"}, {"5", "dist_min"}, {"6", "dist_max"}, {"7", "v_rel"}, {"8", "v_inf"}, {"9", "T-sigma-f"}, {"10", "h"}, {"11", "ID"}}),
    #"Replaced Value" = Table.ReplaceValue(#"Renamed Columns","'","",Replacer.ReplaceText,{"ID"})
in
    #"Replaced Value"
```

El codigo en Python es el siguiente:

### Solicitud a Api **Asteroids - NeoWs**  en Python
Esta da los asteroides cercanos a ala tierra en los proximos 7 dias , recuerden usar su propia Key en **api_key=XXXXXXXX**


```python
import requests
import json #Por si las dudas
import pandas as pd
import numpy as np
response = requests.get("https://api.nasa.gov/neo/rest/v1/feed?api_key=XXXXXXXX")
datastore =response.json()
```
### Agregar todos los Id a una lista 
Se agregan todos los Id's a una lista y solo por si las dudas  solo traigo los valores unico de la lista

```python
for x in datastore["near_earth_objects"]:
  for y in datastore["near_earth_objects"][x]:
     lista.append(y["id"])
u =np.unique(np.array(lista))
```

### Buscar esos Id en la otra Api

Despues se hace solicitud a Api SSD/CNEOS   _SSD (Solar System Dynamics) and CNEOS (Center for Near-Earth Object Studies)_ por cada Id de la lista y se une a cada uno de las peticiones para poder saber si va a volver a pasar cerca de la tierra dentro la fecha del get
y cambia la lista a dataframe

```python
df2=pd.DataFrame()
errornum=0
#Solicitud a Api SSD/CNEOS  por cada Id de la lista 
for w in u:
   response2 = requests.get("https://ssd-api.jpl.nasa.gov/cad.api?des="+w+"&date-min=1900-01-01&date-max=2100-01-01&dist-max=0.2")
   json_string =response2.json()
#En caso de que el Id no exista dentro de la peticion de la Api   
   try:
       df = json_string["data"]
   except:
       errornum=1+errornum
   else:
    a=0
#Agrega el Id de NeoWs a los Datos de SSD/CNEOS    
    for i in df:
       df[a].append(np.array2string(w))
       a=a+1
#Cambia la lista a Dataframe y los concatena en un nuevo Dataframe       
    df= pd.DataFrame(df)
    df2=df2.append(df, ignore_index=True)
print(df2)
```     



Si buscan algunas otras Apis pueden encontrarlas en
https://api.nasa.gov/

