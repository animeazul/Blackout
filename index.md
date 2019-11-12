
# Cuando volveran los asteroides malditos

Queria hacer algo divertido con powerbi y python solamente por diversion y encontre estas dos Apis
- ***Asteroid and comet close approaches to the planets in the past and future*** https://ssd-api.jpl.nasa.gov/doc/cad.html
- **Asteroids - NeoWs**   https://api.nasa.gov/neo/rest/v1/neo/browse/
Y pense en realizar un pequeño reporte de los asteoroides que pasan cerca de la tierra.

## 1.- Primero API KEY
Si buscan las llaves para Apis pueden registrarse en https://api.nasa.gov/ y tambien encontrar mas APIS.

# Solicitud a Api **Asteroids - NeoWs**  en Python
Esta da los asteroides cercanos a ala tierra en los proximos 7 dias , recuerden usar su propia Key en **api_key=XXXXXXXX**


```python
import requests
import json #Por si las dudas
import pandas as pd
import numpy as np
response = requests.get("https://api.nasa.gov/neo/rest/v1/feed?api_key=XXXXXXXX")
datastore =response.json()
```
# Agregar todos los Id a una lista 
Se agregan todos los Id's a una lista y solo por si las dudas  solo traigo los valores unico de la lista

```python
for x in datastore["near_earth_objects"]:
  for y in datastore["near_earth_objects"][x]:
     lista.append(y["id"])
u =np.unique(np.array(lista))
```
<html>
  <a href="google.com">hola</a>
  </html>
# Buscar esos Id en la otra Api

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

