# Painel da Oficina

Painel para lançar vendas, despesas e dízimo da oficina, com histórico e gráficos.
Os dados ficam no arquivo `dados.json` (Google Drive › Mecanica › Painel), que **não** vai para o GitHub.

## Como abrir

- **Computador (Chrome/Edge):** abrir `painel.html` direto da pasta do Drive e escolher o `dados.json` na primeira vez.
- **Tablet/celular:** abrir o endereço do GitHub Pages, tocar em "Abrir pelo Google Drive", entrar com a conta Google
  e escolher o `dados.json` na primeira vez. Depois, no menu do Chrome, "Adicionar à tela inicial".

## Configuração do Google (feita uma vez)

O modo Google Drive precisa de três valores em `painel.html` (constante `GOOGLE`):

| Campo      | Onde pegar no Google Cloud Console                                   |
|------------|----------------------------------------------------------------------|
| `clientId` | APIs e serviços › Credenciais › ID do cliente OAuth (Aplicativo da Web) |
| `apiKey`   | APIs e serviços › Credenciais › Chave de API                          |
| `appId`    | Página inicial do projeto › "Número do projeto"                        |

No projeto: ativar **Google Drive API** e **Google Picker API**; tela de consentimento OAuth "Externo",
com o e-mail da conta do Drive como usuário de teste; no ID do cliente, em "Origens JavaScript autorizadas",
colocar o endereço do GitHub Pages (ex.: `https://usuario.github.io`).
Esses valores não são segredos: o acesso aos dados só acontece depois do login na conta Google,
e o painel só enxerga o arquivo escolhido no seletor (permissão `drive.file`).
