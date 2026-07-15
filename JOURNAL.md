# Registro de Aprendizado
# Tratamento de "title" (ABORTADO)

**Contexto:** Decisão inicial de tratar a coluna `title`, extraindo o ano da string.
- Foi elaborada a expressão regular (Regex).
- Foi criada uma nova coluna do tipo inteiro contendo apenas o ano do filme.

## Tratamento de Dados Ausentes (Ano)
Identifiquei dois problemas em três instâncias:
1. **Caso de Série de TV**: Registro com duas datas (lançamento e fim).
2. **Ausência de Ano**: Duas instâncias não apresentavam o ano de lançamento no título.


### Caso 1: Série de TV — *Fawlty Towers (1975-1979)*
A decisão inicial foi remover esta instância do dataset. O motivo é que o registro se trata de uma série e não de um filme, configurando um ruído discrepante das características dos dados analisados.

### Caso 2: Ausência da data de lançamento
Por serem apenas duas instâncias, considerou-se a remoção direta. Um dos filmes é *"Li'l Quinquin"*, que também possui o gênero ausente, reforçando a intenção de exclusão. No entanto, antes de prosseguir, foi verificado o dataset `ratings.csv` para descobrir se esses títulos possuíam uma quantidade considerável de avaliações.

Após filtrar e contar as avaliações para cada filme, descobriu-se que ambos possuíam apenas 1 avaliação cada. A decisão final foi remover as duas instâncias.

## Mudança de Rota
A minha intenção inicial era remover manualmente essas 3 instâncias. Contudo, percebi que aplicar um filtro prévio baseado no volume de avaliações eliminaria esses mesmos casos de forma automática. 

Por isso, abortei este plano de tratamento manual. **Reiniciei** o fluxo voltando alguns passos para a EDA (Análise Exploratória de Dados), unindo os datasets para criar um filtro de qualidade geral (*threshold*). Isso poupa custo computacional e evita retrabalho.

---

# EDA (Análise Exploratória de Dados)

O foco desta etapa é analisar os dois datasets de forma conjunta para descobrir os principais atributos e aplicar o primeiro grande filtro de qualidade nos dados.

## Dataset: `movies.csv` (após tratamento de dados duplicados)
- **Dimensões:** 10.327 linhas e 3 colunas.
- **Estrutura:**
  - `movieId` (int)
  - `title` (str)
  - `genres` (str)
- **Dados faltantes:** Nenhum registro nulo.

### Observações:
* **Coluna `genres`:** O formato atual é uma string com múltiplos valores separados por barra vertical (ex: `Adventure|Animation|Children|Comedy|Fantasy`). Este formato pode atrapalhar o sistema de recomendação. Pensando em uma estrutura ideal, esses gêneros deveriam ser separados (relação de 1 para N) e convertidos em IDs categóricos, o que facilita o processamento por modelos de Machine Learning. Resta a dúvida sobre a necessidade de aplicar essa transformação neste momento.
* **Coluna `title`:** Exemplo `"Toy Story (1995)"`. Extrair o ano do título pode gerar correlações interessantes futuramente. Há casos complexos como `"Man Without a Face, The (1993)"`, que possuem artigos deslocados. Os gêneros continuam sendo o atributo de maior peso neste dataset.

## Dataset: `ratings.csv` (após tratamento de dados duplicados)
- **Dimensões:** 105.335 linhas e 4 colunas.
- **Estrutura:**
  - `userId` (int)
  - `movieId` (int)
  - `rating` (float)
  - `timestamp` (int)
- **Dados faltantes:** Nenhum registro nulo.

### Observações:
Vou desconsiderar a coluna `timestamp` inicialmente por ser complexa e de menor importância para a etapa atual. A variável `rating` é o alvo mais importante de todo o sistema de recomendação. As colunas `movieId` e `userId` possuem grande valor estratégico para a aplicação de filtragens de volume.

### Próximos passos para avaliar a qualidade dos registros:
1. Analisar o volume de avaliações por filme.
2. Analisar o volume de avaliações fornecidas por usuário.
3. Definir o *Threshold* (limiar de corte).

---

## Passo 1: Contagem de avaliações por filme
Existem 10.323 filmes avaliados; 4 filmes do catálogo não obtiveram nenhuma avaliação.

> **Dúvida:** Devo realizar o *merge* dos datasets agora para explorar os dados de forma unificada? Optei por prosseguir separadamente. Unir as bases neste momento consumiria mais recurso computacional de forma desnecessária. É mais eficiente filtrar quais avaliações e avaliadores serão considerados para, somente então, juntar as tabelas.

Ao executar o método `.describe()` sobre os dados de contagem de avaliações por filme (`rating_counts`), obteve-se a seguinte distribuição:
- **Média (mean):** 10.20
- **Desvio Padrão (std):** 22.83
- **Mínimo (min):** 1
- **25% (Q1):** 1
- **Mediana (50%):** 3
- **75% (Q3):** 8
- **Maximum (max):** 325

### Conclusão:
A amostra geral é pequena. O ideal seria aplicar um corte severo para prezar pela qualidade, mas o tamanho total dos dados restringe essa abordagem. Um filme com apenas 3 avaliações possui baixa relevância estatística, mas se o corte for muito alto, a amostra final será drasticamente reduzida. Embora 3 avaliações pareça um número baixo, este valor representa a mediana (50%) dos dados e será adotado como o *threshold* inicial.

---

## Passo 2: Contagem de avaliações por usuário
Existem 668 usuários únicos no dataset.

Ao executar o método `.describe()` sobre as avaliações por usuário (`ratings_per_user`), obteve-se a seguinte distribuição:
- **Média (mean):** 157.68
- **Desvio Padrão (std):** 319.66
- **Mínimo (min):** 20
- **25% (Q1):** 35
- **Mediana (50%):** 70.50
- **75% (Q3):** 153
- **Máximo (max):** 5677

### Conclusão:
Como o valor mínimo de avaliações por usuário é 20, este perfil já demonstra engajamento consistente e relevância para o modelo, eliminando o risco de usuários esporádicos. Portanto, todos os avaliadores serão mantidos na amostra.

> **To-Do (Futuro):** Avaliar a criação de um sistema de pesos onde usuários com maior volume de avaliações (ex: 5.677 registros) tenham impacto proporcionalmente diferente daqueles com volume mínimo (ex: 20 registros). Esse mesmo raciocínio de pesos pode ser aplicado aos filmes, mas deve ser feito com cautela em etapas avançadas para evitar a introdução de vieses no modelo.

> **Ideia (Futuro):** Incorporar contextos comerciais ao sistema de recomendação, como impulsionamento pago de títulos específicos ou ganchos de lançamentos multimídia (ex: livros do mesmo universo).

---

## Passo 3: Aplicação do filtro e Resultados

O critério estabelecido foi filtrar o dataset de filmes, mantendo apenas aqueles com no mínimo 3 avaliações (mediana). Todos os usuários foram mantidos.

> **Dúvida:** Por falta de experiência prática em sistemas de recomendação, surge o receio de excesso de zelo ao remover dados. No entanto, os registros estão sendo descartados apenas do conjunto de treinamento do modelo, e não deletados do banco original. Restringir o volume de filmes irrelevantes pode ser benéfico para o aprendizado do modelo, mitigando o risco de *overfitting* (sobreajuste).

### Execução:
1. Geração de dois histogramas para visualizar a distribuição das contagens de avaliações por filme.
2. Extração dos IDs filtrados.

Após a aplicação dos filtros, foram consolidadas as seguintes alterações nos dataframes:

* **`ratings`:** De 105.335 avaliações, foram removidas 6.546, resultando em uma amostra final de 98.789 avaliações (impacto de aproximadamente 6,2% nos dados originais).
* **`movies`:** De 10.327 filmes, foram removidos 5.096, resultando em uma amostra final de 5.231 filmes. O filtro reduziu o catálogo original quase pela metade (49,3%).

---

# Feature Engineering (ABORTADO)

A ideia inicial era avançar para a remoção da coluna `timestamp` do dataset de avaliações. No entanto, ao consultar a documentação do Pandas para aplicar o método `.drop()`, me deparei com o `.drop_duplicates()`. 

Isso me alertou para uma validação crucial que havia passado batida na primeira etapa de EDA: a verificação de registros duplicados nas duas bases.

Embora o dataset de avaliações (`df_ratings`) não apresentasse duplicatas em sua chave composta (`userId` + `movieId`) antes do tratamento, uma checagem no dataset de filmes (`df_movies`) revelou um problema de integridade: **2 títulos estavam duplicados**, operando sob IDs diferentes.

---

# Revisão da Análise Exploratória (EDA): Tratamento de Títulos Duplicados
Os casos de colisão encontrados no dataset de filmes foram:

1. **Men with Guns (1997)**
   * `movieId 1788`: Action|Drama
   * `movieId 26982`: Drama

2. **War of the Worlds (2005)**
   * `movieId 34048`: Action|Adventure|Sci-Fi|Thriller
   * `movieId 64997`: Action|Sci-Fi

### Impacto Identificado nos Ratings:
Antes de qualquer remoção, verifiquei o volume de interações associado a cada ID:
* `34048`: 33 avaliações | `64997`: 3 avaliações
* `1788`: 3 avaliações  | `26982`: 2 avaliações

### Abordagem de Resolução Aplicada:
Para não descartar o histórico de avaliações legítimas dos usuários e garantir a melhor qualidade de metadados, adotei a seguinte estratégia de consolidação:
1. **Seleção de Identificadores:** Optei por preservar os IDs `1788` e `34048` por conterem os dados de gênero (`genres`) mais completos no mapeamento original.
2. **Correção:** No dataset de avaliações, remapeei e substituí os IDs antigos (`26982` -> `1788` e `64997` -> `34048`).
3. **Tratamento de Colisão Induzida:** A unificação dos IDs poderia fazer com que um mesmo usuário que avaliou ambas as versões do filme gerasse uma linha duplicada para o par (`userId`, `movieId`). Para mitigar isso, apliquei o `.drop_duplicates(keep='first')` logo após o mapeamento.
4. **Remoção de IDs Obsoletos:** No dataset de filmes, descartei os IDs duplicados redundantes, zerando a contagem de títulos duplicados.

Reexecutei todo o fluxo de threshold e filtros (Seção 2) e atualizei os indicadores de volumetria para prosseguir de forma segura para a Engenharia de Features.

---

# Engenharia de Atributos e Processamento de Texto

Primeiramente, removi a coluna `timestamp` do dataset de `ratings`.

No dataset de `movies`, decidi investigar primeiro a coluna `title`.

## Title

O objetivo foi extrair o ano presente na string do título, pois esse dado pode ser útil para o sistema de recomendação.

Ao extrair o ano, apenas uma instância apresentou problema:

*Fawlty Towers (1975-1979)* — caso já identificado na primeira abordagem.

Removi essa instância tanto do dataset de `movies` quanto do de `ratings`. Como se trata de uma série, e não de um filme, esse registro pode ser considerado um ruído em relação às características dos dados analisados.

Após a extração, foi armazenado o ano na coluna `year`, facilitando o uso dessa informação nas próximas etapas.

> **Dúvida**: Ainda fiquei em dúvida sobre manter ou remover o ano da coluna `title`.
> Se eu remover essa informação, alguns filmes passam a ter exatamente o mesmo título, diferenciando-se apenas pelo ano de lançamento. Por outro lado, se eu mantiver o ano no texto, existe o risco de o modelo inferir similaridade entre filmes apenas porque compartilham o mesmo ano, mesmo sendo obras completamente distintas.
> Não tenho certeza de qual abordagem introduz menos ruído na representação textual.

Optei por limpar a coluna `title`, removendo o ano e priorizando a redução de falsas similaridades geradas pela presença dessa informação no texto.

## Genres

Pretendo iniciar extraindo todas as possibilidades de gêneros presentes no *dataset*. Em tratamentos anteriores, percebi que existe um marcador como "`(no genres listed)`" em alguns registros, mas ainda não sei a quantidade exata. Também penso em remover completamente a coluna original `genres` e transformá-la em variáveis booleanas. Posso estar enganado e ainda irei validar isso, mas acredito que a técnica se chama *one-hot encoding*. Por exemplo: em vez de ter uma *string* como `'Adventure|Animation'`, a ideia é criar colunas onde `adventure` seja `True` ou `1`, `animation` seja `True` ou `1`, e `comedy` seja `False` ou `0`. Essa é a minha ideia inicial, mas preciso ver o que vai aparecer e se essa abordagem é recomendada.

No dataset `filtered_movies`, verifiquei que não há nenhum caso com o gênero `"(no genres listed)"`.

Utilizando o método `.str.get_dummies()` do Pandas, consegui mapear quantos gêneros existem e a quantidade de registros em cada um:

```text
action          1159
adventure        777
animation        250
children         357
comedy          1924
crime            796
documentary      109
drama           2486
fantasy          429
film-noir         58
horror           491
imax             126
musical          194
mystery          376
romance          958
sci-fi           589
thriller        1306
war              244
western           95
```

Apliquei a técnica de one-hot encoding e removi a coluna original genres.

Após isso, o dataframe *filtered_movies* passou a conter `22 colunas`, das quais apenas `title` é do tipo string (`str`) e o restante são inteiros (`int`).

---

# Exportação dos Dados

Com a conclusão da etapa `Engenharia de Atributos e Processamento de Texto`, os datasets limpos e otimizados foram persistidos em disco para serem consumidos diretamente pela etapa de modelagem, evitando a reexecução desnecessária do pipeline de limpeza.

Os dados foram salvos no diretório `../datasets/processed/`.

---

# Nota de Atualização (Retorno ao Marco 1) 
Após os testes iniciais de modelagem no Marco 2, identifiquei a necessidade de reprocessar a coluna `title` para tratar artigos no final (ex: "Matrix, The"). Com a nova coluna `search_title` gerada, o dataframe final *filtered_movies* passou a conter **23 colunas**, sendo duas do tipo string (`title` e `search_title`) e o restante numéricas (`movieId`, `year` e os gêneros em inteiros). Toda a pipeline do notebook `01_eda_and_cleaning.ipynb` foi executada novamente para consolidar essa nova estrutura.

---