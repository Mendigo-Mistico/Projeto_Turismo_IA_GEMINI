# Projeto_Notas_Escolares_IA_GEMINI
Projeto Desafio da Imersão GEMINI + ALURA

# Descrição:

O meu projeto consiste em uma analise de planilha de notas de um escola, e retorna quais alunos tiveram as menores notas no geral, em cada materia e da um resumo para cada uma dessa materias para que possa ser feita uma aula direcionada as materias em deficit.

Observação:
É importante notar que, para os alunos do Fundamental I (1º ao 5º ano), algumas matérias como Física, Química, Biologia, Filosofia, Sociologia etc., não são oferecidas, portanto, possuem  nota 0,00. Isso não significa necessariamente que o aluno teria baixo desempenho nessas áreas.
A análise se baseia apenas nas médias gerais e nas notas individuais. Outros fatores, como o contexto individual do aluno, o nível de dificuldade da turma e da escola, entre outros, também podem ser relevantes para uma avaliação mais completa.



**Instalação inicial das bibliotecas e configurações de API_KEY**

!pip install -q -U google-generativeai
# Importar a Python SDK
import google.generativeai as genai

# Uso Seguro da API_KEY
from google.colab import userdata
api_key = userdata.get('Secret_Key')
genai.configure(api_key=api_key)

#Calulos Matematicos e Manipulação de DataFrames
import numpy as np
import pandas as pd

# Entrada de Dados
from google.colab import files
from pathlib import Path
import os
import io


Definindo parametros da para a IA


generation_config = {
    "candidate_count": 1,
    "temperature": 1,
}

safety_settings = {
    "Harassment": "BLOCK_NONE",
    "Hate": "BLOCK_NONE",
    "Sexual": "BLOCK_NONE",
    "Dangerous": "BLOCK_NONE",
}

Selecionando a versão utilizada no modelo e setando os parâmentros

model = genai.GenerativeModel (model_name='gemini-1.5-pro-latest',
                               generation_config=generation_config,
                               safety_settings=safety_settings)

Base de Entrada da Planilha

*   Serie sendo de 1º Fundamental a 3º Medio
*   Salas sendo de A, B ou C
*   Materia podenendo ser Colocadas de acordo com a necessidade do professor

*   **Lembrete: As medias do alunos de ensino fundamental, serão diferentes por terem menos materias que no ensimo médio**





![Planilha.png](data:image/png;base64,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)

**Todas as informações fora geradas pelo Gemini e são fictícias**.

Subindo o Arquivo **CSV** das notas dos alunos e Transformando em DataFrame

#Exluir Arquivos Anteriores
arquivos_para_manter = ['.config', 'sample_data']

for filename in os.listdir('/content'):
  if filename not in arquivos_para_manter:
    caminho_completo = os.path.join('/content', filename)
    if os.path.isfile(caminho_completo):
      os.remove(caminho_completo)
      print(f"Arquivo '{filename}' excluído.")
    else:
      print(f"'{filename}' não é um arquivo.")

# Carrega o arquivo das Notas
uploaded = files.upload()

df_notas = pd.read_csv(list(uploaded.keys())[0])

# Preenche as áreas vazias com 0
df_notas = df_notas.fillna(0)
df_notas


Converti as Colunas em Numericas para não dar erro no momento de calcular as médias. Deixando somente as três de cabeçalho como texto

# Seleciona as colunas que não devem ser numéricas
colunas_numericas = df_notas.columns[3:]

# Converte as colunas selecionadas para numéricas
df_notas[colunas_numericas] = df_notas[colunas_numericas].apply(pd.to_numeric, errors='coerce')
df_notas.dtypes

**Analise do Data Frame**


*   Encontrar os 3 Aluno com a menor nota geral.
*   Encontrar qual a materia e sala que teve a menor nota.



# Substitui valores vazios por 0
df_notas = df_notas.fillna(0)
menor_nota = np.nanmin(df_notas.iloc[:, 3:].values)

# Cria uma coluna "Nota Geral" com a média das notas
df_notas['Nota Geral'] = df_notas.iloc[:, 3:].mean(axis=1)

# Ordena o DataFrame pela "Nota Geral" e pega os 3 alunos com as menores notas
alunos_menor_nota = df_notas.sort_values(by='Nota Geral').head(3)

# Exibe os alunos com a menor nota geral em ordem alfabetica por nome
print("3 Alunos com Menor Nota Geral:")
alunos_menor_nota = alunos_menor_nota.sort_values(by='Nome')
print(alunos_menor_nota[['Nome', 'Nota Geral']].round(2))

# Encontra a menor nota no DataFrame
menor_nota = df_notas.iloc[:, 3:].min().min()

# Encontra a matéria e sala com a menor nota
materia_menor_nota = df_notas.iloc[:, 3:].min().idxmin()
sala_menor_nota = df_notas[df_notas[materia_menor_nota] == menor_nota]['Sala'].iloc[0]

# Encontra a matéria e sala com a menor nota
materia_menor_nota = df_notas.iloc[:, 3:].min().idxmin()
sala_menor_nota = df_notas[df_notas[materia_menor_nota] == menor_nota]['Sala'].iloc[0]
serie_menor_nota = df_notas[df_notas[materia_menor_nota] == menor_nota]['Serie'].iloc[0]

print("\nMatéria com Menor Nota:", materia_menor_nota)
print("Sala com Menor Nota:", sala_menor_nota)
print("Serie com Menor Nota:", serie_menor_nota)


# Substitui valores vazios por 0
df_notas = df_notas.fillna(0)

# Calcula a média por matéria
medias_materias = df_notas.iloc[:, 3:].mean()
menor = medias_materias[-2]
nome_materia = medias_materias.index[-2]

# Cria uma coluna "Nota Geral" com a média das notas
df_notas['Nota Geral'] = df_notas.iloc[:, 3:].mean(axis=1)

# Ordena o DataFrame pela "Nota Geral" e pega os 3 alunos com as menores notas
alunos_menor_nota = df_notas.sort_values(by='Nota Geral').head(3)

# Exibe os alunos com a menor nota geral
print("\n3 Alunos com Menor Nota Geral:")
print(alunos_menor_nota[['Nome', 'Nota Geral']].round(2))

# Encontra a menor nota no DataFrame
menor_nota = df_notas.iloc[:, 3:].min().min()


print(f"A matéria '{nome_materia}' tem a penúltima menor média, com valor {menor:.2f}.(Ultimo é a Média Geral)")



#-------------------------------------------------

# Criando uma lista para armazenar as médias das séries
medias_por_serie = []

# Iterando sobre as séries únicas no DataFrame
for serie in df_notas['Serie'].unique():
    # Filtrando o DataFrame para obter apenas os alunos da série atual
    alunos_serie = df_notas[df_notas['Serie'] == serie]
    # Calculando a média das notas de todas as matérias para a série atual
    media_serie = alunos_serie.iloc[:, 3:].mean().mean()
    # Adicionando a média da série à lista
    medias_por_serie.append(media_serie)

# Encontrando a menor média
menor_media = min(medias_por_serie)

# Encontrando os índices das séries com a menor média
indices_series_menor_media = [i for i, media in enumerate(medias_por_serie) if media == menor_media]

# Encontrando os nomes das séries com a menor média
series_menor_media = df_notas['Serie'].unique()[indices_series_menor_media].tolist()



print("Séries com a Menor Média:", series_menor_media)

Define qual será a entrada do Prompt do Gemini



prompt = f"""
A menor nota encontrada foi {menor:.2f} na matéria {nome_materia} do ({serie_menor_nota}).

Crie um relatório completo sobre o desempenho dos alunos, destacando os alunos com dificuldades e as áreas que precisam de mais atenção. Inclua também sugestões de como melhorar o desempenho geral da turma.
"""

print(prompt)

response = model.generate_content(prompt)
print(response.text)
