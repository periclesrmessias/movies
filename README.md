# Nosso Cinema

Dashboard de filmes assistidos e pendentes, com dados sincronizados direto do Google Sheets.

## Como funciona

O arquivo `index.html` busca os dados da planilha publicada em CSV toda vez que a página é aberta (`fetch` no carregamento). Não existe build, backend ou GitHub Actions: é um único arquivo estático que lê a planilha em tempo real no navegador de quem acessa.

Fluxo: Google Sheets (aba "Filmes") → Publicar na web (CSV) → `index.html` busca e renderiza.

## Publicar no GitHub Pages

1. Copie o arquivo `index.html` para a raiz do repositório `movies` (substituindo o que já existe, se houver).
2. Faça commit e push para a branch `main`.
3. No GitHub: Settings → Pages → em "Build and deployment", Source = "Deploy from a branch", Branch = `main` / `/(root)`. Salve.
4. Em alguns minutos o dashboard estará em `https://SEU_USUARIO.github.io/movies/`.

Comandos via linha de comando, se preferir:

```bash
git clone https://github.com/SEU_USUARIO/movies.git
cp index.html movies/index.html
cd movies
git add index.html
git commit -m "Dashboard com dados dinâmicos do Google Sheets"
git push
```

## Manter a planilha publicada corretamente

O link CSV usado no dashboard depende da planilha continuar publicada nesse formato específico:

1. No Google Sheets, na aba **Filmes**: Arquivo → Compartilhar → **Publicar na web**.
2. Em "Link", selecione a aba **Filmes** (não "Todo o documento") e o formato **CSV**.
3. Clique em Publicar.
4. Qualquer edição na planilha aparece automaticamente no CSV publicado em poucos minutos — não é necessário republicar depois de cada alteração, só na primeira configuração.

Se algum dia trocar de planilha ou de aba, é só gerar um novo link (mesmo processo) e atualizar a constante `SHEET_CSV_URL` no topo do `<script>` do `index.html`.

## Estrutura de colunas esperada na aba "Filmes"

| Coluna na planilha | Uso no dashboard |
|---|---|
| Filme | Nome do filme |
| Visto | `TRUE`/`FALSE` — se já foi assistido pelo casal |
| Visto Antes G. | `TRUE`/`FALSE` — se Gabrielle já tinha visto antes |
| Visto Antes P. | `TRUE`/`FALSE` — se Péricles já tinha visto antes |
| Data | Data no formato `DD/MM/AAAA` (pode ficar vazia) |
| Streaming | Onde está disponível |
| Nota Gabrielle | Nota de 0 a 5 (aceita vírgula ou ponto decimal) |
| Nota Pericles | Nota de 0 a 5 (aceita vírgula ou ponto decimal) |
| Gênero | Categoria do filme |

Não altere os nomes dessas colunas na planilha sem também atualizar o mapeamento em `csvRowsToMovies()` dentro do `index.html`.

## Comportamento em caso de falha

Se o dashboard não conseguir buscar a planilha (sem internet, planilha despublicada, etc.), ele exibe um aviso no topo e cai automaticamente para os dados salvos localmente no arquivo (a última versão conhecida), então o site nunca fica quebrado.

## Compartilhar com a Gabrielle

Basta enviar o link `https://SEU_USUARIO.github.io/movies/`. Não precisa de login nem permissão especial — a página é pública, e os dados são sempre os mais recentes da planilha no momento em que ela abrir o link.
