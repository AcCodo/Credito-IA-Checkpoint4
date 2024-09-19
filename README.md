# Credito-IA-Checkpoint4
Projeto de estudo - Uso de Machine Learning para prever grupo de crédito e valores aprovados.

## Equipe:
- André Campos Codo
  - 87145
- Gabriel Ribeiro Itagyba
  - 87098
- Isabella de Souza Campos
  - 89244
- Isabelle Raslosnek Ziteli Panico
  - 87608
 
## Setup
Para o setup do projeto baixe os dois principais arquivos:
- Checkpoint4_IA.ipynb
- solicitacoescredito.csv

Importe o python notebook para o Google Colab e importe o .csv no seu Google Drive numa pasta de nome "coisas do colab/" ou renomeie o caminho de acordo com a sua vontade no primeiro bloco de código no Colab, mudando a parte com o nome da pasta para o caminho de sua escolha. (O mesmo terá que ser feito na segunda parte da execução do código, na categoria de Predição de Valor Aprovado.
Ao executar, o Google Colab irá solicitar sua permissão para montar o Drive junto do Colab e você deve autorizar para que o .csv seja usado no código.

Após isso, apenas execute o notebook por completo.

## Definição dos Problemas
- 1: Detalhamento da Análise e Identificação de Grupos
- 2: Prever grupo e aprovação do cliente
- 3: Prever valor de crédito aprovado

### Problema 1: Identificação de Grupos
- Para identificar grupos, nós criamos um Índice de Balanço. Somando valores positivos, e subtraindo negativos.
  - Somamos: ativoCirculante, passivoCirculante, totalAtivo, totalPatrimonioLiquido e faturamentoBruto
  - Subtraímos: endividamento e duplicatasAReceber
- A partir do IB (Índice de Balanço), e do Valor Solicitado para crédito podemos usar algoritmos como KMeans para identificar os grupos, mas antes alguns passos extra:
  - Removemos solicitações que tem status: DocumentacaoReprovada, EmAnaliseDocumentacao e AguardandoAprovacao. Por não serem relevantes a análise.
  - Removemos solicitações que são outliers a partir da remoção de solicitações com Valor Solicitado acima de 50% do Terceiro Quartil, e abaixo de 50% do Primeiro Quartil. 
  - Balanceamos todos os status restantes (AprovadoAnalista, AprovadoComite, ReprovadoAnalista e ReprovadoComite) por status de AprovadoAnalista ter 80% das solicitações de treino.
  - E preenchemos todos os casos vazios de informação com 0.0 no lugar
  - Então separamos 20% das solicitações para analisar os resultados depois e treinamos o modelo KMeans a partir dos outros 80% dos dados.
- Clusterização no final aparenta fazer sentido conforme imagem abaixo:
- ![image](https://github.com/user-attachments/assets/0bb73507-584c-48f7-a1b4-c6a522a673f1)

Agora que temos as solicitações agrupadas em 4 principais grupos de solicitações, podemos partir para o segundo problema

### Problema 2: Prever aprovação
- Com os grupos criados, fica mais fácil o uso de algoritmos de predição, como por exemplo: KNNClassifier, que usaremos nesse caso. A partir da distribuição de status diferentes em cada um dos 4 grupos o uso de KNN fica prático, sem precisar de tratamento de dados diferentes para o caso.

Em um dos nossos testes, tivemos os seguintes resultados:
78.18% de Acurácia e 79.38% de F1 Score.

Com a distribuição de valores de Status entre os grupos a seguir:

    Grupo    Status Previsto     Quantidades
    0        AprovadoAnalista     261
             AprovadoComite        19
             ReprovadoAnalista     13 
             ReprovadoComite        1
         
    1        AprovadoAnalista      29
             AprovadoComite         1
             
    2        AprovadoAnalista      61
             AprovadoComite         1
             ReprovadoAnalista      1
             
    3        AprovadoAnalista      41
             ReprovadoAnalista     11
             AprovadoComite         1

Os resultados nos deixaram satisfeitos por parecem representar apropriadamente a proporção entre uma precisão e recall (F1 Score).

### Problema 3: Prever Valor Aprovado de Crédito
- Nesse caso, recomeçamos o tratamento de dados, por ser uma análise completamente diferente, mas aproveitamos algumas regras utilizadas anteriormente.
  - Removemos status indesejados, mas dessa vez, além dos status referentes a Documentação, também removemos status Reprovados, por não terem valores de crédito algum.
  - Como da última vez, removemos alguns outliers, seguindo a mesma lógica do agrupamento: Valor Solicitado acima de 50% do Terceiro Quartil, e abaixo de 50% do Primeiro Quartil.
  - E preenchemos os valores vazios (NaN) com 0.0, afim de não perder muitas linhas de informação.
- Então selecionamos essencialmente todas as colunas com valores numéricos.
  - 'maiorAtraso', 'margemBrutaAcumulada', 'percentualProtestos', 'prazoMedioRecebimentoVendas', 'titulosEmAberto', 'valorSolicitado', 'ativoCirculante', 'passivoCirculante', 'totalAtivo', 'totalPatrimonioLiquido', 'endividamento', 'duplicatasAReceber', 'estoque', 'faturamentoBruto', 'margemBruta', 'periodoDemonstrativoEmMeses', 'custos', 'capitalSocial', 'scorePontualidade', 'limiteEmpresaAnaliseCredito'.
  - O motivo da seleção é que acreditamos que a análise é feita com uma verdadeira mistura de vários fatores, que todos devem ser considerados de certa forma para a regressão.
- Selecionamos apenas 80% dos valores disponíveis para nós, para usar os outros em testes, e utilizamos o modelo: RandomForestRegression

Após a execução do treinamento e testes percemos os seguintes resultados para avaliação:
- R-squared: 0.7409
- MSE: 63,983,855.6947
- RMSE: 7,998.9909
- MAE: 5,546.4189

Devido a um R² elevado, chegamos a conclusão de que os campos selecionados e tratativas feitas foram suficientes para uma boa acurácia no nosso modelo.
