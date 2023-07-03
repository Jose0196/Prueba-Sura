# Prueba-Sura
import requests
from bs4 import BeautifulSoup
import pandas as pd

from openpyxl import Workbook

# URL de la página web
url = "https://www.superbancos.gob.pa/estadisticas-financieras/carta-bancaria/estadisticas-financieras"

# Headers de la solicitud HTTP
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36"
}

# Realizar solicitud HTTP GET a la página web
response = requests.get(url, headers=headers)

# Crear objeto BeautifulSoup con el contenido HTML de la página web
soup = BeautifulSoup(response.content, "html.parser")

# Encontrar el formulario de búsqueda
form = soup.find("form", {"id": "form"})

# Obtener los parámetros necesarios para enviar la solicitud de descarga de datos
viewstate = form.find("input", {"id": "__VIEWSTATE"})["value"]
eventvalidation = form.find("input", {"id": "__EVENTVALIDATION"})["value"]

# Datos de búsqueda (segundo trimestre de 2021 hasta primer trimestre de 2023)
data = {
    "__EVENTTARGET": "",
    "__EVENTARGUMENT": "",
    "__LASTFOCUS": "",
    "__VIEWSTATE": viewstate,
    "__EVENTVALIDATION": eventvalidation,
    "ctl00$contenido$ddlVista": "Carta Bancaria/Estadisticas Financieras",
    "ctl00$contenido$ddlTipoEst": "2",
    "ctl00$contenido$ddlTipoDesag": "3",
    "ctl00$contenido$ddlBanco": "22",  # ID de Banco General en la lista de bancos
    "ctl00$contenido$ddlMoneda": "T",
    "ctl00$contenido$ddlIndicadores": "TAC",
    "ctl00$contenido$ddlAnio": ["2021", "2022", "2023"],
    "ctl00$contenido$ddlTrimestre": ["2", "3", "4", "1"],
    "ctl00$contenido$btnBuscar": "Buscar"
}

# Realizar solicitud HTTP POST para descargar los datos
response = requests.post(url, headers=headers, data=data)

# Leer la respuesta como un archivo Excel utilizando pandas
df = pd.read_excel(response.content, sheet_name="Respuesta")

# Filtrar las columnas deseadas (Total de activos y Patrimonio total)
df_filtered = df[["Fecha", "Total de activos", "Patrimonio total"]]

# Crear un nuevo archivo Excel con los datos filtrados
output_file = "Salida.xlsx"
with pd.ExcelWriter(output_file) as writer:
    df_filtered.to_excel(writer, sheet_name="Datos", index=False)

# Imprimir mensaje de éxito
print(f"Los datos se han extraído correctamente y se han guardado en el archivo '{output_file}'.")
