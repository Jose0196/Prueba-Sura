# Prueba-Sura
import requests
from bs4 import BeautifulSoup
import pandas as pd

def scrape_banco_general():
    # URL del sitio web
    url = 'https://www.superbancos.gob.pa/estadisticas-financieras/carta-bancaria/estadisticas-financieras'

    # Realizar la solicitud GET al sitio web
    response = requests.get(url)

    # Crear objeto BeautifulSoup
    soup = BeautifulSoup(response.content, 'html.parser')

    # Encontrar el formulario para seleccionar el banco
    form = soup.find('form', {'id': 'CartaBancaria'})

    # Obtener el token CSRF necesario para enviar la solicitud de búsqueda
    csrf_token = form.find('input', {'name': 'csrfmiddlewaretoken'}).get('value')

    # Crear el payload para la solicitud POST
    payload = {
        'csrfmiddlewaretoken': csrf_token,
        'envio_datos': 'true',
        'formato': 'individual',
        'banco': 'Banco General',
        'datos': 'Activos',
        'agrupacion': 'trimestral'
    }

    # Realizar la solicitud POST para obtener los datos
    response = requests.post(url, data=payload)

    # Crear objeto BeautifulSoup para los datos obtenidos
    soup = BeautifulSoup(response.content, 'html.parser')

    # Encontrar la tabla que contiene los datos
    table = soup.find('table', {'class': 'responsive'})

    # Leer la tabla en un DataFrame usando pandas
    df = pd.read_html(str(table))[0]

    # Filtrar y formatear los datos
    df = df[['Período', 'Total']].copy()
    df['Período'] = pd.to_datetime(df['Período'], format='%d-%m-%Y')
    df = df.set_index('Período')
    df.columns = ['Total Activos']

    return df

if __name__ == '__main__':
    df = scrape_banco_general()

    # Filtrar los datos para el rango de fechas deseado
    start_date = pd.to_datetime('2021-04-01')
    end_date = pd.to_datetime('2023-03-31')
    df = df.loc[(df.index >= start_date) & (df.index <= end_date)]

    # Guardar los datos en un archivo Excel
    df.to_excel('salida.xlsx')
