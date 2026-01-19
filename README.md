# Deepnote

- Executando python online por questões de segurança para usuário final

### Getting started Docker


```
services:
  jupyter:
    image: python:3.12.8-slim
    container_name: deepnote_jupyter
    volumes:
      - .:/app
    working_dir: /app
    ports:
      - "0.0.0.0:22222:8888"
    environment:
      - PYTHONDONTWRITEBYTECODE=1
      - PYTHONUNBUFFERED=1
    command: >
      sh -c "pip install --no-cache-dir 
      deepnote-toolkit jupyterlab jupyterlab-deepnote 
      pandas openpyxl numpy fastapi uvicorn python-multipart && 
      jupyter lab 
      --ip=0.0.0.0 
      --port=8888 
      --no-browser 
      --allow-root 
      --ServerApp.token='' 
      --ServerApp.password='' 
      --ServerApp.allow_origin='*' 
      --ServerApp.check_origin=False 
      --ServerApp.disable_check_xsrf=True 
      --ServerApp.allow_remote_access=True 
      --ServerApp.base_url='/python/' 
      --LabApp.base_url='/python/'"
```

### Ler Planilha com 2 Colunas com valores inteiros

```
import pandas as pd

# O parâmetro header=None diz que a planilha NÃO tem títulos nas colunas
# O parâmetro names=['A', 'B'] dá nome às colunas para o código funcionar
df = pd.read_excel('dados.xlsx', header=None, names=['A', 'B'])

# Agora o restante do seu loop deve funcionar perfeitamente:
df['Multiplicacao'] = 0.0

for i in df.index:
    valor_a = df.loc[i, 'A']
    valor_b = df.loc[i, 'B']
    resultado = valor_a * valor_b
    df.at[i, 'Multiplicacao'] = resultado

display(df)
```
