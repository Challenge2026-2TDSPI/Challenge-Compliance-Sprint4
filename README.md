# ChallengeAPI — Compliance, Quality Assurance & Tests

Entrega da Sprint 4 do Challenge 2026 — FIAP / 2TDS.

Projeto Clyvo / PathPet: API REST desenvolvida em ASP.NET Core 8 para gerenciamento de tutores, pets, vacinas e consultas, com autenticação JWT e integração com Oracle e MongoDB.

## Links da entrega

| Item | Link |
|---|---|
| Parte A — Plano de testes manuais | [Azure Test Plans — PathPet](https://dev.azure.com/PathPet/PathPet/_testPlans) |
| Nome do plano | **Sprint4 - QA** |
| Parte B — Vídeo da automação | [Configuração, execução e resultados no Postman](https://www.youtube.com/watch?v=8lEQJ2L4zrQ) |
| Repositório da entrega | [Challenge-Compliance-Sprint4 — develop](https://github.com/Challenge2026-2TDSPI/Challenge-Compliance-Sprint4/tree/develop) |
| Código-fonte da API | [ChallengeAPI — master](https://github.com/Challenge2026-2TDSPI/ChallengeAPI/tree/main) |
| API publicada | https://clyvovet-api-rm562156.azurewebsites.net |

O acesso ao projeto Azure DevOps exige autenticação e as permissões concedidas à conta do avaliador.

## Equipe

| Nome | RM |
|---|---|
| Eduardo Augusto de Oliveira Souza | RM565269 |
| Fellipe Costa de Oliveira | RM564673 |
| Felype Ferreira Maschio | RM563009 |
| Gustavo Vieira de Matos | RM563304 |
| Pedro Henrique dos Santos Costa | RM562156 |

## Parte A — Testes manuais de sistema

Os casos estão documentados no plano **Sprint4 - QA**, no Azure Test Plans.

A abordagem utiliza testes funcionais de sistema, com técnica de caixa-preta, baseados em cenários de uso e com dados de entrada e resultados esperados previamente definidos.

Cada caso apresenta:

- Objetivo da validação.
- Preparação e pré-condições.
- Dados de entrada.
- Procedimento de execução.
- Resultado esperado para cada passo.

### Organização do plano

| Suíte | Cobertura |
|---|---|
| 01 — Disponibilidade e Integrações | Saúde da API, conexão Oracle, conectividade MongoDB e operações com pets no MongoDB |
| 02 — Autenticação JWT | Login e controle de acesso à rota protegida |
| 03 — Tutores | Cadastro, consultas e validações |
| 04 — Pets | Cadastro, consultas e atualização |
| 05 — Vacinas e Consultas | Operações dos módulos de vacinação e atendimento |
| 06 — Exclusão e Integridade | Exclusões e verificação dos resultados |

As requisições manuais são realizadas contra a API publicada. Os resultados e as evidências das execuções são consultados no Azure Test Plans.

## Parte B — Automação de testes

**Ferramenta:** Postman Collection Runner.

A coleção contém **15 casos de teste e 41 asserções**, cobrindo disponibilidade, autenticação, tutores e pets.

O vídeo apresenta a configuração, a execução e os resultados da automação, com 41 asserções aprovadas na execução demonstrada.

### Massa de dados

- Tutor: `Tutor QA Automacao`
- Telefone: `11999990000`
- E-mail: `tutor.qa@petcare.com`
- Pet: `Thor`
- Espécie: `Cachorro`
- Raça: `Labrador`
- Idade inicial: `3`
- Idade após atualização: `4`
- Clínica informada no cadastro: `clinicaId = 1`

O token e os IDs gerados são armazenados nas variáveis `token`, `tutorId` e `petId` e reutilizados pelos casos seguintes.

### Casos automatizados

| ID | Objetivo | Entrada | Saída esperada |
|---|---|---|---|
| CT00 | Verificar API e Oracle | `GET /health` | HTTP 200 e `status: Healthy` |
| CT01 | Autenticar com credenciais válidas | `POST /api/Auth/login` | HTTP 200 e token JWT |
| CT02 | Rejeitar login inválido | Login com `senhaErrada123` | HTTP 401, sem token |
| CT03 | Bloquear acesso sem token | `GET /api/Auth/dados-protegidos` sem autenticação | HTTP 401 |
| CT04 | Permitir acesso com token | Mesma rota com Bearer token | HTTP 200 e mensagem identificando o usuário |
| CT05 | Cadastrar tutor | Dados controlados do tutor | HTTP 201, ID gerado, dados enviados e header Location |
| CT06 | Validar campos obrigatórios | Tutor sem nome e e-mail | HTTP 400 e erros dos campos ausentes |
| CT07 | Cadastrar pet | Dados do Thor e tutorId criado | HTTP 201 e dados correspondentes |
| CT08 | Consultar pet | `GET /api/Pets/{{petId}}` | HTTP 200 e pet correto |
| CT09 | Rejeitar ID divergente | PUT com ID da URL diferente do corpo | HTTP 400 |
| CT10 | Atualizar pet | PUT com idade 4 | HTTP 204 |
| CT11 | Confirmar persistência | GET após a atualização | HTTP 200 e idade 4 |
| CT12 | Excluir pet | `DELETE /api/Pets/{{petId}}` | HTTP 204 |
| CT13 | Consultar pet excluído | GET do pet removido | HTTP 404 |
| CT14 | Excluir tutor de teste | `DELETE /api/Tutores/{{tutorId}}` | HTTP 204 |

Do CT01 ao CT14, a coleção também verifica se o tempo de resposta é inferior ao limite configurado em `tempoMaximoMs`, inicialmente 2000 ms. O CT00 fica fora dessa medição.

Essa verificação mede o tempo de resposta das requisições; não simula carga ou usuários simultâneos.

## Arquivos

```text
README.md
postman/
├── ChallengeAPI-QA.postman_collection.json
└── ChallengeAPI-Local.postman_environment.json
```

## Como executar a automação

1. Importe os dois arquivos JSON da pasta `postman` no Postman.
2. Selecione o environment **ChallengeAPI - Local**.
3. Configure as variáveis:

| Variável | Valor |
|---|---|
| `baseUrl` | `https://clyvovet-api-rm562156.azurewebsites.net` para a API publicada ou `http://localhost:5241` para execução local |
| `usuarioAdmin` | `admin` |
| `senhaAdmin` | Senha do usuário de teste configurado na API |
| `tempoMaximoMs` | `2000` |

4. Abra o Collection Runner.
5. Execute a coleção completa, na ordem CT00 a CT14.
6. Confira os resultados das asserções.

A execução depende da API e do banco Oracle disponíveis. O CT00 verifica essa pré-condição e interrompe a sequência se o estado retornado não for `Healthy`.

Os últimos casos removem o pet e o tutor criados pela execução. Caso ela seja interrompida, confira se ficaram registros de teste pendentes.