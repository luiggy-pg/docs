# Chat
Aplicativo de chat baseado no ChatGPT.

## Sumário:
- [Sobre](#sobre);
- [Inicialização](#inicialização);
- [Ambiente](#ambiente);
- [Rotas](#rotas).

## Sobre
Aplicativo de chat baseado no ChatGPT utilizando Solid.js, com controle de uso de tokens por usuário e integração com o Pipedrive.

## Inicialização
Para uma deploy local, certifique-se de incluir as [variáveis de ambiente](#ambiente) necessárias.

Construindo o aplicativo (/app/):
```sh
vite build
```

Inicializando o aplicativo (/server/):
```sh
node main.js
```

## Ambiente
Variáveis de ambiente usadas pelo projeto.
- `ADDRESS`: Endereço de ligação. 
    - Obrigatório.
    - Normalmente `127.0.0.1`.
- `PORT`: Porta de ligação. 
    - Obrigatório.
    - Normalmente `3000`. 
- `DB_URL`: URL do banco de dados. 
    - Obrigatório.
    - URL para o banco de dados MongoDB, normalmente `mongodb://localhost:27017`.
- `DIST_URL`: URL de distribuição.
    - Obrigatório.
    - Localização na qual o chat estará rodando.
- `LOGIN_URL`: URL de login.
    - Obrigatório.
    - Localização para onde o usuário será redirecionado em caso de credenciais inválidas.
- `INT_URL`: URL de integração. 
    - Obrigatório.
    - Localização para a integração principal.
- `SHARED_SECRET`: Segredo Compartilhado. 
    - Obrigatório.
    - Token usado para verificação da assinatura JWT.
- `OAI_ASSISTANT`: Assistente OpenAI. 
    - Obrigatório.
    - ID do Assistente OpenAI sobre o qual o Chat será baseado.
- `OAI_TOKEN`: Token OpenAI.
    - Obrigatório.
    - Token da API OpenAI.
- `PIPEDRIVE_TOKEN`: Token Pipedrive. 
    - Obrigatório.
    - Token da API Pipedrive.

## Rotas
Rotas do projeto, podendo ser divididas em duas rotas principais:
- Chat: `/server/routes/chat/index.js`;
- Dashboard: `/server/routes/dashboard/index.js`.

Ambas são importadas pelo arquivo principal do servidor, `/server/main.js`.

![](docs/static/routes.svg)

### /chat/thread

**GET** `/chat/thread`

**Descrição:**  
Cria um novo thread ou retorna um existente com base na sessão. Se o usuário não existir, criará um novo usuário com valores de token padrão. Se o saldo de tokens do usuário estiver esgotado, concederá novos tokens com base nas configurações de concessão de tokens.

**Parâmetros de Consulta:**
- `s` (string): O identificador da sessão.

**Respostas:**
- **200 OK**: Retorna os detalhes do tópico junto com as informações do usuário e dados da sessão.
- **401 Unauthorized**: Se a sessão for inválida ou não fornecida.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /chat/file

**GET** `/chat/file`

**Descrição:**  
Retorna um arquivo associado a um thread com base no ID do arquivo fornecido.

**Parâmetros de Consulta:**
- `id` (string): O ID do arquivo a ser recuperado.

**Respostas:**
- **200 OK**: Retorna o conteúdo do arquivo.
- **400 Bad Request**: Se o ID do arquivo não for fornecido ou for inválido.

### /chat/flag

**GET** `/chat/flag`

**Descrição:**  
Marca uma mensagem específica em um thread. Requer uma sessão válida e detalhes específicos da mensagem.

**Parâmetros de Consulta:**
- `t` (string): O ID do tópico.
- `m` (string): O ID da mensagem.
- `s` (string): O identificador da sessão.

**Respostas:**
- **200 OK**: Indica que a mensagem foi marcada com sucesso.
- **400 Bad Request**: Se os parâmetros de consulta obrigatórios estiverem ausentes.
- **401 Unauthorized**: Se a sessão for inválida ou não corresponder.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /chat/message

**GET** `/chat/message`

**Descrição:**  
Adiciona uma mensagem ao thread e a processa. Gerencia o uso de tokens e atualiza os tokens do usuário de acordo. Transmite a resposta de volta como eventos.

**Parâmetros de Consulta:**
- `t` (string): O ID do tópico.
- `m` (string): O conteúdo da mensagem.
- `s` (string): O identificador da sessão.

**Respostas:**
- **200 OK**: Transmite os eventos de resposta, incluindo deltas de mensagens e informações de uso.
- **400 Bad Request**: Se os parâmetros de consulta obrigatórios estiverem ausentes.
- **401 Unauthorized**: Se a sessão for inválida ou não corresponder.
- **429 Too Many Requests**: Se o saldo de tokens do usuário for insuficiente.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /dashboard/overall/usage

**GET** `/dashboard/overall/usage`

**Descrição:**  
Retorna estatísticas gerais de uso para um intervalo de datas específico, incluindo tokens usados e uso médio. Fornece detalhamentos por dia, mês, empresa e usuário.

**Parâmetros de Consulta:**
- `date_from` (string): A data inicial para as estatísticas de uso (formato ISO 8601).
- `date_to` (string): A data final para as estatísticas de uso (formato ISO 8601).

**Respostas:**
- **200 OK**: Retorna as estatísticas de uso e detalhamentos.
- **400 Bad Request**: Se os parâmetros de data forem inválidos.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /dashboard/overall/resume

**POST** `/dashboard/overall/resume`

**Descrição:**  
Gera um resumo das estatísticas de uso com base nos dados fornecidos. Utiliza um modelo GPT para criar uma visão geral narrativa das informações de uso.

**Corpo da Requisição:**
- `stats` (objeto): Os dados das estatísticas de uso.

**Respostas:**
- **200 OK**: Retorna o resumo gerado.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /dashboard/transactions

**GET** `/dashboard/transactions`

**Descrição:**  
Retorna uma lista de transações com opções de paginação e filtragem. Inclui estatísticas agregadas para o intervalo de datas especificado.

**Parâmetros de Consulta:**
- `limit` (número): O número de transações a serem retornadas.
- `start` (número): O número de transações a serem ignoradas (para paginação).
- `date_from` (string): A data inicial para as transações (formato ISO 8601).
- `date_to` (string): A data final para as transações (formato ISO 8601).

**Respostas:**
- **200 OK**: Retorna as transações e estatísticas agregadas.
- **400 Bad Request**: Se os parâmetros de consulta forem inválidos.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /dashboard/users

**GET** `/dashboard/users`

**Descrição:**  
Retorna uma lista de usuários com opções de paginação.

**Parâmetros de Consulta:**
- `limit` (número): O número de usuários a serem retornados.
- `start` (número): O número de usuários a serem ignorados (para paginação).

**Respostas:**
- **200 OK**: Retorna a lista de usuários.
- **400 Bad Request**: Se os parâmetros de consulta forem inválidos.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /dashboard/users/:id

**GET** `/dashboard/users/:id`

**Descrição:**  
Retorna detalhes de um usuário específico pelo ID.

**Parâmetros de Caminho:**
- `id` (string): O ID do usuário a ser recuperado.

**Respostas:**
- **200 OK**: Retorna os detalhes do usuário.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /dashboard/users/:id/tokens

**POST** `/dashboard/users/:id/tokens`

**Descrição:**  
Atualiza o saldo de tokens e a quantidade de concessão para um usuário específico.

**Parâmetros de Caminho:**
- `id` (string): O ID do usuário a ser atualizado.

**Corpo da Requisição:**
- `token_balance` (número): O novo saldo de tokens.
- `token_grant` (número): A nova quantidade de concessão de tokens.

**Respostas:**
- **200 OK**: Confirma que o saldo de tokens e a quantidade de concessão foram atualizados.
- **400 Bad Request**: Se os parâmetros necessários estiverem ausentes ou forem inválidos.
- **500 Internal Server Error**: Se ocorrer um erro durante a solicitação.

### /int/tokens

**POST** `/int/tokens`

**Descrição:**  
Atualiza ou cria um usuário com uma cota de tokens especificada com base no ID do usuário (integração).

**Parâmetros de Consulta:**
- `id` (string): O ID