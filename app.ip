import dash
from dash import dcc, html, Input, Output
import plotly.express as px
import pandas as pd
import numpy as np
import os # IMPORTANTE: Para leer variables de entorno de Binder

# 1. Cargar datos
df = pd.read_csv('power_nap_vs_coffee_effectiveness_dataset.csv')

# 2. Funciones de apoyo
def predecir_alerta(horas_sueno):
    log_odds = 0.1342 + (0.0213 * horas_sueno)
    probabilidad = 1 / (1 + np.exp(-log_odds))
    return probabilidad

# 3. Inicializar la App con soporte para Proxy de Binder
app = dash.Dash(__name__)

# CONFIGURACIÓN CRÍTICA PARA BINDER:
if 'JUPYTERHUB_SERVICE_PREFIX' in os.environ:
    app.config.update({
        'requests_pathname_prefix': os.environ['JUPYTERHUB_SERVICE_PREFIX'] + 'proxy/8050/'
    })

# 4. Diseño (Layout)
app.layout = html.Div([
    html.H1("Dashboard de Gestión de Fatiga - Proyecto U. Compensar", style={'textAlign': 'center'}),
    
    html.Div([
        html.P("Este tablero muestra los hallazgos sobre la eficacia del café vs la siesta.")
    ], style={'padding': '20px'}),

    dcc.Graph(id='box-plot', figure=px.box(df, x="intervention_type", y="alertness_score_after", 
                                          color="intervention_type", title="Distribución de Alerta por Intervención")),
    
    html.Label("Selecciona el rango de edad:"),
    dcc.RangeSlider(
        id='age-slider',
        min=df['age'].min(), max=df['age'].max(),
        value=[df['age'].min(), df['age'].max()],
        marks={str(int(age)): str(int(age)) for age in np.linspace(df['age'].min(), df['age'].max(), 5)}
    ),
    
    dcc.Graph(id='scatter-plot'),

    html.Hr(),

    html.Div([
        html.H2("Calculadora de Probabilidad de Alerta"),
        html.P("Mueve el slider para ver cómo tus horas de sueño afectan la probabilidad de éxito."),
        dcc.Slider(id='sueno-slider', min=0, max=12, step=0.5, value=7, marks={i: str(i) for i in range(13)}),
        html.Div(id='resultado-prediccion', style={'fontSize': '24px', 'fontWeight': 'bold', 'color': '#2c3e50', 'marginTop': '20px'})
    ], style={'backgroundColor': '#f9f9f9', 'padding': '30px', 'borderRadius': '10px'})
])

# 5. Callbacks
@app.callback(
    Output('scatter-plot', 'figure'),
    Input('age-slider', 'value')
)
def update_scatter(selected_age):
    filtered_df = df[(df['age'] >= selected_age[0]) & (df['age'] <= selected_age[1])]
    fig = px.scatter(filtered_df, x="intervention_duration_minutes", y="alertness_score_after",
                     color="intervention_type", 
                     title="Relación Duración vs Alerta (Filtrado por Edad)")
    return fig

@app.callback(
    Output('resultado-prediccion', 'children'),
    Input('sueno-slider', 'value')
)
def update_prediction(horas):
    prob = predecir_alerta(horas)
    return f"Probabilidad de estar 'Bien Alerta' (>=70 pts): {prob*100:.2f}%"

if __name__ == '__main__':
    app.run_server(host='0.0.0.0', port=8050, debug=False)
