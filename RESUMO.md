# Copa das Equipes – Great Slam 2026

Documentação técnica do aplicativo. Serve para retomar o desenvolvimento
em qualquer momento, mesmo em uma sessão nova sem histórico.

---

## ⚠️ Fonte de verdade dos dados

**O Firestore é a fonte de verdade dos resultados.**

O arquivo `backup/estado-AAAA-MM-DD.json` é apenas uma foto de um momento.
Ele fica desatualizado assim que um novo resultado é lançado pelo app.

Antes de alterar dados em qualquer sessão futura, confirme o estado atual
no Firestore ou no próprio app em modo admin. Já houve um caso de dado
antigo ser reintroduzido por confiar no JSON local.

---

## O que é o app

Aplicativo web de gestão da Copa das Equipes, competição por equipes entre
academias de tênis de Fortaleza, dentro do circuito GREAT SLAM.

- **Publicado em:** https://great-slam.github.io/Copa-das-Equipes/
- **Repositório:** great-slam/Copa-das-Equipes
- **Organizador:** Olavo Pimentel

Dois modos de uso:

| Modo | Quem usa | O que pode fazer |
|---|---|---|
| Público | Atletas e visitantes | Só leitura: ranking, jogos, grupos, calendário, regulamento, contatos |
| Admin | Organizador | Lança e edita resultados, gerencia equipes e grupos |

---

## Stack

- **React 18** via CDN (UMD) — sem npm, sem bundler
- **JSX compilado** com Babel CLI → JavaScript puro
- **CSS próprio** — design system escrito à mão, sem biblioteca de componentes
- **Firebase Firestore** (compat SDK v10.12.0) — sincronização em tempo real
- **GitHub Pages** — hospedagem gratuita
- Entrega final: **um único arquivo HTML** (~231 KB)

Não usa Vite, MUI, TypeScript, Webpack nem Next.js. A escolha é deliberada:
o app é pequeno e específico, e o arquivo único funciona em qualquer lugar
sem infraestrutura.

---

## Arquivos do projeto

### No repositório GitHub

```
index.html                      ← app publicado (renomeado de CopaGS2026_v3.html)
og-image.jpg                    ← preview social 1200×630, necessário para o link no WhatsApp
RESUMO.md                       ← este arquivo
fontes/copa_v4_compiled.js      ← JS compilado (fonte principal)
fontes/copa_css.css             ← design system
fontes/fb.js                    ← integração Firebase
fontes/admin.js                 ← modo admin + listener de sincronização
backup/estado-AAAA-MM-DD.json   ← foto do estado (NÃO é fonte de verdade)
```

### Para retomar em uma sessão nova

Envie o `copa-gs-fontes.zip` (ou os arquivos da pasta `fontes/`) junto com
este RESUMO.md. Sem eles, é necessário extrair tudo do HTML compilado, o que
funciona mas dá mais trabalho e aumenta o risco de perder ajustes.

---

## Estrutura do estado

Chave do localStorage: **`copa-gs-v9`**
(bumpar a versão sempre que a estrutura mudar, para invalidar o cache dos usuários)

```
state = { academias, atletas, partidas }
```

### academias
```
{ id, nome, capitao }
```

| id | nome | capitão |
|---|---|---|
| ac1 | Fast Tennis Guararapes | Rafael Medeiros |
| ac2 | ACT | David Coffee |
| ac3 | ACT Garden | Chicão |
| ac4 | Espaço Tennis | Samuel |

### atletas
```
{ id, nome, academiId, categoria, grupo, ordem, isBye, atStatus }
```
- `categoria`: Ômega, Alpha, Beta, Gamma, Delta, Iniciante, Feminino, Sub-12
- `grupo`: 'A' ou 'B'
- `atStatus`: 'ativo' | 'bye' | 'desistente'

Cada academia indica 16 atletas — 2 por categoria, um em cada grupo.
Vagas não preenchidas viram BYE.

### partidas
```
{ id, categoria, grupo, atleta1Id, atleta2Id, acad1Id, acad2Id,
  sets1, sets2, games1, games2, placarRaw, stbInfo, superTb,
  status, vencedor, porWO, woMotivo, rodadaIdx, jogoIdx, historico }
```
- `status`: 'pendente' | 'bye' | 'wo' | 'realizado'
- `placarRaw`: `{ s1a, s1b, s2a, s2b, stbA, stbB }`
- `rodadaIdx`: 0, 1 ou 2 (1ª, 2ª e 3ª rodada)

Total: **96 partidas** (8 categorias × 2 grupos × 6 jogos)

---

## Regras de pontuação

Implementadas em `calcClass()`:

- Cada vitória vale **1 ponto** para a academia do atleta
- **WO com vencedor** conta como J, V e D — mas **não** computa saldo de sets nem de games
- **Duplo WO** não conta como jogo para ninguém (nem J, nem V, nem D),
  para que V + D sempre feche com J
- Desempate: pontos → saldo de sets → saldo de games → sorteio
- Super Tie-Break conta como 1 game no saldo

Formato das partidas: melhor de 3 sets, sets de 6 games, tie-break de 7 em 6×6,
No-Ad, e Super Tie-Break de 10 pontos no lugar do terceiro set.

---

## Firebase

- **Projeto:** copa-das-equipes---great-slam
- **Coleção:** `copa` → **Documento:** `estado` → **Campo:** `state`

Todo o estado vive em um único documento. As regras do Firestore estão
abertas (`allow read, write: if true`) — adequado para um app de torneio,
mas não para dados sensíveis.

### Como a sincronização funciona

**Escrita** — `saveState()` grava no localStorage e, quando
`window._fbAdminMode` é verdadeiro, também no Firestore. Toda alteração feita
em modo admin sincroniza automaticamente. Não é preciso apertar Sincronizar
a cada resultado.

**Leitura** — `admin.js` mantém um `onSnapshot` no documento. Quando o
Firestore muda, compara a **assinatura** dos dados (IDs + status + vencedor +
sets das partidas concluídas, e IDs + nomes dos atletas) com a assinatura
local. Só recarrega a página se forem diferentes.

**Por que assinatura e não JSON completo:** comparar o JSON inteiro causava
loop infinito de refresh, porque a ordem das chaves variava entre
carregamentos e o app achava que os dados tinham mudado quando não tinham.

**Botão Sincronizar** (Dashboard admin) — envia o estado local inteiro para o
Firestore. Use depois de subir um `index.html` novo com dados atualizados,
para que o Firestore receba essas mudanças.

---

## Modo admin

O acesso é por senha, guardada em `sessionStorage` na chave `gs_admin`.

- Persiste ao recarregar a página
- Sai ao fechar a aba ou clicar em "Sair do admin"
- `isAdmin` do React é inicializado **lendo o sessionStorage**, não como
  `false` — foi o que resolveu o problema de perder o admin a cada refresh
- `window._fbAdminMode` é sincronizado junto, pois é ele que autoriza a
  escrita no Firestore

A senha está no `index.html` publicado, o que significa que é uma barreira
de conveniência, não de segurança. Adequado ao contexto, mas bom ter clareza
sobre isso.

---

## Como gerar um novo build

```python
import json, datetime

css     = open('/tmp/copa_css.css', encoding='utf-8').read()
main_js = open('/tmp/copa_v4_compiled.js', encoding='utf-8').read()
fb_js   = open('/tmp/fb.js', encoding='utf-8').read()
adm_js  = open('/tmp/admin.js', encoding='utf-8').read()
state   = json.dumps(json.load(open('/tmp/state_inline.json', encoding='utf-8')),
                     ensure_ascii=False)

# PATCH OBRIGATÓRIO — sem isso o Firebase não recebe as alterações
old = "const saveState = s => localStorage.setItem(KEY, JSON.stringify(s));"
new = """const saveState = s => {
  localStorage.setItem(KEY, JSON.stringify(s));
  if (window._fbAdminMode && window._fbDb) {
    window._fbDb.collection('copa').doc('estado').set({ state: s })
      .catch(function(e){ console.warn('[FB] Erro:', e); });
  }
};"""
patched = main_js.replace(old, new)
assert "collection('copa')" in patched, "PATCH FALHOU"

# Ordem dos scripts no HTML (não alterar):
#   1. window.INITIAL_STATE
#   2. fb.js
#   3. admin.js
#   4. JS principal patchado
#   5. ReactDOM.createRoot(...).render(CopaProvider > App)
```

Sempre validar antes de publicar:
```bash
node --check /tmp/copa_v4_compiled.js
```

Editar JSX compilado à mão quebra parênteses com facilidade. Essa checagem
já evitou duas telas em branco.

---

## Menus

**Público** — Ranking · Jogos · Grupos · Calendário · Regulamento · Contatos · Stats
**Admin** — Dashboard · Jogos · Ranking · Equipes · Grupos · Calendário · Regulamento · Stats

Contatos aparece nos dois menus do atleta, mas só no menu lateral do admin,
para não sobrecarregar a barra inferior.

---

## Identidade visual

Paleta e tipografia alinhadas ao GREAT SLAM HUB.

```
Dourado    #d4af37   Dourado claro  #f0d67a   Dourado médio  #c9a94e
Base       #0e0e0f   Cards          #1a1a1d   Recuo          #141416
Bordas     #2a2a2e                  #3a3a40
Texto      #f5f3ee → #a8a6a0 → #8f8e88   (três níveis, sem pesos competindo)
Apoio      verde #6fbf7f   vermelho #c9705f   WO #c9955f
```

- **Inter** no corpo (15px, letter-spacing −0.006em, números tabulares)
- **Cormorant Garamond** nos títulos de seção
- Cards com raio 18px e padding 20px
- Hover eleva 2px com borda dourada

Os ícones do menu são SVG de cor única — decisão consciente de não usar
emojis ali, para manter o menu uniforme e profissional. Emojis aparecem como
conteúdo (calendário, regulamento, contatos), não como navegação.

---

## Histórico de problemas resolvidos

Registrado para não repetir o diagnóstico:

| Problema | Causa | Solução |
|---|---|---|
| Loop de refresh | Comparação por JSON completo | Comparação por assinatura |
| Admin caía ao recarregar | `isAdmin` iniciava como `false` | Inicializa lendo sessionStorage |
| Visitante não via atualização | React renderizava antes do Firebase | Listener atualiza e recarrega |
| Firestore vazio | `saveState` só gravava no localStorage | Patch do `saveState` no build |
| Dados antigos reintroduzidos | JSON local tratado como fonte de verdade | Firestore é a fonte de verdade |
| Tela em branco | Parêntese faltando no JSX compilado | `node --check` antes do build |
| Firebase falha em `file://` | CDN bloqueado em arquivo local | Normal — só funciona pelo GitHub Pages |

---

## Calendário 2026

| Etapa | Data |
|---|---|
| Habilitação das academias + indicação do capitão | 04 a 11/08 |
| Indicação e inscrição dos 16 atletas | 12 a 30/08 |
| Divulgação das equipes e grupos | 31/08 |
| 1ª Rodada | 01 a 08/09 |
| 2ª Rodada | 09 a 16/09 |
| 3ª Rodada | 17 a 24/09 |
| Premiação na academia campeã | 25/09 |

---

## Contatos da organização

- Site: www.greatslam.com.br
- WhatsApp: (85) 98118-2225 — Olavo Pimentel
- Instagram: @great.slam
- E-mail: greatslam.tenis@gmail.com
