# Cronograma de Correções - MAP-OS App Flutter

> Objetivo: Corrigir todos os problemas que impedem o app de funcionar corretamente via HTTP e HTTPS, no navegador (Chrome/web) e no celular (Android/iOS).

---

## Fase 1 - Correções Críticas (Impedem funcionamento)

### 1.1 ✅ URL base duplicando `/api/v1`
- **Arquivo:** `lib/api/apiConfig.dart`
- **Correção:** `_normalizeURL()` remove `/api/v1` do final e adiciona `https://` automaticamente

### 1.2 ✅ `indexEndpoint` vazio retornava HTML
- **Arquivo:** `lib/api/apiConfig.dart`
- **Correção:** Mudado de `''` para `/api/v1/`

### 1.3 ✅ Verificação de internet falhava no navegador
- **Arquivos:** Todos os 8 controllers + dashboard_page
- **Correção:** `if (kIsWeb) return true;` + trocado URL para HTTPS

### 1.4 ✅ `android:usesCleartextTraffic` faltando
- **Arquivo:** `android/app/src/main/AndroidManifest.xml`
- **Correção:** Adicionado `android:usesCleartextTraffic="true"`

### 1.5 ✅ iOS ATS sem exceção para HTTP
- **Arquivo:** `ios/Runner/Info.plist`
- **Correção:** Adicionado `NSAllowsArbitraryLoads`

### 1.6 ✅ Tutorial com URL errada
- **Arquivo:** `lib/widgets/TutorialWidget.dart`
- **Correção:** Exemplo atualizado para URL sem `/api/v1`

### 1.7 ✅ Validação de URL no login
- **Arquivo:** `lib/pages/login/login_page.dart`
- **Correção:** Validação de URL vazia, sem protocolo, e HTTPS obrigatório no web

### 1.8 ✅ Normalização de URL sempre executa
- **Arquivo:** `lib/api/apiConfig.dart`
- **Correção:** `ensureBaseURLInitialized()` sempre chama `initBaseURL()` para normalizar URLs antigas

---

## Fase 2 - Correções Necessárias (Funcionamento parcial/instável)

### 2.1 ✅ Header `App-Version` ausente nos controllers
- **Arquivos:** `clientsController`, `osController`, `servicesController`, `productsController`, `chamadosController`
- **Correção:** Adicionado `'App-Version': APIConfig.appVersion` em todos os headers de request

### 2.2 ✅ Chamados usa endpoint de OS
- **Conclusão:** O Mapos **não possui** endpoint `/api/v1/chamados`. OS e Chamados são a mesma entidade (`/api/v1/os`). O controller já está correto.

### 2.3 ✅ Abas de OS fazem HTTP direto sem retry de token (403)
- **Arquivos:** `detalhes_tab.dart`, `descontos_tab.dart`, `produtos_tab.dart`, `servicos_tab.dart`
- **Correção:** Adicionado verificação de status 403 com `TokenController().regenerateToken()` e retry em todas as chamadas HTTP diretas

### 2.4 ✅ `anexos_tab.dart` usava campo `id` em vez de `idOs`
- **Arquivo:** `lib/pages/os/tabs/anexos_tab.dart` (linha 84)
- **Correção:** Trocado `widget.ordemServico!['id']` por `widget.ordemServico!['idOs']`

### 2.5 ✅ Imports nativos desnecessários em `anexos_tab.dart`
- **Arquivo:** `lib/pages/os/tabs/anexos_tab.dart`
- **Correção:** Removidos `device_info_plus` e `path_provider` (problemáticos no web e não usados)

### 2.6 ✅ `flutter_downloader` import podia crashar web
- **Arquivo:** `lib/main.dart`
- **Correção:** Removido import e inicialização do `flutter_downloader` (não usado no app)

### 2.7 ✅ HTTPS obrigatório no web (Mixed Content)
- **Arquivos:** `lib/api/apiConfig.dart`, `lib/pages/login/login_page.dart`
- **Correção:** Quando `kIsWeb` é true, URLs `http://` são automaticamente convertidas para `https://`

### 2.8 ✅ Tratamento offline no web - fallback para cache
- **Arquivos:** Todos os 6 controllers + dashboard_controller
- **Correção:** Adicionado `try/catch` com fallback para cache local quando `kIsWeb` é true e a API falha

---

## Fase 3 - Melhorias de Robustez

### 3.1 ✅ Erro do calendário não tratado graciosamente
- **Arquivo:** `lib/controllers/calendarController.dart`
- **Correção:** `_loadEventsFromLocal` retorna lista vazia em vez de lançar exceção. `fetchCalendarData` não imprime erro no catch.

### 3.2 ✅ Typo `prodtuostesEndpoint`
- **Arquivo:** `lib/api/apiConfig.dart` + 3 arquivos referenciando
- **Correção:** Renomeado para `produtosEndpoint`

### 3.3 ✅ Typo "Errro na conexão"
- **Arquivos:** `clientsController`, `servicesController`, `productsController`
- **Correção:** Corrigido para "Erro na conexão"

---

## Fase 4 - Pendências Futuras (não bloqueantes)

### 4.1 ❌ Navegação com `Navigator.push` acumula pilha
- **Arquivo:** `lib/widgets/bottom_nav_menu.dart`
- **Problema:** Cada troca de tab faz push na pilha de navegação
- **Sugestão:** Usar `IndexedStack` ou `pushReplacement`

### 4.2 ❌ SharedPreferences no web tem limite de ~5MB
- **Problema:** Cache grande pode exceder limite do `localStorage`
- **Sugestão:** Implementar limpeza de cache antigo quando `QuotaExceededError`

### 4.3 ❌ Botão "Adicionar OS" vai para "Em Desenvolvimento"
- **Arquivo:** `lib/pages/os/os_page.dart`
- **Sugestão:** Implementar tela de criação de OS ou manter como feature futura

### 4.4 ❌ CORS no servidor Mapos para deploy web em produção
- **Problema:** Sem headers CORS no servidor, app web não funciona em produção (sem `--disable-web-security`)
- **Sugestão:** Adicionar headers CORS no `.htaccess` do Mapos

### 4.5 ❌ Formato de data inconsistente em `detalhes_tab.dart`
- **Arquivo:** `lib/pages/os/tabs/detalhes_tab.dart`
- **Problema:** `dataInicial` usa formato original, `dataFinal` usa ISO8601
- **Sugestão:** Normalizar ambos os formatos

### 4.6 ❌ `print()` em produção
- **Arquivos:** Múltiplos controllers
- **Sugestão:** Substituir por `debugPrint()` ou remover

---

## Como Testar

1. **No Chrome (web):** Rode com `flutter run -d chrome --web-browser-flag=--disable-web-security`
2. **Na tela de login**, clique no ícone de engrenagem (⚙️)
3. **Configure a URL:** `https://jj-ferreiras.com.br/mapos/index.php`
4. **Faça login** com suas credenciais do Mapos
5. **Teste:** Dashboard, Clientes, OS/Chamados, Produtos, Serviços
6. **Para deploy web em produção**, é necessário configurar CORS no servidor Mapos (item 4.4)

---

## Status Final

| Fase | Total | Concluído | Pendente |
|------|-------|-----------|----------|
| Fase 1 (Críticas) | 8 | 8 ✅ | 0 |
| Fase 2 (Necessárias) | 8 | 8 ✅ | 0 |
| Fase 3 (Robustez) | 3 | 3 ✅ | 0 |
| Fase 4 (Futuras) | 6 | 0 | 6 ❌ |
| **Total** | **25** | **19 ✅** | **6 ❌** |