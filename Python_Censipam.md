#Gráficos:


#Importação do exercel no visual:

import pandas as pd

df = pd.read_csv(r"C:\Users\renan.reis\Documents\PROGRAMAÇÃO_PYTHON\series_tsm_mensal_climatica.csv",
    sep=";",
    decimal=","
)

print(df)
print(df.columns)

observações:

df.columns = df.columns.str.strip()

print(df)
print(df.columns)




1 - Gráfico de Variação do Enso 2020 e 2026
#Valores sendo representandos no fundo do gráfico




import matplotlib.pyplot as mat
import pandas as pd

df['data'] = pd.to_datetime({
    'year': df['ano'],
    'month': df['mes'],
    'day': 1
})

fig, ax = mat.subplots(figsize=(14,7))

ax.plot(df['data'], df['Nino_1_2'], color="#07C0AE", label='Niño 1 e 2')
ax.plot(df['data'], df['Nino_3'], color="#0288D1", linestyle='-.', label='Niño 3')

for i in range(len(df)-1):

    inicio = df['data'].iloc[i]
    fim = df['data'].iloc[i+1]
    valor = df['Nino_3_4'].iloc[i]

    if valor >= 0.5:
        cor = '#FFD6B0'      # El Niño

    elif valor <= -0.5:
        cor = '#CFE8F7'      # La Niña

    else:
        cor = '#E6E6E6'      # Neutro

    ax.axvspan(
        inicio,
        fim,
        color=cor)

ax.set_xlabel('Ano')
ax.set_ylabel('Valores')
ax.set_title('Variação do ENSO entre 2020 e 2026')
ax.legend()



2- Gráfico de linha sobre análise geral El ninho 2026.



import matplotlib.pyplot as mat
import numpy as np

ano_2026 = df[df['ano'] == 2026]

meses = {
    1:'Jan',
    2:'Fev',
    3:'Mar',
    4:'Abr',
    5:'Mai',
    6:'Jun',
    7:'Jul',
    8:'Ago',
    9:'Set',
    10:'Out',
    11:'Nov',
    12:'Dez'
}

mat.figure(figsize=(12,6))

mat.plot(ano_2026['mes'], ano_2026['Nino_1_2'], color = '#A52A2A', marker = 'o', label='El Ninho 1 e 2')
mat.plot(ano_2026['mes'], ano_2026['Nino_3'], color = "#F1AF5D", marker = 'o', label='El Ninho 3')
mat.plot(ano_2026['mes'], ano_2026['Nino_3_4'], color = "#F5E61A", marker = 'o', label='El Ninho 3 e 4')
mat.plot(ano_2026['mes'], ano_2026['Nino_4'], color = "#FF6600", marker = 'o', label='El Ninho 4')


mat.xticks(
    ano_2026['mes'],
    ano_2026['mes'].map(meses)
)    

mat.xlabel('Mês')
mat.ylabel('Valores')
mat.title('Análise Geral El Ninho  2026')
mat.legend(title='')

mat.show()


mat.savefig('abir ENSO 3')
mat.show()



3 - Gráfico de linha de vavriação de PIB nos municípios.



import matplotlib.pyplot as plt
import matplotlib.ticker as ticker

# Se o pivot já está em bilhões, apenas copie
pivot_bilhoes = pivot.copy()

fig, ax = plt.subplots(figsize=(10, 7))

# Plotar cada município
for municipio in pivot_bilhoes.index:
    ax.plot(
        pivot_bilhoes.columns,
        pivot_bilhoes.loc[municipio],
        linewidth=1.5,
        label=municipio
    )

# Eixo X
ax.set_xticks(pivot_bilhoes.columns)
ax.set_xticklabels(pivot_bilhoes.columns, rotation=45)

# Eixo Y
ax.set_ylim(0, 25)
ax.yaxis.set_major_formatter(
    ticker.FuncFormatter(lambda x, pos: f'{x:.0f} M')
)

# Grade
ax.grid(True, linestyle='--', alpha=0.4)

# Títulos
ax.set_xlabel("Ano", fontsize=11)
ax.set_ylabel("PIB (R$ MILHÕES)", fontsize=11)
ax.set_title(
    "COMPARAÇÃO DO PIB DOS MUNICÍPIOS DA FAIXA DE FRONTEIRA DO ESTADO DE RONDÔNIA (2011–2021)",
    fontsize=13,
    fontweight="bold"
)

# Legenda
ax.legend(
    title="Município",
    loc="upper center",
    bbox_to_anchor=(0.5, -0.18),
    ncol=4,
    fontsize=8,
    frameon=False
)
