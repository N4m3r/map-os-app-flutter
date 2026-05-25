# Cronograma - Funcionalidade "Abrir Chamado" no App Flutter

## Visao Geral

Adicionar a funcionalidade de **abrir chamados** no app `map-os-app-flutter`, utilizando a API REST do Mapos (`api/v1/os`). No Mapos, chamados sao Ordens de Servico (OS) com status inicial "Aberto". O objetivo e permitir que o usuario crie, liste e visualize chamados diretamente pelo app, **sem alterar a estrutura atual** - apenas acrescentando novos arquivos e fazendo modificacoes minimas pontuais nos arquivos existentes.

### Decisao arquitetural

- Usar o endpoint **`api/v1/os`** (POST) da API admin, pois o app autentica como usuario admin/tecnico
- Campos obrigatorios para criar: `dataInicial`, `dataFinal`, `status`, `clientes_id`, `usuarios_id`
- Campos opcionais: `defeito`, `observacoes`, `descricaoProduto`
- Status inicial padrao: `"Aberto"`
- Seguir os padroes existentes do app: `StatefulWidget`, `setState`, controllers como classes simples, `Map<String, dynamic>`, `SharedPreferences` para cache, tratamento 403 com `TokenController`

---

## Estrutura de Arquivos a Criar

```
lib/
  controllers/
    chamados/
      chamadosController.dart        # Controller CRUD de chamados
  pages/
    chamados/
      chamados_page.dart             # Lista de chamados (paginada)
      chamados_add_page.dart         # Formulario para abrir chamado
      chamados_view_page.dart        # Visualizar detalhes do chamado
```

## Arquivos Existentes a Modificar (minimo)

| Arquivo | Modificacao |
|---|---|
| `lib/api/apiConfig.dart` | Adicionar constante `chamadosEndpoint` apontando para `/os` |
| `lib/widgets/bottom_nav_menu.dart` | Adicionar icone "Chamados" na barra de navegacao inferior (substituir ou adicionar tab) |

> **Alternativa sem modificar o bottom_nav**: Adicionar botao de acesso aos chamados dentro da `DashboardPage` (card de status ou botao flutuante), evitando qualquer mudanca na navegacao inferior.

---

## Fases do Cronograma

### Fase 1 - Configuracao da API e Controller (2h)

**1.1** Adicionar endpoint em `lib/api/apiConfig.dart`:
```dart
static const String chamadosEndpoint = '/os';
```

**1.2** Criar `lib/controllers/chamados/chamadosController.dart` com os metodos:
- `getAllChamados(int page)` - GET `/os` (paginado, igual ao padrao de `ControllerClients`)
- `getChamadoById(String id)` - GET `/os/{id}`
- `addChamado(Map<String, dynamic> data)` - POST `/os`
- `deleteChamado(String id)` - DELETE `/os/{id}`

Padroes a seguir (copiar de `clientsController.dart`):
- `APIConfig.ensureBaseURLInitialized()` antes de cada request
- Header `Authorization: Bearer <token>`
- Retry em caso de 403 via `TokenController().regenerateToken()`
- Cache offline via `SharedPreferences`
- Verificacao de conexao com `hasInternetConnection()`

---

### Fase 2 - Pagina de Listagem de Chamados (3h)

**2.1** Criar `lib/pages/chamados/chamados_page.dart`:
- StatefulWidget seguindo o padrao de `clients_page.dart`
- `ListView.builder` paginado com scroll infinito
- Shimmer loading durante carregamento
- Card para cada chamado exibindo: `idOs`, `status` (com cor), `dataInicial`, `clientes_id` (nome do cliente)
- Filtro por status (Dropdown: Todos, Aberto, Em Andamento, Finalizado, etc.)
- Pull-to-refresh
- FAB (FloatingActionButton) para "Abrir Chamado" -> navega para `ChamadosAddPage`
- Tap no card -> navega para `ChamadosViewPage`

---

### Fase 3 - Formulario "Abrir Chamado" (4h)

**3.1** Criar `lib/pages/chamados/chamados_add_page.dart`:
- Form com `GlobalKey<FormState>` (padrao existente)
- Campos do formulario:

| Campo | Tipo | Obrigatorio | Widget |
|---|---|---|---|
| Cliente | Selecao | Sim | `TypeAheadField` (autocomplete, busca via API `clientes?search=`) |
| Tecnico | Selecao | Sim | `DropdownButton` (busca via API `usuarios`) |
| Data Inicial | Data | Sim | `TextFormField` com `showDatePicker` |
| Data Final | Data | Sim | `TextFormField` com `showDatePicker` |
| Status | Selecao | Sim | `DropdownButton` (default "Aberto") |
| Descricao do Produto | Texto | Nao | `TextFormField` multiline |
| Defeito | Texto | Nao | `TextFormField` multiline |
| Observacoes | Texto | Nao | `TextFormField` multiline |

- Validacao com `validator` em cada campo obrigatorio
- Botao "Abrir Chamado" que chama `addChamado()`
- Sucesso: toast/snackbar + `Navigator.pushReplacement` para `ChamadosViewPage`
- Erro: toast/snackbar com mensagem de erro

> **Nota**: O endpoint de usuarios (`/usuarios`) ja esta definido em `apiConfig.dart` como `usuarioEndpoint`. Sera necessario criar um metodo no controller para buscar tecnicos ativos.

---

### Fase 4 - Pagina de Visualizacao do Chamado (3h)

**4.1** Criar `lib/pages/chamados/chamados_view_page.dart`:
- Seguir padrao de `clients_view_page.dart`
- `FutureBuilder` para carregar dados do chamado
- Card de detalhes exibindo:
  - ID do Chamado
  - Status (com badge colorido)
  - Cliente
  - Tecnico responsavel
  - Data Inicial / Data Final
  - Descricao do Produto
  - Defeito
  - Observacoes
  - Laudo Tecnico (se houver)
  - Valor Total
- Botoes de acao:
  - "Editar" -> navega para pagina de edicao (fase futura, por enquanto exibe "Em Desenvolvimento")
  - "Excluir" -> dialog de confirmacao com captcha matematico (padrao existente)
- Shimmer loading

---

### Fase 5 - Integracao de Navegacao (2h)

**5.1 Opcao A - Modificar bottom_nav_menu.dart** (adicionar tab "Chamados"):
- Adicionar sexto item na `CircleNavBar`: icone `Boxicons.bx_message_square_add` + label "Chamados"
- Adicionar caso no `switch` de navegacao para `ChamadosPage`
- Ajustar indices dos outros tabs

**5.2 Opcao B - Acesso via Dashboard** (sem alterar bottom nav):
- Adicionar card "Chamados" na `DashboardPage` com icone e contador de chamados abertos
- Ao tocar, navega para `ChamadosPage`
- Modificacao apenas em `dashboard_page.dart`

> **Recomendacao**: Opcao B e menos invasiva e nao quebra a navegacao existente de 5 tabs.

---

### Fase 6 - Testes e Ajustes (2h)

**6.1** Testes manuais:
- Login no app
- Abrir chamado com todos os campos obrigatorios
- Abrir chamado com campos opcionais
- Validar campo a campo (cliente vazio, data invalida, etc.)
- Listar chamados com paginacao e filtro
- Visualizar detalhes do chamado
- Excluir chamado com confirmacao
- Testar offline (cache)
- Testar renovacao de token (403 -> retry)

**6.2** Ajustes de UX:
- Cores de status consistentes com o web (verde=Finalizado, laranja=Em Andamento, vermelho=Cancelado, azul=Aberto)
- Toasts de feedback em todas as acoes
- Responsividade para diferentes tamanhos de tela

---

## Resumo de Tempo

| Fase | Descricao | Tempo Estimado |
|---|---|---|
| 1 | API + Controller | 2h |
| 2 | Pagina de Listagem | 3h |
| 3 | Formulario Abrir Chamado | 4h |
| 4 | Pagina de Visualizacao | 3h |
| 5 | Integracao de Navegacao | 2h |
| 6 | Testes e Ajustes | 2h |
| **Total** | | **16h** |

---

## Ordem de Dependencia

```
Fase 1 (API + Controller)
  |
  +---> Fase 2 (Listagem) ---> Fase 5 (Navegacao)
  |
  +---> Fase 3 (Formulario) ---> Fase 5 (Navegacao)
  |
  +---> Fase 4 (Visualizacao) ---> Fase 5 (Navegacao)
  |
  +---> Fase 6 (Testes) [depende de todas as fases acima]
```

Fases 2, 3 e 4 podem ser feitas em paralelo apos a Fase 1.

---

## Endpoints da API Utilizados

| Acao | Metodo | Endpoint | Campos |
|---|---|---|---|
| Listar chamados | GET | `/os` | `page`, `per_page` (query params) |
| Buscar chamado | GET | `/os/{id}` | - |
| Criar chamado | POST | `/os` | `dataInicial`, `dataFinal`, `status`, `clientes_id`, `usuarios_id`, `defeito?`, `observacoes?`, `descricaoProduto?` |
| Excluir chamado | DELETE | `/os/{id}` | - |
| Buscar clientes | GET | `/clientes` | `search` (query param) |
| Buscar usuarios | GET | `/usuarios` | - |

---

## Status do Mapos e Cores Sugeridas

| Status | Cor no App |
|---|---|
| Aberto | `Colors.blue` |
| Orcamento | `Colors.purple` |
| Negociacao | `Colors.deepPurple` |
| Aprovado | `Colors.teal` |
| Aguardando Pecas | `Colors.amber` |
| Em Andamento | `Colors.orange` |
| Finalizado | `Colors.green` |
| Faturado | `Colors.indigo` |
| Cancelado | `Colors.red` |

---

## Observacoes

- Nao serao criadas telas de edicao de chamado nesta etapa (futura)
- A tab "Anotacoes" dentro de OS tambem sera tratada como funcionalidade futura
- O app usa autenticacao admin, portanto o endpoint `/os` (admin) e o correto, nao o `/client/os`
- Toda a UI sera em pt_BR, seguindo o padrao existente
- Os nomes de arquivos e classes seguirao o padrao existente (camelCase para classes, snake_case para arquivos)