# Deepnote

- Executando python online por questões de segurança para usuário final

### NGINX com o proxy configurado
```
location /python/ {
		    proxy_pass http://host-docker-do-jupyter:987321;
			
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
		}
```

### JupyterLab Standalone (Subir individualmente para teste sem autenticação)
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

### JupyterLab utilizando Autenticação via Keycloak

``` 
services:
  jupyter:
    image: python:3.12.8-slim
    container_name: deepnote_jupyter
    working_dir: /app
    volumes:
      - .:/app
    expose:
      - "8888"
    environment:
      PYTHONDONTWRITEBYTECODE: "1"
      PYTHONUNBUFFERED: "1"
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
      --ServerApp.trust_xheaders=True
      --ServerApp.disable_check_xsrf=True
      --ServerApp.allow_remote_access=True
      --ServerApp.base_url='/python/'"
    networks:
      - csc

  oauth2-proxy:
    image: quay.io/oauth2-proxy/oauth2-proxy:v7.7.1
    container_name: jupyter_oauth2_proxy
    depends_on:
      - jupyter
    ports:
      - "127.0.0.1:22222:4180"
    environment:
      OAUTH2_PROXY_PROVIDER: keycloak-oidc
      OAUTH2_PROXY_OIDC_ISSUER_URL: https://auth.meu-keycloak-no-nginx-server-proxy/auth/realms/meurealm

      OAUTH2_PROXY_CLIENT_ID: jupyterlab-hml
      OAUTH2_PROXY_CLIENT_SECRET: <CLIENT_SECRET>

      OAUTH2_PROXY_COOKIE_SECRET: <COOKIE_SECRET>
      OAUTH2_PROXY_COOKIE_SECURE: "true"
      OAUTH2_PROXY_COOKIE_DOMAIN: meu-nginx-server-proxy.com.br
      OAUTH2_PROXY_COOKIE_PATH: /python

      OAUTH2_PROXY_REVERSE_PROXY: "true"
      OAUTH2_PROXY_PROXY_PREFIX: /python/oauth2
      OAUTH2_PROXY_REDIRECT_URL: https://meu-nginx-server-proxy.com.br/python/oauth2/callback

      # upstream com /python/ porque o Jupyter está com base_url /python/
      OAUTH2_PROXY_UPSTREAMS: http://jupyter:8888/python/

      OAUTH2_PROXY_EMAIL_DOMAINS: "*"
      OAUTH2_PROXY_HTTP_ADDRESS: 0.0.0.0:4180

      OAUTH2_PROXY_PASS_ACCESS_TOKEN: "true"
      OAUTH2_PROXY_SET_AUTHORIZATION_HEADER: "true"
      OAUTH2_PROXY_SET_XAUTHREQUEST: "true"

      OAUTH2_PROXY_PROVIDER_CA_FILE: /etc/ssl/certs/ca-certificates.crt
      OAUTH2_PROXY_HTTP_CLIENT_TIMEOUT: 30s
    networks:
      - csc

networks:
  csc:
    external: true
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
