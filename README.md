
# Script para Integração com a API do Mercado Livre

Este script Python permite autenticar na API do Mercado Livre, renovar tokens e buscar os produtos mais vendidos de uma categoria específica. Ele também salva os resultados em um arquivo CSV.

---

## Pré-requisitos

Certifique-se de que você tem:

- Python 3.7+ instalado.
- A biblioteca `requests` instalada:
  ```bash
  pip install requests
  ```
- A biblioteca `pandas` instalada:
  ```bash
  pip install pandas
  ```
- Uma aplicação criada no [Mercado Livre Developer Center](https://developers.mercadolivre.com.br/pt_br/api-docs-pt-br) para obter:
  - **Client ID**
  - **Client Secret**
  - **Access Token** e **Refresh Token**.

---

## Como Obter o Access Token e o Refresh Token

Para gerar o `access_token` e o `refresh_token`, siga os passos abaixo:

1. **Obtenha o código de autorização:**
   Gere o link de autorização substituindo os valores abaixo:
   ```
   https://auth.mercadolivre.com.br/authorization?response_type=code&client_id=SEU_CLIENT_ID&redirect_uri=SEU_REDIRECT_URI
   ```
   - Substitua `SEU_CLIENT_ID` pelo seu Client ID.
   - Substitua `SEU_REDIRECT_URI` pelo Redirect URI configurado na sua aplicação.

   Acesse este link no navegador. Após autorizar a aplicação, você será redirecionado para o `redirect_uri` com um parâmetro `code` na URL, como no exemplo:
   ```
   https://seu_callback_url.com?code=SEU_AUTHORIZATION_CODE
   ```

2. **Troque o código de autorização pelo token de acesso:**
   Use o código retornado (`SEU_AUTHORIZATION_CODE`) para obter o `access_token` e o `refresh_token` com o seguinte script:

   ```python
   import requests

   client_id = "SEU_CLIENT_ID"
   client_secret = "SEU_CLIENT_SECRET"
   redirect_uri = "SEU_REDIRECT_URI"
   authorization_code = "SEU_AUTHORIZATION_CODE"

   url = "https://api.mercadolibre.com/oauth/token"
   payload = {
       "grant_type": "authorization_code",
       "client_id": client_id,
       "client_secret": client_secret,
       "redirect_uri": redirect_uri,
       "code": authorization_code
   }
   response = requests.post(url, data=payload)

   if response.status_code == 200:
       tokens = response.json()
       print("Access Token:", tokens["access_token"])
       print("Refresh Token:", tokens["refresh_token"])
   else:
       print("Erro:", response.json())
   ```

3. Salve os valores de `access_token` e `refresh_token` retornados.

---

## Configuração

Substitua as seguintes variáveis no script principal pelos valores reais da sua aplicação:

- **`SEU_CLIENT_ID`**: O ID da sua aplicação no Mercado Livre.
- **`SEU_CLIENT_SECRET`**: O segredo da sua aplicação.
- **`SEU_ACCESS_TOKEN`**: O token de acesso inicial.
- **`SEU_REFRESH_TOKEN`**: O token de renovação inicial.

---

## Uso

Copie o código abaixo para um arquivo Python e execute-o.

```python
import requests
import json
import pandas as pd

# Configurações
categoria_id = 'MLB1144'  # Eletrodomésticos
access_token = 'SEU_ACCESS_TOKEN'
refresh_token = 'SEU_REFRESH_TOKEN'

# Função para renovar o access token usando o refresh token
def renovar_token(refresh_token):
    url = 'https://api.mercadolibre.com/oauth/token'
    dados = {
        'grant_type': 'refresh_token',
        'client_id': 'SEU_CLIENT_ID',
        'client_secret': 'SEU_CLIENT_SECRET',
        'refresh_token': refresh_token
    }
    resposta = requests.post(url, data=dados)
    print("Status Code:", resposta.status_code)
    print("Response Body:", resposta.text)
    if resposta.status_code == 200:
        novo_token = resposta.json()
        return novo_token['access_token'], novo_token['refresh_token']
    else:
        print(f"Erro ao renovar token: {resposta.status_code}, {resposta.text}")
        return None, None

# Função para buscar produtos mais vendidos
def buscar_mais_vendidos(access_token, refresh_token):
    headers = {
        'Authorization': f'Bearer {access_token}'
    }
    produtos_url = f'https://api.mercadolibre.com/sites/MLB/search?category={categoria_id}&sort=sold_quantity&limit=50'
    top_produtos = requests.get(produtos_url, headers=headers)

    # Verifica se o access token expirou
    if top_produtos.status_code == 401:
        print("Token expirado, tentando renovar...")
        access_token, refresh_token = renovar_token(refresh_token)
        if access_token:
            headers = {
                'Authorization': f'Bearer {access_token}'
            }
            top_produtos = requests.get(produtos_url, headers=headers)
        else:
            print("Falha ao renovar o token.")
            return

    if top_produtos.status_code == 200:
        mais_vendidos = top_produtos.json()
        print("Top produtos mais vendidos:",
              json.dumps(mais_vendidos['results'], indent=4))

        # Salvar os produtos mais vendidos em formato CSV
        salvar_mais_vendidos_csv(mais_vendidos)
    else:
        print(f"Erro ao buscar os produtos mais vendidos: {top_produtos.status_code}, {top_produtos.text}")

# Função para salvar os produtos mais vendidos em formato CSV
def salvar_mais_vendidos_csv(mais_vendidos, file_name=r'C:\caminho_para_salvar\vendidos.csv'):
    # Convertendo os dados relevantes em um DataFrame
    produtos = mais_vendidos['results']
    df = pd.json_normalize(produtos)  # Normaliza o JSON para transformar em DataFrame
    # Salvando o CSV com delimitador ;
    df.to_csv(file_name, index=False, sep=';')
    print(f"Dados dos produtos mais vendidos salvos como {file_name} com delimitador ';'.")

# Exemplo de uso
buscar_mais_vendidos(access_token, refresh_token)
```

---

## Resultados

O script salvará um arquivo CSV com as informações dos produtos mais vendidos, incluindo:

- Nome do produto
- Preço
- Quantidade de vendas
- Outros detalhes relevantes.

---

## Licença

Este projeto é licenciado sob a **MIT License** - veja o arquivo [LICENSE](LICENSE) para detalhes.
