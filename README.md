# Plataforma Águia

A Plataforma Águia disponibiliza API para acesso a serviços via aplicativos de terceiros e interação com a plataforma de operações de campo.

Veja abaixo como implementar a integração com a Plataforma Águia em seu sistema ou site.

# API de Comunicação com

A API de comunicaçãom com o sistema água usa o protocolo HTTPS e a especificação JSON-RPC.

Abaixo, instruções sobre como ter acesso a API.

# Endpoint

O endpoint de acesso a API é: https://api.sistemaaguia.com/ veja abaixo como acessar via GET e POST.

# Autenticação e sequencia de acesso:

Plataforma Águia usa o conceito de autorização e autenticação. 

Primeiro, você deve autorizar seu dispositivo para então prosseguir com a autenticação.

Portanto, você deve:
- Invocar AuthorizeDevice para autorizar o dispositivo.
- Invocar AuthorizeUser para autenticar o usuário.
- A partir desse ponto, você pode invocar outros métodos restritos da Plataforma Águia.

# Restrições de Segurança:

- métodos ```api_public_*```: podem ser invocados sem autenticação.
- métodos ```api_account_*```: existe autorização de dispositivo e autenticação de usuário para acesso.

# Instruções de Gestão de Segurança na Plataforma Águia:

- Após autorizar cadastrar o dispositivo, é necessário que o gestor autorize o dispositivo para autenticação e acesso a serviços via API.
- No Painel de Gerenciamento vá em Autenticação e Autorizações e pesquise por novos dispositivos.

# JSON-RPC

- O gateway de API usa o formato jsonrpc para intercâmbio de dados.
- A invocação de méotodos jsonrpc pode ser via POST e GET, veja abaixo como invocar e a resposta esperada.

# Formato de invocação com GET:

O formato para call via GET é:

```
/api/v1.0/{method}/?{INDEX0}={VALOR0}&{INDEX1}={VALOR1}&{INDEX2}={VALOR2}&id={RANDOM}
```

Onde:

- {method}: indica o recurso a ser acessado, veja a lista abaixo de métodos disponíveis.
- {INDEX0}: a lista de parâmetros é sequencia, o nome dos parâmetros devem ser: 0, 1, 2 e assim por diante.
- {id}: caso você queria identificar sua requisição para tratar o resultado, o id sempre é retornado.

# Exemplos:

Exemplo, envie a seguinte requisição GET:

https://api.sistemaaguia.com/api/v1/AuthorizeDevice/55555555555555/nome/IMEI-02932392832DFKDFDHF?id=1

Veja logo abaixo o resulado esperado.

# Formato de invocação com POST:

Você pode enviar uma requisição POST para o seguinte endereço:

Endpoint POST: https://api.sistemaaguia.com/api/v1.0

Com o seguinte corpo:

```
{"jsonrpc": "2.0", "method": "{method}", "params": ["{VALOR0}", "{VALOR1}", "{VALOR2}"], "id": "{RANDOM}" }
```

Exemplo:

```
{"jsonrpc": "2.0", "method": "AuthorizeDevice", "params": ["55555555555555", "nome", "IMEI-02932392832DFKDFDHF"], "id": "2" }
```

# Resultado tanto para GET quanto POST:
```
{
          "jsonrpc": "2.0",
          "method": "AuthorizeDevice",
          "params": {
                    "success": true,
                    "result": [
                              {
                                        "coderro": 1,
                                        "msgerro": "Dispositivo não habilitado!",
                                        "cnpj": "55555555555555",
                                        "imei": "nome"
                              }
                    ]
          }
}
```

Descrição dos campos de resultado:

- ```method```: o método requisitado ou método de erro de sistema.
- ```success```: indica que a API foi executada com suces e você tem result.
- ```result```: é sempre um array, com o resultado na execução do serviço.
- ```result[0].coderro```: se 0 indica sucess, se maior que um, ouve erro no processamento dos dados.
- ```result[0].msgerro```: mensagem de erro para exibir ao usuário.

Métodos de erro que podem se retornados:
 
- ```AuthorizeDevice```: erro na autorizácão do dispositivo.
- ```noMethodId```: método não foi informado.
- ```invalidPrefix```: falta prefixo de seguraça api_ na requisição.
- ```accessDenied```: Requisição tentando acesso a métodos para somente usuário autenticado, mas, sem autenticar.

# Métodos Disponíveis:

## Autorizar Dispositivo

* Essa é a primeira API que deve ser executada para registrar o dispositivo na conta corporativa da empresa.
* Após receber o retorno, guarde o cookie da sessão e envie sempre para identificação do dispositivo.
* Normalmente, os SDK HTTPS de plataformas já tem opção para armazenar e enviar cookies automaticamente.

### /api/v1.0/AuthorizeDevice/{cnpj}/{unidade}/{imei}

- ```{cnpj}```: cnpj, somente números da empresa.
- ```{unidade}```: login da unidade organizacional solicitando a auotrização, peça ao seu gerente de operações.
- ```{imei}```: IMEI ou identificador único como um UUID do dispositivo. Preferencialmente, aplice o MD5 antes de enviar.
- Opcional: ```{id}```: identificador da requisição. O valor sempre retorna na requisição.

## Exemplo:

https://api.sistemaaguia.com/api/v1/AuthorizeDevice/55555555555555/nome/IMEI-122233334444FKDFDHF

Dispositivo autorizado na plataforma, prossiga para a tela de login:

```
{
          "jsonrpc": "2.0",
          "method": "AuthorizeDevice",
          "params": {
                    "success": true,
                    "result": [
                              {
                                        "coderro": 0,
                                        "msgerro": "Dispositivo Permitido!",
                                        "cnpj": "55555555555555",
                                        "imei": "IMEI-02932392832DFKDFDHF"
                              }
                    ]
          }
}
```

https://api.sistemaaguia.com/api/v1/AuthorizeDevice/55555555555555/nome/IMEI-122233334444FKDFDHFx

Dispositivo cadastro, mas, ainda não habilitado, favor habilita na plataforma:

```
{
          "jsonrpc": "2.0",
          "method": "AuthorizeDevice",
          "params": {
                    "success": true,
                    "result": [
                              {
                                        "coderro": 1,
                                        "msgerro": "Dispositivo não habilitado!",
                                        "cnpj": "55555555555555",
                                        "imei": "nome"
                              }
                    ]
          }
}
```


## Autorizar Usuário

* Essa é a segunda API que deve ser executada para autenticar o usuário.
* Após autenticado, guarde os cookies recebidos e os envie sempre na requisição. Normalmente é o padrão para o SDK HTTPS desde que você configure um storage para os cookies. Vejas as opções do seu SDK.
* A partir desse ponto, se sucesso, você está autenticado e já pode invocar outras API abaixo.

https://api.sistemaaguia.com/api/v1/AuthorizeUser/nome1/dfafiadkjfadkfhadkfahdfkadaf?id=2

### /api/v1.0/AuthorizeUser/{login}/{hash-md5-da-senha}

- ```{login}```: login da unidade organizacional solicitando a auotrização, peça ao seu gerente de operações.
- ```{hash-md5-da-senha}```: Aplique o hash MD5 na senha antes de enviar.
- Opcional: ```{id}```: identificador da requisição. O valor sempre retorna na requisição.

Exemplo:

https://api.sistemaaguia.com/api/v1/AuthorizeUser/nome/FFC58105BF6F8A91ABA0FA2D99E6F106?id=2

*Exemplo de usuário autorizado:*

```
{
          "jsonrpc": "2.0",
          "method": "AuthorizeUser",
          "id": "2",
          "params": {
                    "success": true,
                    "result": [
                              {
                                        "coderro": 0,
                                        "msgerro": "Acesso permitido!",
                                        "razao_social": "EMPRESA TESTE",
                                        "apelido": "NOME",
                                        "cnpj": "55555555555555",
                                        "imei": "IMEI-02932392832DFKDFDHF",
                                        "p3": ""
                              }
                    ]
          }
}
```

A partir desse ponto, prossiga para a aplicação e invoque outras procedures.


*Exemplo de usuário não autorizado:*

```
{
          "jsonrpc": "2.0",
          "method": "AuthorizeUser",
          "id": "2",
          "params": {
                    "success": true,
                    "result": [
                              {
                                        "coderro": 1,
                                        "msgerro": "Acesso não permitido!",
                                        "razao_social": null,
                                        "apelido": null,
                                        "cnpj": "55555555555555",
                                        "imei": "nome1",
                                        "p3": "nome1"
                              }
                    ]
          }
}
```

# 

### /api/v1.0/api_account_sp_mobile_autoriza_abastecimento?0={qrcode_cliente}&1={qrcode_bico}&2={placa_cliente}&3={url_foto}

- ```{login}```: login da unidade organizacional solicitando a auotrização, peça ao seu gerente de operações.
- ```{hash-md5-da-senha}```: Aplique o hash MD5 na senha antes de enviar.
- Opcional: ```{id}```: identificador da requisição. O valor sempre retorna na requisição.

Exemplo:

https://api.sistemaaguia.com/api/v1/api_account_sp_mobile_autoriza_abastecimento?0=1&1=2&2=aaa-123&3=0a0dfk2fdkfjadkfadf.jpg

Params:

- ```0```: qrcode_cliente varchar(100) - id do cliente caputrado via qrcode.
- ```1```: qrcode_bico varchar(100) - bico da bomba caputrado via qrcode.
- ```2```: placa_cliente varchar(20) - placa capturada via foto (reconhecimento).
- ```3```: url_foto varchar(200) - se foto foi digitada, fazer upload da foto para o s3 e informar o id da imagem.
