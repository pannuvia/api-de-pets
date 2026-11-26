# Cadastro de Pets API

API REST simples para cadastro de usuários e pets, criada para estudos de testes e automação em nível de API.

O projeto usa Express, autenticação JWT, documentação Swagger e armazenamento em memória. Ao reiniciar a aplicação, os registros criados são perdidos.

## Tecnologias

- Node.js
- Express
- JWT
- Swagger UI
- Mocha, Chai, Sinon e Supertest

## Estrutura

```text
src/
  model/      Modelos e armazenamento em memória
  service/    Regras de negócio da aplicação

rest/
  app.js
  server.js
  auth.js
  swagger.js
  controller/

test/
  rest/
```

## Instalação

```powershell
npm install
```

## Como Rodar

```powershell
npm start
```

Também é possível usar o script específico da API REST:

```powershell
npm run start-rest
```

A API roda por padrão em:

```text
http://localhost:3000
```

A documentação Swagger fica disponível em:

```text
http://localhost:3000/api-docs
```

## Autenticação

As rotas públicas não exigem token:

- `POST /register`
- `POST /login`
- `GET /status`

As demais rotas exigem o header:

```text
Authorization: Bearer <token>
```

## Endpoints

### Status

```http
GET /status
```

Retorna o status da API e não exige autenticação.

Exemplo de resposta:

```json
{
  "status": "Operacional",
  "version": "1.0.0",
  "database": {
    "status": "Operacional",
    "maxConnections": 900
  }
}
```

### Usuários

```http
POST /register
POST /login
GET /users
```

### Pets

```http
POST /pets
GET /pets
GET /mypets
DELETE /pets/:id
```

O endpoint `GET /mypets` permite filtros opcionais por query string:

```http
GET /mypets?name=Rex
GET /mypets?type=Cachorro
GET /mypets?name=Rex&type=Cachorro
```

## Dados Iniciais

Ao iniciar a aplicação, alguns usuários e pets já vêm pré-cadastrados em memória.

### Usuários

| id | username | password |
| --- | --- | --- |
| 1 | `tiago` | `123456` |
| 2 | `ricardo` | `456789` |

As senhas são armazenadas no model em formato de hash usando `bcryptjs`, mas podem ser usadas em texto puro no login.

### Pets

| id | userId | name | type |
| --- | --- | --- | --- |
| 1 | 1 | `Luke` | `Cão` |
| 2 | 1 | `Maurício` | `Jabuti` |
| 3 | 2 | `Pandora` | `Gato` |

O campo `userId` indica a qual usuário cada pet pertence.

## Testes

O projeto possui testes de controller e testes externos para a API REST.

Para testes externos, crie um arquivo `.env` na raiz do projeto:

```env
BASE_URL_REST=http://localhost:3000
```

Comandos disponíveis:

```powershell
npm run test-rest-controller
npm run test-rest-external
```

Antes de executar os testes externos, suba a API REST.

## Teste com JMeter

O plano disponível é `projetos/apiDePets-aulas.jmx`. Ele define as variáveis `protocol=http`, `baseUrl=localhost` e `port=3000`.

O único grupo de threads ativo é um **Thread Group** padrão, configurado com 1 usuário, ramp-up de 1 segundo e 1 iteração. Em cada execução, ele:

1. Faz `POST /login` com o usuário `tiago` e a senha `123456`.
2. Extrai `$.token` da resposta para a variável `tokenExtraido`.
3. Faz `POST /pets` com o token Bearer e o corpo `{"name":"pretinha","type":"gato"}`.

Para executar, inicie a API com `npm start`, abra `projetos/apiDePets-aulas.jmx` no JMeter 5.6.3 e clique em **Start**. O listener ativo do grupo é **View Results Tree**. O **Summary Report** está desabilitado nesse grupo.

### Limitações do plano atual

- O grupo `jp@gc - Ultimate Thread Group` está desabilitado, então não executa carga adicional.
- O fluxo ativo não faz `GET /pets` e não configura modelos de carga aberta e fechada.
- As asserções de resposta do fluxo ativo estão desabilitadas; em particular, `POST /pets` deveria retornar `201`.
- O nome do pet é fixo (`pretinha`), então execuções repetidas sem reiniciar a API podem receber `409` se esse pet já existir para o usuário.

Assim, este JMX documenta o fluxo que está ativo no arquivo, mas ainda não cobre sozinho todos os requisitos do exercício de carga.

## Observações

- O banco em memória deixa a aplicação leve e previsível para estudos.
- Como os dados são reiniciados a cada execução, os cenários de teste podem criar massa sem depender de um banco real.
- A camada `src/service` concentra as regras de negócio usadas pelos controllers REST.
