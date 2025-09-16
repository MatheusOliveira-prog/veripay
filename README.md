1. Introdução ao Projeto
   
O VeriPay é um aplicativo móvel com um backend robusto, projetado para aumentar a segurança em transações de e-commerce e entre pessoas (P2P). O projeto unifica duas ideias centrais de segurança:

PayGuard: Um sistema de pagamento em custódia (escrow) que protege tanto o comprador quanto o vendedor, garantindo que o pagamento só seja liberado após a confirmação do recebimento e da qualidade do produto.

ShopSafe AI: Uma camada de inteligência artificial que analisa transações e links de e-commerce em tempo real para detectar comportamentos fraudulentos e links maliciosos, fornecendo uma pontuação de risco para o usuário.
O projeto consiste em um backend construído com Python e Flask e um aplicativo nativo para Android construído com Kotlin e Jetpack Compose.

2. Arquitetura da Solução
O sistema é dividido em três componentes principais que se comunicam via uma API RESTful e WebSockets:
Backend (Servidor Python):
Serve como o "cérebro" da aplicação.
Gerencia o banco de dados (usuários, transações, mensagens).
Processa a lógica de negócio (autenticação, fluxo de escrow).
Realiza as análises de risco (IA) e de links.
Fornece um painel de administração para gerenciamento manual.
Frontend (Aplicativo Android):
É a interface com a qual o usuário interage.
Consome a API RESTful do backend para todas as operações.
Estabelece uma conexão WebSocket para a funcionalidade de chat em tempo real.
Implementa uma interface moderna e reativa, seguindo a arquitetura MVVM.
Banco de Dados (SQLite/PostgreSQL):
Armazena de forma persistente todos os dados da aplicação. Projetado para ser facilmente migrado entre SQLite (desenvolvimento) e PostgreSQL (produção).

3. Documentação do Backend (app.py)
O backend é construído com o micro-framework Flask, escolhido pela sua simplicidade e flexibilidade.
Tecnologias Utilizadas:

Flask: Framework web.

Flask-SQLAlchemy: ORM para interação com o banco de dados.

Flask-Bcrypt: Criptografia de senhas.

Flask-SocketIO: Habilita comunicação em tempo real (WebSockets) para o chat.

PyJWT: Geração e validação de JSON Web Tokens para autenticação.

Gunicorn/Eventlet: Servidor WSGI de produção.

Requests & BeautifulSoup4: Para a análise de links externos.

Estrutura do Banco de Dados (Modelos):

User: Armazena dados do usuário. A senha é sempre armazenada como um hash bcrypt.

Transaction: O coração do sistema escrow. Contém descrição, valor, status e as chaves estrangeiras para o vendedor e o comprador.

ChatMessage: Armazena uma única mensagem, ligada a uma transação e a um autor.

BehavioralEvent: Registra eventos de comportamento do usuário (ex: tempo de preenchimento de formulário) para a análise de risco da IA.

Endpoints da API (Rotas RESTful):

POST /auth/register: Cria um novo usuário.

POST /auth/login: Autentica um usuário e retorna um access_token JWT.

POST /transactions: Cria uma nova transação.

GET /transactions: Lista as transações do usuário logado.

GET /transaction/<id>: Retorna os detalhes de uma transação específica.

POST /transaction/<id>/[action]: Endpoints para as ações do escrow (accept, pay, ship, confirm, dispute).

POST /events: Registra um evento de comportamento para a IA.

GET /transaction/<id>/risk_score: Calcula e retorna a pontuação de risco de uma transação.

POST /verify-link: Analisa um link externo e retorna uma análise de segurança.

Eventos de WebSocket (Chat):

join_room: Conecta o cliente à sala de chat de uma transação específica.

send_message: Recebe uma nova mensagem do cliente, salva no banco de dados e a retransmite para todos os participantes da sala.

Painel de Administração:

Uma interface web simples, construída com Flask e HTML/Bootstrap, acessível em /admin.
Permite a criação, visualização, edição e deleção (CRUD) de usuários, transações e mensagens de chat.
Protegido por autenticação básica (root/root).

5. Documentação do Frontend (Aplicativo Android)
O aplicativo foi desenvolvido com foco em uma arquitetura moderna e manutenível.
Tecnologias Utilizadas:

Kotlin: Linguagem de programação principal.

Jetpack Compose: Framework declarativo para a construção de toda a UI.

MVVM (Model-View-ViewModel): Arquitetura padrão que separa a lógica da interface.

Compose Navigation com Animações: Para uma navegação fluida entre as telas.

Retrofit: Cliente HTTP para comunicação com a API RESTful.

Socket.IO Client: Para a comunicação em tempo real do chat.

Coroutines & StateFlow: Para gerenciamento de estado e operações assíncronas.

Estrutura do Projeto:

data: Contém ApiService (definição das rotas), ApiModels (modelos de dados), TokenManager (gerenciamento do JWT), SocketManager (gerenciamento do WebSocket), e AuthInterceptor (adiciona o 
token a todas as chamadas).

ui: Contém todos os @Composable que formam as telas do aplicativo, separados por funcionalidade (auth, home, detail, chat, etc.).

viewmodel: Contém as classes ViewModel para cada tela, que orquestram a lógica e os dados.

Principais Funcionalidades Implementadas:

Login Persistente: O app verifica a validade de um token JWT salvo. Se for válido, o usuário é direcionado para a tela principal, caso contrário, para a de login.

Tela Principal com Abas: Uma MainScreen organiza o conteúdo em duas abas: "Transações" e "Verificar Links".

Lista de Transações: Exibe as transações em seções ("Ações Pendentes" e "Histórico") e usa ícones para indicar o papel do usuário.

Tela de Detalhes: Mostra todas as informações de uma transação e os botões de ação apropriados para o estado e o usuário atual.

Linha do Tempo e Análise de Risco: Na tela de detalhes, um componente visual mostra o progresso do status da transação. Para o vendedor, um card animado exibe a pontuação de risco da IA.

Fluxo de Criação: Telas dedicadas para a criação de novas transações.

Chat em Tempo Real: Uma tela de chat funcional que se atualiza instantaneamente com novas mensagens via WebSockets.

Verificador de Links: Uma tela dedicada onde o usuário pode colar URLs externas e receber uma análise de segurança completa, alimentada pelo backend.
