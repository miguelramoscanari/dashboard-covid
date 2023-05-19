# Dashboard Covid con Power BI y ETL con Python
Implementación de un Dashboard sobre COVID, usando por Visualizacíon Power Bi y para ETL con Python.

# Contenido
- "covid.csv", "covid_mundial.csv", "dim_calendario.csv", "dim_departamento.csv", "dim_pais.csv: fuente de datos
- "covid_etl_miguel.ipynb": notebook con el codigo fuente python, para proceso ETL
- "py_covid_miguel.pbix": libro de POWER BI

# Diagrama Datamart del proyecto
Se diseño dos datarmart:
- Datamart con tabla hecho "covid" y con dimensiones "dim_departamento", "dim_calendario"
- Datamart con tabla hecho "covid_mundial" y con dimensiones "dim_pais", "dim_calendario"
- En la tabla hecho "covid" y "covid_mundial" ambos tiene como metrica "nro positivo" y "nro_fallecido"
![Image text](https://github.com/miguelramoscanari/dashboard-covid/blob/main/Datamart_covid.png)

# Dashboard en Power BI de covid del Perú y en el Mundo.

# ![Image text](https://github.com/miguelramoscanari/dashboard-covid/blob/main/dashboard_peru.png)


# ![Image text](https://github.com/miguelramoscanari/dashboard-covid/blob/main/dashboard_mundo.png)

# Origen de la fuente de datos del proyecto
```
Casos positivos por COVID-19 - [Ministerio de Salud - MINSA]
https://www.datosabiertos.gob.pe/dataset/casos-positivos-por-covid-19-ministerio-de-salud-minsa

Fallecidos por COVID-19 - [Ministerio de Salud - MINSA]
https://www.datosabiertos.gob.pe/dataset/fallecidos-por-covid-19-ministerio-de-salud-minsa

Data Mundial de covid
https://github.com/owid/covid-19-data/tree/master/public/data
```

# Proceso ETL con Python

Importando librerias, para importar csv que estan en Internet
```
from io import StringIO
import pandas as pd
import requests

headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.76 Safari/537.36'}
```
Creando los DataFrames correspondientes
```
# LEER FALLECIDOS POR COVID
url = 'https://files.minsa.gob.pe/s/t9AFqRbXw3F55Ho/download'
s = requests.get(url, headers= headers).text
df_fall = pd.read_csv(StringIO(s), sep=";", encoding='utf_8')

# LEER POSITIVOS POR COVID
url = 'https://files.minsa.gob.pe/s/eRqxR35ZCxrzNgr/download'
s = requests.get(url, headers= headers).text
df_posi = pd.read_csv(StringIO(s), sep=";", encoding='utf_8')

# LEER COVID MUNDIAL
url = 'https://covid.ourworldindata.org/data/owid-covid-data.csv'
s = requests.get(url, headers= headers).text
df_mundo = pd.read_csv(StringIO(s), sep=",", encoding='utf_8')

# LEER DEPARTAMENTOS
df_dep = pd.read_excel("departamentos.xlsx")

# CAMBIANDO NOMBRE DE COLUMNAS A MINUSCULAS
df_fal = df_fall.rename(str.lower, axis='columns')
df_pos = df_posi.rename(str.lower, axis='columns')
df_dep = df_dep.rename(str.lower, axis='columns')
df_mun = df_mundo.rename(str.lower, axis='columns')
```
#  ETL de fallecidos por covid
```
# CAMBIANDO EL CAMPO FECHA A TIPO STRING, DARLE FORMATO AAAA-MM-01
df_fal['fecha_fallecimiento'] = df_fal['fecha_fallecimiento'].astype('string').str[:8]
df_fal['fecha_fallecimiento'] = df_fal['fecha_fallecimiento'].str[:4] + '-' + df_fal['fecha_fallecimiento'].str[4:6] + '-01'

# HALLANDO 'etapa_de_vida'
df_fal['etapa_de_vida']=df_fal['edad_declarada'].apply(lambda x: 'NIÑO' if x<=9 else ('ADOLESCENTE' if x<=19 else ('JOVEN' if x<=29 else ('ADULTO' if x<=59 else 'MAYOR'))))

# JOIN ENTRE DATAFRAME FALLECIDOS Y DEPARTAMENTOS
df_fal = df_fal.merge(df_dep, left_on='departamento', right_on='departamento', how='left')

# CREANDO DATAFRAME FALLECIDOS POR SEXO Y ETAPA DE VIDA Y POR DIA
df_fallecido = df_fal.groupby(['departamentocode', 'fecha_fallecimiento', 'sexo', 'etapa_de_vida'])['departamentocode'].count().reset_index(name='cantidad')
df_fallecido = df_fallecido.rename(columns={'fecha_fallecimiento': 'fecha'})

df_fallecido.head()
```

```
	departamentocode	fecha	sexo	etapa_de_vida	cantidad
0	PE-AMA	2020--0-01	FEMENINO	ADOLESCENTE	3
1	PE-AMA	2020--0-01	FEMENINO	ADULTO	54
2	PE-AMA	2020--0-01	FEMENINO	JOVEN	9
3	PE-AMA	2020--0-01	FEMENINO	MAYOR	95
4	PE-AMA	2020--0-01	FEMENINO	NIÑO	3
```

# ETL de positivos por covid

```
# CAMBIANDO EL CAMPO FECHA A TIPO STRING, DARLE FORMATO AAAA-MM-01
df_pos['fecha_resultado'] = df_pos['fecha_resultado'].astype('string').str[:8]
df_pos['fecha_resultado'] = df_pos['fecha_resultado'].str[:4] + '-' + df_pos['fecha_resultado'].str[4:6] + '-01'

# HALLANDO 'etapa_de_vida'
df_pos['etapa_de_vida']=df_pos['edad'].apply(lambda x: 'NIÑO' if x<=9 else ('ADOLESCENTE' if x<=19 else ('JOVEN' if x<=29 else ('ADULTO' if x<=59 else 'MAYOR'))))

# JOIN ENTRE DATAFRAME POSITIVOS Y DEPARTAMENTOS
df_pos = df_pos.merge(df_dep, left_on='departamento', right_on='departamento', how='left')

# CREANDO DATAFRAME POSITIVOS POR SEXO Y ETAPA DE VIDA Y POR DIA
df_positivo = df_pos.groupby(['departamentocode', 'fecha_resultado', 'sexo', 'etapa_de_vida'])['departamentocode'].count().reset_index(name='cantidad')
df_positivo = df_positivo.rename(columns={'fecha_resultado': 'fecha'})

df_positivo.head()
```
```
departamentocode	fecha	sexo	etapa_de_vida	cantidad
0	PE-AMA	2020-04-01	FEMENINO	ADOLESCENTE	3
1	PE-AMA	2020-04-01	FEMENINO	ADULTO	50
2	PE-AMA	2020-04-01	FEMENINO	JOVEN	14
3	PE-AMA	2020-04-01	FEMENINO	MAYOR	9
4	PE-AMA	2020-04-01	FEMENINO	NIÑO	6
```
JOIN DE POSITIVOS Y FALLECIDOS POR COVID
```
df_covid = df_positivo.merge(df_fallecido[['cantidad', 'departamentocode', 'fecha', 'sexo', 'etapa_de_vida']], on=['departamentocode', 'fecha', 'sexo', 'etapa_de_vida'], how='left')
df_covid = df_covid.rename(columns={'cantidad_x': 'positivo', 'cantidad_y': 'fallecido'})
df_covid['fallecido'] = df_covid['fallecido'].fillna(0)
```
# ETL mundial covid

```
# CAMBIANDO EL CAMPO FECHA A TIPO STRING, FORMATO AAAA-MM-01
df_mun['date'] = df_mun['date'].astype('string')
df_mun['date'] = df_mun['date'].str[:7] + '-01'

# RENOMBRANDO CAMPOS PARA FACILIDAD DE LECTURA
df_mun = df_mun.rename(columns={'iso_code': 'paisCode', 'location':'pais',  'continent':'continente', 'date': 'fecha'})

# AGRUPANDO PARA CREAR TABLE HECHOS COVID MUNDIAL
df_covid_mundial = df_mun.groupby(['paisCode', 'fecha'])['new_cases', 'new_deaths'].sum().reset_index()
df_covid_mundial = df_covid_mundial.rename(columns={'new_cases': 'positivo', 'new_deaths':'fallecido'})

# CREANDO DIMENSION PAIS
df_pais = df_mun.groupby(['paisCode', 'pais', 'continente'])['new_cases'].count().reset_index()
df_pais = df_pais[['paisCode', 'pais', 'continente']]
```

# ETL Creando dimension calendario

```
# HALLANDO FECHA MAXIMA
fec_max1 = df_covid['fecha'].max()
fec_max2 = df_covid_mundial['fecha'].max()
fec_max =  fec_max1 if fec_max1 > fec_max2 else fec_max2
fec_max = pd.to_datetime(fec_max).date()

# HALLANDO FECHA MINIMA
fec_min1 = df_covid['fecha'].min()
fec_min2 = df_covid_mundial['fecha'].min()
fec_min =  fec_min1 if fec_min1 > fec_min2 else fec_min2
fec_min = pd.to_datetime(fec_min).date()

mes_dict = {
    i + 1: mes
    for i, mes in enumerate(['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Setiembre', 'Octubre', 'Noviembre', 'Diciembre'])
}

df_calendario = pd.DataFrame({'fecha': pd.date_range(fec_min, fec_max)})
df_calendario['Año'] = df_calendario.fecha.dt.year
df_calendario['Trimestre'] = df_calendario.fecha.dt.quarter.map(lambda x :  'Trim. ' + str(x))
df_calendario['Mes'] = df_calendario.fecha.dt.month.map(lambda x: mes_dict[x])
df_calendario['Dia'] = df_calendario.fecha.dt.day
df_calendario['AñoMes'] = df_calendario.fecha.astype('string').str[:4] + df_calendario.fecha.astype('string').str[5:7]
```
# ETL Creando dimension departamento

EN POWER BI, EN GRAFICO TIPO MAPAS, PARA QUE IDENTIFICA UN DEPARTAMENTO DE UN PAIS ES 
NECESARIO CREAR UN CAMPO CON FORMATO >> DEPARTAMENTO (DEPARTAMENTO), (PAIS)
```
df_dep['departamentoPais'] = 'DEPARTAMENTO ' + df_dep['departamento'] + ', PERU'
```
El resto del codigo esta el notebook de python.





