# prueba
import plotly.express as px
import plotly.graph_objects as go
import dash
from dash import dcc, html, Dash, callback, Input, Output
import pandas as pd
from plotly.data import gapminder
from dash import html
from dash import dcc
from dash.dependencies import Input, Output
import pandas as pd

df_esp = pd.read_excel(r"C:\Users\Luis Jimenez\OneDrive - Carat Dominicana\Documents\data_sns_afiliados_especialidad.xlsx")

sfs = pd.read_excel(r"C:\Users\Luis Jimenez\OneDrive - Carat Dominicana\Downloads\Evolución Histórica del SFS.xlsx")

lab = pd.read_excel(r"C:\Users\Luis Jimenez\OneDrive - Carat Dominicana\Downloads\Data de Laboratorios.xlsx")

sfs['Periodo'] = sfs['Periodo'].astype(str)
sfs['Periodo'] = sfs['Periodo'].str.replace(r'^(\d{4})', r'\1-', regex=True)
sfs

especialidades = df_esp['Especialidades'].unique()
especialidades
Total_Afiliados = df_esp['Total Afiliados'].sum()
pruebas = lab['Pruebas de Laboratorios'].unique()

df_espt= df_esp[df_esp['Especialidades']!='Total general']
periodo = sfs['Periodo'].unique()
sfs['Periodo'] = pd.to_datetime(sfs['Periodo'], format='%Y-%m')

max_afiliados = sfs['total afiliados'].max()
promedio_penetracion = sfs['PENETRACIÓN%'].mean()


app = dash.Dash(__name__)

# Diseño del dashboard

# Layout principal
app.layout = html.Div(
    style={
        'backgroundColor': 'white',  # Fondo de la página (azul claro)
        'fontFamily': 'Georgia, serif',  # Fuente general
        'padding': '20px',  # Espaciado interno
        'minHeight': '100vh',  # Asegura que el fondo ocupe toda la altura de la pantalla
        'position': 'relative',  # Necesario para posicionar la imagen
    },
    children=[
        # Imagen en la esquina superior izquierda
        html.Img(
            src=r"\assets\afp_siembraDefault.png",  # Ruta a la imagen del logo
            style={
                'position': 'absolute',  # Posicionamiento absoluto
                'top': '30px',  # Distancia desde la parte superior
                'left': '30px',  # Distancia desde la izquierda
                'height': '100px',  # Altura de la imagen
            }
        ),

        # Título
        html.H1(
            "Panorama del Seguro Nacional de Salud",
            style={
                'textAlign': 'center',  # Centrar el título
                'color': '#2c3e50',  # Color del texto
                'fontFamily': 'Gotham, serif',  # Fuente personalizada
                'fontSize': '36px',  # Tamaño de la fuente
                'marginTop': '20px',  # Margen superior
                'marginBottom': '20px',  # Margen inferior
            }
        ),

        # Componente para manejar la URL
        dcc.Location(id='url', refresh=False),

        # Botones de navegación
        html.Div(
            [
                html.Button(
                    'Panorama de Consultas',
                    id='btn-page-1',
                    n_clicks=0,
                    style={
                        'backgroundColor': '#007BFF',
                        'color': 'white',
                        'border': 'none',
                        'padding': '10px 20px',
                        'margin': '0 10px',
                        'borderRadius': '5px',
                        'cursor': 'pointer',
                        'fontSize': '16px',
                        'fontFamily': 'Arial, sans-serif',
                    }
                ),
                html.Button(
                    'Panorama de los Regímenes',
                    id='btn-page-2',
                    n_clicks=0,
                    style={
                        'backgroundColor': '#28a745',
                        'color': 'white',
                        'border': 'none',
                        'padding': '10px 20px',
                        'margin': '0 10px',
                        'borderRadius': '5px',
                        'cursor': 'pointer',
                        'fontSize': '16px',
                        'fontFamily': 'Arial, sans-serif',
                    }
                ),
                html.Button(
                    'Pruebas de Laboratorio',
                    id='btn-page-3',
                    n_clicks=0,
                    style={
                        'backgroundColor': '#e99d7e',
                        'color': 'white',
                        'border': 'none',
                        'padding': '10px 20px',
                        'margin': '0 10px',
                        'borderRadius': '5px',
                        'cursor': 'pointer',
                        'fontSize': '16px',
                        'fontFamily': 'Arial, sans-serif',
                    }
                )
            ],
            style={
                'marginBottom': '20px',  # Margen inferior
                'textAlign': 'center',  # Centrar los botones
            }
        ),

        # Contenedor para el contenido de la página
        html.Div(id='page-content')
    ]
)

page_1_layout = html.Div([
    # Primera fila: Gráfico de barras horizontales
    html.Div(
        dcc.Graph(id='barra_totalPorEspecialidad'),
        style={
            'borderRadius': '15px',  # Esquinas redondeadas
            'overflow': 'hidden',  # Asegura que el gráfico no sobresalga
            'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',  # Sombra opcional
            'marginBottom': '40px',  # Espacio entre los gráficos
            'backgroundColor': 'white',  # Fondo blanco
            'padding': '10px',
            'width': '90%',  # Ancho del contenedor del gráfico
            'margin': '0 auto',  # Centrar el contenedor  # Espaciado interno
        }
    ),

    # Segunda fila: Gráfico de barras apiladas
    html.Div(
        dcc.Graph(id='stacked-bar-chart'),
        style={
            'borderRadius': '15px',
            'overflow': 'hidden',
            'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',
            'marginBottom': '40px',
            'backgroundColor': 'white',
            'padding': '10px',
            'width': '90%',  # Ancho del contenedor del gráfico
            'margin': '0 auto',  # Centrar el contenedor
        }
    ),

    # Tercera fila: Dropdown y Gráfico de pastel
    html.Div([
        html.Div(
            dcc.Dropdown(
                id='especialidad-dropdown',
                options=[{'label': i, 'value': i} for i in especialidades],
                value='Cardiología',  # Valor inicial del dropdown
                clearable=False,
                style={
                    'width': '50%',  # Ancho del dropdown
                    'margin': '0 auto',  # Centrar el dropdown
                    'marginBottom': '40px',
                    'width': '90%',  # Ancho del contenedor del gráfico
                    'margin': '0 auto',  # Centrar el contenedor  # Espacio debajo del dropdown
                }
            )
        ),
        html.Div(
            dcc.Graph(id='pie-chart'),
            style={
                'borderRadius': '15px',
                'overflow': 'hidden',
                'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',
                'backgroundColor': 'white',
                'padding': '10px'
            }
        )
    ])
])
# Layout de la página 2 (Información)
page_2_layout = html.Div([
    
    
    # Filtro de rango de fechas
    html.Label("Selecciona un rango de fechas:"),
    dcc.DatePickerRange(
        id='fecha-picker-range',
        min_date_allowed=min(sfs['Periodo']),  # Fecha mínima permitida
        max_date_allowed=max(sfs['Periodo']),  # Fecha máxima permitida
        initial_visible_month=min(sfs['Periodo']),  # Mes inicial visible
        start_date=min(sfs['Periodo']),  # Fecha inicial seleccionada
        end_date=max(sfs['Periodo']),  # Fecha final seleccionada
        display_format='DD/MM/YYYY'  # Formato de visualización
    ),
    
    # Widget para mostrar el máximo de afiliados y el promedio de penetración
    html.Div([
        html.Div([
            html.H3("Máximo de Total Afiliados"),
            html.P(f"{max_afiliados}")  # Mostrar el valor máximo
        ], style={'width': '48%', 'display': 'inline-block', 'textAlign': 'center', 'border': '1px solid #ddd', 'padding': '10px','borderRadius': '15px',  # Esquinas redondeadas
            'overflow': 'hidden','fontFamily': 'Gotham, serif',  # Asegura que el gráfico no sobresalga
            'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',  # Sombra opcional
            'marginBottom': '20px',  # Espacio entre los gráficos
            'backgroundColor': 'white'}),
        
        html.Div([
            html.H3("Penetración del Regimen Contributivo"),
            html.P(f"{promedio_penetracion:.2%}")  # Mostrar el promedio en formato de porcentaje
        ], style={'width': '48%', 'display': 'inline-block', 'textAlign': 'center', 'border': '1px solid #ddd', 'padding': '10px','borderRadius': '15px',  # Esquinas redondeadas
            'overflow': 'hidden','fontFamily': 'Gotham, serif',  # Asegura que el gráfico no sobresalga
            'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',  # Sombra opcional
            'marginBottom': '20px',  # Espacio entre los gráficos
            'backgroundColor': 'white'})
    ], style={'margin-bottom': '20px'}),
    
    # Gráfico de líneas
     html.Div(dcc.Graph(id='grafico-fechas'),style={
                'borderRadius': '15px',
                'overflow': 'hidden',
                'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',
                'backgroundColor': 'white',
                'padding': '10px'}
        ),
     
     html.Div(
            dcc.Graph(id='penetracion-chart'),
            style={
                'borderRadius': '15px',
                'overflow': 'hidden',
                'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',
                'backgroundColor': 'white',
                'padding': '10px'
            }
        )
])

page_3_layout = html.Div([
    # Primera fila: Gráfico de barras horizontales
    html.Div(
        dcc.Graph(id='barra_totalPorPrueba'),
        style={
            'borderRadius': '15px',  # Esquinas redondeadas
            'overflow': 'hidden',  # Asegura que el gráfico no sobresalga
            'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',  # Sombra opcional
            'marginBottom': '40px',  # Espacio entre los gráficos
            'backgroundColor': 'white',  # Fondo blanco
            'padding': '10px',
            'width': '90%',  # Ancho del contenedor del gráfico
            'margin': '0 auto',  # Centrar el contenedor  # Espaciado interno
        }
    ),

    # Segunda fila: Gráfico de barras apiladas
    html.Div(
        dcc.Graph(id='stacked-bar-chart-Prueba'),
        style={
            'borderRadius': '15px',
            'overflow': 'hidden',
            'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',
            'marginBottom': '40px',
            'backgroundColor': 'white',
            'padding': '10px',
            'width': '90%',  # Ancho del contenedor del gráfico
            'margin': '0 auto',  # Centrar el contenedor
        }
    ),

    # Tercera fila: Dropdown y Gráfico de pastel
    html.Div([
        html.Div(
            dcc.Dropdown(
                id='prueba-dropdown',
                options=[{'label': i, 'value': i} for i in pruebas],
                value='Hemograma Completo',  # Valor inicial del dropdown
                clearable=False,
                style={
                    'width': '50%',  # Ancho del dropdown
                    'margin': '0 auto',  # Centrar el dropdown
                    'marginBottom': '40px',
                    'width': '90%',  # Ancho del contenedor del gráfico
                    'margin': '0 auto',  # Centrar el contenedor  # Espacio debajo del dropdown
                }
            )
        ),
        html.Div(
            dcc.Graph(id='prueba-chart'),
            style={
                'borderRadius': '15px',
                'overflow': 'hidden',
                'boxShadow': '0 4px 8px rgba(0, 0, 0, 0.2)',
                'backgroundColor': 'white',
                'padding': '10px'
            }
        )
    ])
])

# Callback para actualizar el contenido de la página basado en la URL

@app.callback(
    Output('url', 'pathname'),
    [Input('btn-page-1', 'n_clicks'),
     Input('btn-page-2', 'n_clicks'),
     Input('btn-page-3', 'n_clicks')]
)
def update_url(btn1_clicks, btn2_clicks,btn3_clicks):
    ctx = dash.callback_context

    if not ctx.triggered:
        return '/'
    else:
        button_id = ctx.triggered[0]['prop_id'].split('.')[0]

        if button_id == 'btn-page-1':
            return '/'
        if button_id == 'btn-page-3':
            return '/p'
        elif button_id == 'btn-page-2':
            return '/info'

@app.callback(
    Output('page-content', 'children'),
    [Input('url', 'pathname')]
)
def display_page(pathname):
    if pathname == '/info':
        return page_2_layout
    if pathname == '/p':
        return page_3_layout
    else:
        return page_1_layout
#---------------------------------------------------------- PRIMER GRAFICO-----------------------------------------     
# Callback para actualizar el gráfico
@app.callback(
    Output('barra_totalPorEspecialidad', 'figure'),
    [Input('barra_totalPorEspecialidad', 'id')]  # Este input no es necesario, pero se deja para mantener la estructura de callback
)
def update_graph(_):
    # Crear el gráfico de barras apiladas
    df_top10 = df_espt.sort_values(by='Total Consultas', ascending=False).head(10)
    fig = px.bar(df_top10, 
                 y='Especialidades',  # Especialidades en el eje Y (barras horizontales)
                 x='Total Consultas',  # Total de consultas en el eje X
                 title='Total de Consultas por Especialidad',  # Título del gráfico
                 orientation='h',  # Barras horizontales
                 labels={'Total Consultas': 'Número de Consultas', 'Especialidades': 'Especialidad'})
    
    fig.update_layout({
            'plot_bgcolor': 'rgba(240, 240, 240, 1)',  # Fondo del área del gráfico
            'paper_bgcolor': 'rgba(240, 240, 240, 1)',  # Fondo del área exterior
        })
    return fig

#---------------------------------------------------------- SEGUNDO GRAFICO-----------------------------------------    
# Callback para actualizar el gráfico
@app.callback(
    Output('stacked-bar-chart', 'figure'),
    [Input('stacked-bar-chart', 'id')]  # Este input no es necesario, pero se deja para mantener la estructura de callback
)
def update_graph(_):
    # Crear el gráfico de barras apiladas
    fig = px.bar(df_espt, 
                 x='Especialidades', 
                 y=['Total Afiliados', 'Total No Afiliados'], 
                 title='Comparativo de Afiliados y No Afiliados por Especialidad',
                 labels={'value': 'Número de Pacientes', 'variable': 'Tipo'},
                 barmode='stack')  # Modo apilado
    return fig

#---------------------------------------------------------- TERCER GRAFICO-----------------------------------------   

@app.callback(
    Output('pie-chart', 'figure'),
    [Input('especialidad-dropdown','value')]  # Este input no es necesario, pero se deja para mantener la estructura
)
def update_pie_chart(especialidad_seleccionada):
    # Filtrar el DataFrame para la especialidad seleccionada
    filtered_df = df_espt[df_espt['Especialidades'] == especialidad_seleccionada]    
    # Crear un gráfico de pastel para la proporción de afiliados vs no afiliados
    Masculino = filtered_df['Usurios Masculinos'].values[0]
    Femenino = filtered_df['Uusarios Femeninos'].values[0]
    fig = px.pie(filtered_df, names=['Maculino', 'Femenino'], 
                 values=[Masculino, Femenino], 
                 title='Proporción de Afiliados vs No Afiliados')
    return fig




@app.callback(
    Output('grafico-fechas', 'figure'),
    [Input('fecha-picker-range', 'start_date'),
     Input('fecha-picker-range', 'end_date')]
)
def update_graph(start_date, end_date):
    # Convertir las fechas seleccionadas a datetime
    start_date = pd.to_datetime(start_date)
    end_date = pd.to_datetime(end_date)

    # Filtrar el DataFrame basado en el rango de fechas seleccionado
    filtered_sfs = sfs[(sfs['Periodo'] >= start_date) & (sfs['Periodo'] <= end_date)]

    # Crear el gráfico con Plotly Express
    fig = px.line(filtered_sfs,
                  x='Periodo',  # Eje X: Periodo
                  y=['total afiliados','Régimen Contributivo (RC)'],  # Eje Y: Valor
                  title=f'Afiliados desde {start_date.strftime("%d/%m/%Y")} hasta {end_date.strftime("%d/%m/%Y")}',
                  )
    
    fig.update_traces(
    line=dict(
        color='blue',  
        width=4,  # Grosor de la línea (4 píxeles)
    )
)

    return fig

@app.callback(
    Output('penetracion-chart', 'figure'),
    [Input('fecha-picker-range', 'start_date'),
     Input('fecha-picker-range', 'end_date')]
)

def update_graph(start_date, end_date):
    # Convertir las fechas seleccionadas a datetime
    start_date = pd.to_datetime(start_date)
    end_date = pd.to_datetime(end_date)

    # Filtrar el DataFrame basado en el rango de fechas seleccionado
    filtered_sfs = sfs[(sfs['Periodo'] >= start_date) & (sfs['Periodo'] <= end_date)]
    

    # Crear el gráfico con Plotly Express
    fig = px.line(filtered_sfs,
                  x='Periodo',  # Eje X: Periodo
                  y=['PENETRACIÓN%']*100,  # Eje Y: Valor
                  title=f'Penetración del Regimen Contributivo desde {start_date.strftime("%d/%m/%Y")} hasta {end_date.strftime("%d/%m/%Y")}')

    return fig


#---------------------------------------------------------- PRIMER GRAFICO-----------------------------------------     
# Callback para actualizar el gráfico
@app.callback(
    Output('barra_totalPorPrueba', 'figure'),
    [Input('barra_totalPorPrueba', 'id')]  # Este input no es necesario, pero se deja para mantener la estructura de callback
)
def update_graph(_):
    # Crear el gráfico de barras apiladas
    lab10 = lab.sort_values(by='Total Pruebas', ascending=False).head(10)
    fig = px.bar(lab10, 
                 y='Pruebas de Laboratorios',  # Especialidades en el eje Y (barras horizontales)
                 x='Total Pruebas',  # Total de consultas en el eje X
                 title='Total de Pruebas por Especialidad',  # Título del gráfico
                 orientation='h',  # Barras horizontales
                 labels={'Total Pruebas': 'Número de Pruebas', 'Pruebas de Laboratorios': 'Pruebas'})
    
    fig.update_layout({
            'plot_bgcolor': 'rgba(240, 240, 240, 1)',  # Fondo del área del gráfico
            'paper_bgcolor': 'rgba(240, 240, 240, 1)',  # Fondo del área exterior
        })
    return fig

#---------------------------------------------------------- SEGUNDO GRAFICO-----------------------------------------    
# Callback para actualizar el gráfico
@app.callback(
    Output('stacked-bar-chart-Prueba', 'figure'),
    [Input('stacked-bar-chart-Prueba', 'id')]  # Este input no es necesario, pero se deja para mantener la estructura de callback
)
def update_graph(_):
    # Crear el gráfico de barras apiladas
    lab10 = lab.sort_values(by='Total Pruebas', ascending=False).head(10)
    fig = px.bar(lab10, 
                 x='Pruebas de Laboratorios', 
                 y=['Total de Afiliados', 'Total No Afiliados'], 
                 title='Comparativo de Afiliados y No Afiliados por Pruebas de laboratorio',
                 labels={'value': 'Número de Pacientes', 'variable': 'Tipo'},
                 barmode='stack')  # Modo apilado
    return fig

#---------------------------------------------------------- TERCER GRAFICO-----------------------------------------   

@app.callback(
    Output('prueba-chart', 'figure'),
    [Input('prueba-dropdown','value')]  # Este input no es necesario, pero se deja para mantener la estructura
)
def update_pie_chart(prueba_seleccionada):
    # Filtrar el DataFrame para la especialidad seleccionada
    filtered_lab = lab[lab['Pruebas de Laboratorios'] == prueba_seleccionada]    
    # Crear un gráfico de pastel para la proporción de afiliados vs no afiliados
    Masculino = filtered_lab['Usuarios Masculinos'].values[0]
    Femenino = filtered_lab['Usuarios Femeninos'].values[0]
    fig = px.pie(filtered_lab, names=['Maculino', 'Femenino'], 
                 values=[Masculino, Femenino], 
                 title='Proporción de Afiliados vs No Afiliados')
    return fig


# Ejecutar la aplicación
if __name__ == '__main__':
    app.run(debug=True)
