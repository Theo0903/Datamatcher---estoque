# Datamatcher---estoque
Manipulação de Excel via Pandas
Utilizado basicamente para montagem do Estoque utilizado pela empresa que trabalho.

import pandas as pd

df_Atc = pd.read_excel(r'C:\AADS_XLS\ajuste\ATACADAO-01-11-2023-HR08-18.xlsx')
df_Atc = df_Atc.drop_duplicates(subset=['codigo'])

df_Jnf = pd.read_excel(r'C:\AADS_XLS\ajuste\JNF-01-11-2023-HR08-17.xlsx')
df_Jnf = df_Jnf.drop_duplicates(subset=['codigo'])


df_Cbc = pd.read_excel(r'C:\AADS_XLS\ajuste\CBC-01-11-2023-HR08-17.xlsx')
df_Cbc = df_Cbc.drop_duplicates(subset=['codigo'])


df_Jfilial = pd.read_excel(r'C:\AADS_XLS\ajuste\JNF_FILIAL_SP-01-11-2023-HR08-18.xlsx')
df_Jfilial = df_Jfilial.drop_duplicates(subset=['codigo'])

df_Bibas = pd.read_excel(r'C:\AADS_XLS\ajuste\BIBAS_MATRIZ-01-11-2023-HR08-17.xlsx')
df_Bibas = df_Bibas.drop_duplicates(subset=['codigo'])

df_estoque = pd.read_excel(r'C:\AADS_XLS\ajuste\ESTOQUE CAMBUCI 06.11 (PT1).xlsx')

df_Importacao = pd.read_excel(r'C:\AADS_XLS\ajuste\\ERROS DE IMPORTAÇÃO.xlsx')

df_Exclusao = pd.read_excel(r'C:\AADS_XLS\ajuste\\LISTA DE EXCLUSÃO.xlsx')
# caso eu queira criar a lista de exclusão com base no arquivo:
"""letra_inicial = "Z"
df_delete_ItemZ = df_Cbc.query('descricao.str.startswith(@letra_inicial)')
df_delete_Refer = df_Cbc.query('refer.isna()')
df_delete_Fabricante = df_Cbc.query('fabricante.isna()')"""

# Removendo itens da lista de exclusão e erros de importação da base Cbc
endereco = df_Cbc['endereco']

exclusao = df_Exclusao['codigo'].tolist()

importacao = df_Importacao['codigo'].tolist()

df_Cbc = df_Cbc[~df_Cbc['codigo'].isin(exclusao)]
print('Excluindo itens da Lista de Exclusão')

df_Cbc = df_Cbc[~df_Cbc['codigo'].isin(importacao)]
print('Excluindo itens da Erros de importação')

df_Cbc['TIPO'] = df_Cbc['codigo'].map(df_estoque.set_index('CÓD')['TIPO'])

df_Cbc['CASA'] = 'CBC'

# Suponha que "Código" seja a coluna que corresponde entre os DataFrames
df_Cbc['Estoque_Bibas'] = df_Cbc['codigo'].map(df_Bibas.set_index('codigo')['estoque'])


def atualizar_estoque_bibas(row):
    if row['Estoque_Bibas'] > row['estoque']:
        row['CASA'] = 'BIBAS'  # Atualiza a coluna 'CASA' para 'BIBAS'
        row['estoque'] = row['Estoque_Bibas']
        print("Processando a atualização da Bibas")
        return row
    else:
        return row


df_Cbc = df_Cbc.apply(atualizar_estoque_bibas, axis=1)
# df_Cbc = df_Cbc.drop(columns=['Estoque_Bibas'])

df_Cbc['Estoque_Jfilial'] = df_Cbc['codigo'].map(df_Jfilial.set_index('codigo')['estoque'])


def atualizar_estoque_Jfilial(row):
    if row['Estoque_Jfilial'] > row['estoque']:
        row['CASA'] = 'J.FILIAL'  # Atualiza a coluna 'CASA' para 'J.FILIAL'
        row['estoque'] = row['Estoque_Jfilial']
        print("Processando a atualização da J.Filial")
        return row
    else:
        return row


df_Cbc = df_Cbc.apply(atualizar_estoque_Jfilial, axis=1)
# df_Cbc = df_Cbc.drop(columns=['Estoque_Jfilial'])


df_Cbc['Estoque_atc'] = df_Cbc['codigo'].map(df_Atc.set_index('codigo')['estoque'])


def atualizar_estoque_atc(row):
    if row['Estoque_atc'] > row['estoque'] and row['TIPO'] == 'NACIONAL':
        row['CASA'] = 'ATC'  # Atualiza a coluna 'CASA' para 'ATC'
        row['estoque'] = row['Estoque_atc']
        print("Processando a atualização da ATC")
        return row
    else:
        return row


df_Cbc = df_Cbc.apply(atualizar_estoque_atc, axis=1)
# df_Cbc = df_Cbc.drop(columns=['Estoque_atc'])


def manter_bibas(row):
    if row['Estoque_Bibas'] > 4:
        row['estoque'] = row['Estoque_Bibas']
        print("Ajustando para Bibas maior que 4.")
        return row
    else:
        return row


df_Cbc = df_Cbc.apply(manter_bibas, axis=1)

df_Cbc.to_excel(f'C:\\AADS_XLS\\ajuste\\Criando_Estoque.xlsx', index=False)
