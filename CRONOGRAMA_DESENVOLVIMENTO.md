# Cronograma de Desenvolvimento - MAP-OS App Flutter

> Itens pendentes da Fase 4 do cronograma de correções, expandidos com planos de implementação detalhados.

---

## Item 4.1 — Navegação com `Navigator.push` acumula pilha

**Arquivo:** `lib/widgets/bottom_nav_menu.dart`

**Problema:** Cada toque na bottom nav faz `Navigator.push`, criando pilha infinita. Botão "voltar" navega por todas as telas anteriores.

**Plano de implementação:**
1. Trocar `Navigator.push` por `Navigator.pushReplacement` em `_handleTap()`
2. Isso substitui a tela atual pela nova, sem acumular na pilha
3. Manter o `PageTransitionType` para animação suave

**Arquivos a alterar:** `bottom_nav_menu.dart`

---

## Item 4.2 — SharedPreferences no web tem limite de ~5MB

**Arquivos:** Todos os controllers que salvam cache

**Problema:** No navegador, `SharedPreferences` usa `localStorage` com limite de ~5MB. Com 761 OSs, 90 clientes, etc., o cache pode exceder esse limite.

**Plano de implementação:**
1. Criar um método utilitário `_safeSavePrefs()` que envolve `prefs.setString()` em try/catch
2. Se ocorrer `QuotaExceededError`, limpar caches antigos (produtos, serviços, etc.) e tentar novamente
3. Aplicar em todos os `_saveXFromLocal()` dos controllers

**Arquivos a alterar:** Todos os controllers + novo utilitário

---

## Item 4.3 — Botão "Adicionar OS" vai para "Em Desenvolvimento"

**Arquivo:** `lib/pages/os/os_page.dart` (linha 148)
**Arquivo stub:** `lib/pages/os/os_add_page.dart`

**Problema:** Não existe tela de criação de OS. O botão "Adicionar" navega para uma página placeholder.

**Plano de implementação:**
1. Reescrever `os_add_page.dart` com formulário completo:
   - Campo: Cliente (autocomplete com busca na API)
   - Campo: Técnico responsável (dropdown com usuários)
   - Campo: Data inicial (date picker)
   - Campo: Data final (date picker)
   - Campo: Status (dropdown)
   - Campo: Descrição do produto (textarea)
   - Campo: Defeito (textarea)
   - Campo: Observações (textarea)
   - Campo: Valor total (numérico)
2. Criar método `addOs()` no `osController.dart`
3. Usar o padrão já existente do `chamados_add_page.dart` como referência

**Arquivos a criar/alterar:** `os_add_page.dart`, `osController.dart`

---

## Item 4.4 — CORS no servidor Mapos (Deploy web em produção)

**Arquivo:** `.htaccess` no servidor Mapos (não no app Flutter)

**Problema:** Sem headers CORS, o app web não funciona em produção (sem `--disable-web-security`).

**Plano de implementação:**
1. Adicionar ao `.htaccess` do Mapos no servidor:
```apache
<IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
    Header set Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization, App-Version"
    Header set Access-Control-Max-Age "86400"
</IfModule>

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_METHOD} OPTIONS
    RewriteRule ^(.*)$ $1 [R=204,L]
</IfModule>
```
2. Isso precisa ser feito **no servidor**, não no app

**Ação:** Fornecer o arquivo `.htaccess` pronto para deploy

---

## Item 4.5 — Formato de data inconsistente em `detalhes_tab.dart`

**Arquivo:** `lib/pages/os/tabs/detalhes_tab.dart`

**Problema:** `dataInicial` é enviado no formato original da API, mas `dataFinal` é convertido para ISO8601. Os dois campos chegam ao servidor em formatos diferentes.

**Plano de implementação:**
1. Normalizar ambos os campos para usar o mesmo formato no envio
2. Converter `dataInicial` para ISO8601 da mesma forma que `dataFinal`
3. Garantir que o parse de `dd/MM/yyyy` funcione corretamente

**Arquivos a alterar:** `detalhes_tab.dart`

---

## Item 4.6 — `print()` em produção

**Arquivos:** 55 instâncias em 13 arquivos

**Problema:** `print()` gera spam no console do navegador e não é desabilitado em release.

**Plano de implementação:**
1. Substituir todos os `print()` por `debugPrint()` que é automaticamente desabilitado em modo release
2. Remover prints de debug que não são úteis (ex: "produtos salvos para uso Offline")
3. Manter apenas prints de erro crítico

**Arquivos a alterar:** Todos os controllers e páginas com `print()`

---

## Ordem de Execução

| Ordem | Item | Dificuldade | Tempo Estimado |
|-------|------|-------------|-----------------|
| 1 | 4.1 — Navegação pushReplacement | Fácil | 5 min |
| 2 | 4.5 — Formato de data | Fácil | 10 min |
| 3 | 4.6 — Substituir print() por debugPrint() | Fácil | 10 min |
| 4 | 4.2 — Cache localStorage limite | Média | 15 min |
| 5 | 4.3 — Tela de adicionar OS | Média | 30 min |
| 6 | 4.4 — CORS no .htaccess | Fácil (arquivo pronto) | 2 min |