
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

## Explicação das Funções no Script

### 1. `renovar_token(refresh_token)`

**Descrição**: Renova o token de acesso (`access_token`) usando o `refresh_token`.

- **Entrada**: 
  - `refresh_token`: Token usado para obter um novo token de acesso.

- **Processo**:
  1. Faz uma requisição POST para o endpoint de autenticação da API do Mercado Livre.
  2. Envia os parâmetros necessários: `grant_type`, `client_id`, `client_secret` e `refresh_token`.
  3. Retorna o novo `access_token` e `refresh_token` se a requisição for bem-sucedida.

- **Saída**: 
  - Um novo `access_token` e um `refresh_token`, ou `None` se ocorrer um erro.

---

### 2. `buscar_mais_vendidos(access_token, refresh_token)`

**Descrição**: Obtém os produtos mais vendidos de uma categoria específica.

- **Entrada**: 
  - `access_token`: Token de acesso válido para autorização na API.
  - `refresh_token`: Usado para renovar o token de acesso caso esteja expirado.

- **Processo**:
  1. Envia uma requisição GET para o endpoint de busca de produtos (`/sites/MLB/search`) com o `access_token` no cabeçalho.
  2. Caso o token de acesso esteja expirado, chama a função `renovar_token` para obter um novo token.
  3. Exibe os produtos mais vendidos no console em formato JSON indentado.
  4. Chama a função `salvar_mais_vendidos_csv` para salvar os resultados em um arquivo CSV.

- **Saída**: 
  - Produtos mais vendidos exibidos no console.
  - Resultados salvos em um arquivo CSV.

---

### 3. `salvar_mais_vendidos_csv(mais_vendidos, file_name)`

**Descrição**: Salva os produtos mais vendidos em um arquivo CSV.

- **Entrada**: 
  - `mais_vendidos`: Dados dos produtos mais vendidos em formato JSON.
  - `file_name`: Caminho do arquivo CSV onde os dados serão salvos.

- **Processo**:
  1. Extrai os dados relevantes dos produtos usando `pandas.json_normalize`.
  2. Salva os dados em um arquivo CSV no local especificado com o delimitador `;`.

- **Saída**: 
  - Um arquivo CSV contendo as informações dos produtos mais vendidos.

---

## Exemplo de Uso

Copie o código abaixo para um arquivo Python e execute-o.

```python
# Exemplo de uso das funções
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
