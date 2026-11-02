# MEMÓRIA DO PROJETO — Painel da Oficina

> Arquivo para orientar qualquer IA ou pessoa que for mexer neste projeto. Leia inteiro antes de alterar algo.
> Este repositório é **público**: não coloque aqui dados de clientes, chaves, senhas nem e-mails.
> Informações privadas (contas, IDs de projeto) ficam em `../MEMORIA-PRIVADA.md`, fora do repositório.

## Regras obrigatórias

1. **Toda alteração aumenta a versão** — constante `VERSAO` no topo do `<script>` de `painel.html`
   (aparece no rodapé "Criado por Guilherme Oliveira | Versão x.x.x") — e ganha uma linha em **Versões**, no fim deste arquivo.
   - correção pequena: `x.x.+1` (1.4.0 → 1.4.1)
   - recurso novo ou mudança visível: `x.+1.0` (1.4.1 → 1.5.0)
   - mudança que quebra o formato do `dados.json`: `+1.0.0`
2. **Nunca enviar dados para o GitHub**: `dados.json`, `.xlsx` e `.csv` estão no `.gitignore`. Conferir `git status` antes de cada commit.
3. **Nunca gravar chaves no código.** A chave da Gemini fica dentro do `dados.json` (`gemini.chave`).
   As únicas credenciais no código são as públicas do Google (ID do cliente OAuth, chave do Picker e número do projeto), que não dão acesso a dados.
4. Textos da interface em **português do Brasil**. Valores em R$ com `brl()`.
5. A venda **nunca é salva sozinha**: voz/Gemini só preenchem o formulário; quem salva é o usuário.
6. Antes de mexer no `dados.json` por fora do painel, fazer cópia de segurança. O painel aberto em outro aparelho recarrega sozinho quando o arquivo muda.

## O que é

Painel em **um único arquivo HTML** (`painel.html`, sem build, sem framework) para o mecânico lançar vendas, despesas e dízimo
e ver histórico e evolução desde nov/2021. Substituiu as planilhas mensais `Mecanica/<ano>/<mês> - Mecanica Vx - <Mês>.xlsx`
(o histórico foi importado delas; as planilhas ficam como arquivo morto).

- Publicado no **GitHub Pages**: `https://guilhermeoliveirati.github.io/painel-oficina/painel.html` (repositório `guilhermeoliveirati/painel-oficina`, branch `main`, pasta raiz). `git push` publica sozinho em ~1 minuto.
- Bibliotecas por CDN: Chart.js 4.4.1 (gráficos) e SheetJS 0.18.5 (gerar Excel). Sem internet, gráficos e Excel não funcionam; o resto sim.

## Arquivos (pasta `Meu Drive/Mecanica/Painel`, que é o repositório git)

| Arquivo | Para quê |
|---|---|
| `painel.html` | todo o painel (HTML + CSS + JS) |
| `dados.json` | **os dados** (fora do git, sincronizado pelo Google Drive) |
| `index.html` | redireciona para `painel.html` |
| `manifest.webmanifest`, `icone*.png/svg` | app instalável ("Adicionar à tela inicial") |
| `LEIA-ME.md` | instruções curtas de uso e configuração do Google |
| `MEMORIA.md` | este arquivo |

## Onde os dados ficam e como são abertos

O painel tem duas "fontes" com a mesma interface (`ler()`, `versao()`, `gravar(json)`):

- **local** — Chrome/Edge no computador, File System Access API sobre o `dados.json` da pasta do Google Drive.
  O identificador do arquivo fica no IndexedDB (`painel-oficina` › `kv` › `handle`). Confere mudanças a cada 5 s.
- **drive** — qualquer navegador (tablet). Login Google Identity Services (token, escopo `drive.file`) + Google Picker para escolher o arquivo uma vez;
  lê/grava pela Drive API v3 (`files/{id}?alt=media` e `upload ... uploadType=media`). Token dura ~1 h (botão "Entrar no Google de novo"). Confere a cada 15 s.
  Guardado no `localStorage`: `modo`, `driveArquivo`, `gToken`, `gJaAutorizou`, `gEmail` (conta usada, vira `hint` no login para não pedir a conta de novo), `jaAbriu`, `zoom`.

**Proteção contra sobrescrita:** antes de gravar, compara a versão atual do arquivo (`lastModified` ou `version` do Drive) com a última conhecida;
se mudou fora do painel, recarrega e avisa em vez de gravar por cima. Gravações vão numa fila (`salvar()`), uma por vez.

## Formato do `dados.json`

```jsonc
{
  "versao": 1,                       // formato do arquivo (não é a versão do painel)
  "atualizado": "2026-11-02T10:00:00",
  "proximoId": 1352,                 // ids: "v123" vendas, "d123" despesas, "p123" pendências
  "taxas": { "Dinheiro": 0, "Pix": 0, "Débito": 0.0137, "Crédito 1x": 0.0315, "...": 0, "Crédito 10x": 0.1106 },
  "margemPadrao": 10,                // opcional; % de margem em peças sugerida (padrão 10)
  "vendas": [ { "id", "data": "AAAA-MM-DD", "cliente", "pecas", "descPecas", "servico", "cobrado", "metodo", "taxa", "lucro", "obs" } ],
  "despesas": [ { "id", "data", "categoria", "descricao", "valor" } ],
  "pendencias": [ ... ],            // legado: a aba Pendências foi removida na 1.6.0; o painel não lê nem grava mais
  "dizimos": { "AAAA-MM": { "itens": [ { "nome", "valor" } | { "nome", "auto": true } ] } },
  "gemini": { "chave": "…", "modelo": "gemini-x.y-flash" }   // opcional
}
```

## Regras de negócio (combinadas com o dono)

- **Venda de peças** = preço original de loja que o cliente paga. **Desconto/margem em peças** (`descPecas`) = o que o mecânico ganhou comprando mais barato; padrão 10% das peças, editável. É **lucro**, não desconto ao cliente.
- **Valor cobrado** = peças + serviço + um valor a mais que o mecânico decide na hora (pode ser menor, se der desconto).
- **Taxa** = `cobrado × taxa do método` (tabela em Config).
- **Lucro da venda = cobrado + descPecas − pecas − taxa** (= serviço + margem + cobrado a mais − taxa). Explicado no botão "i" da tela de venda.
  Planilhas de mar–set/2026 usavam fórmula errada (ignoravam o cobrado a mais); foram corrigidas. Lucros de fev e abr/2022 foram digitados à mão e **não** devem ser recalculados.
- **Lucro final do mês** = soma dos lucros − despesas. Despesas por mês só existem de mar/2026 em diante.
- **Dízimo**: até 4 valores por mês, dízimo = 10% da soma. Mês sem registro = automático: 1º valor "Oficina" = lucro − despesas do mês (`auto: true` acompanha mudanças). Excluir grava `itens: []`.
- **Resumo**: lucro oficina − despesas − dízimo = lucro líquido.
- Nomes de clientes: primeira letra de cada palavra maiúscula (`nomeProprio`), exceto "da, de, do, das, dos, e".

## Abas

Visão geral (KPIs, gráficos mês/ano, mapa mês × ano, fechamento mensal) · Vendas (formulário, voz/Gemini, lista do mês, gerar Excel do mês) ·
Despesas · Clientes (ranking, sumidos, histórico) · Dízimo · Resumo · Config (tamanho da letra, Gemini, margem padrão, taxas, exportar).

## Venda por voz (Gemini)

- Fala → texto: Web Speech API do navegador (`pt-BR`). O Chrome encerra o reconhecimento nas pausas; o painel religa até o usuário tocar em "Parar" (máx. 2 min).
- Texto → campos: Gemini API (`generativelanguage.googleapis.com/v1beta`, cabeçalho `x-goog-api-key`), saída JSON com `responseSchema`,
  `temperature: 0` e `thinkingConfig.thinkingBudget: 0` (muito mais rápido). O prompt leva a data de hoje, os métodos de pagamento e os nomes de clientes já cadastrados.
- Modelos: lista os "Flash" da chave (último que funcionou, depois numerados do mais novo, depois preview, depois lite), tenta até 6 com limite de 10 s cada e passa ao próximo se der 503/429/demora; se todos falharem, espera (usa o `retryDelay` do erro) e faz mais uma rodada. Guarda o que funcionou em `gemini.modelo`.
- Plano gratuito tem **limite por minuto**: não disparar vários testes em paralelo com a chave real (foi o que causou o 429 em 2026-11-02).

## Como testar sem mexer nos dados reais

Copiar `painel.html` + `dados.json` para uma pasta temporária, servir com `python -m http.server 8765` e, no console da página:
`D = await fetch('dados.json').then(r=>r.json()); iniciar();` — sem "fonte", nada é gravado.
`http://localhost:8765` está autorizado no cliente OAuth para testes. Conferir sintaxe extraindo o último `<script>` e rodando `node --check`.

## Versões

| Versão | Data | Mudança |
|---|---|---|
| 1.0.0 | 2026-10-29 | Painel inicial: vendas, despesas, clientes, pendências, gráficos; histórico importado das planilhas 2021–2026 |
| 1.1.0 | 2026-10-30 | Dízimo, Resumo, Excel do mês, margem editável, "cobrado a mais", nova fórmula de lucro, botão "i" |
| 1.2.0 | 2026-10-31 | Web app: abre o dados.json pelo Google Drive (tablet), GitHub Pages, ícone e manifest |
| 1.3.0 | 2026-10-31 | Tamanho da letra por aparelho; orientação de escolha do arquivo só na primeira vez |
| 1.4.0 | 2026-11-02 | Venda por voz com Gemini (troca de modelo, nomes compostos, microfone contínuo); rodapé com autor e versão |
| 1.5.0 | 2026-11-02 | Vendas: campo Obs removido do formulário e da lista (obs antigas continuam no arquivo, no histórico do cliente e no Excel); Gemini não preenche mais obs |
| 1.5.1 | 2026-11-02 | Gemini: diferencia limite do plano gratuito (429) de sobrecarga (503), usa também modelos preview/lite, espera e tenta de novo antes de desistir |
| 1.5.2 | 2026-11-02 | Gemini: modelo desativado pelo Google (404) sai da lista na hora |
| 1.5.3 | 2026-11-02 | Campos numéricos: o 0 some ao tocar no campo e volta se sair sem digitar |
| 1.6.0 | 2026-11-02 | Login Google lembra a conta usada (não pede para escolher de novo); aba Pendências removida |
| 1.7.0 | 2026-11-02 | Status discreto no topo ("☁ Drive"); botão "Exportar" à direita na linha do Mês/Buscar; texto de preenchimento automático movido para o "i" |
