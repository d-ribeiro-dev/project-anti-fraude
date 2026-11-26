# Detecção de Fraudes em Cartão de Crédito

## Problema real do projeto

Ensinar o computador a identificar fraudes em cartão de crédito (`class = 1`) no meio de compras normais (`class = 0`).

## O desafio: desbalanceamento

* **99,83%** das transações são compras normais (`0`).
* Apenas **0,17%** são fraudes (`1`).

Se um modelo chutar "tudo é compra normal" em 100% dos casos, ele acerta em **99,83%** das vezes, mas em contrapartida não pega nenhuma fraude — que é o prejuízo real. Por isso, não podemos olhar só para os acertos gerais; precisamos acompanhar:

* **Precision (Precisão):** Quando o modelo diz *"é fraude sim"*, quantas vezes ele acerta? Isso ajuda a evitar que bloqueemos compras legítimas.
* **Recall (Revocação / Sensibilidade):** De todas as fraudes que realmente existiam, quantas o modelo conseguiu pegar? Ajuda a evitar que fraudes passem batidas.
* **F1-score:** O equilíbrio entre precisão e recall na classe de fraude (`1`).

## O que fiz

### 1. Carreguei os dados e conferi a proporção

Fiz a leitura direto da URL pública com `pandas` e conferi quantas linhas eram `0` e quantas eram `1`. Ler direto pela URL evita baixar arquivos pesados soltos na máquina no início e já mostra de cara o tamanho do desbalanceamento.

### 2. Preparei os dados e separei treino e prova

Comprimi os valores extremos (como `Amount_log`) para que números muito altos não distorcessem tanto o aprendizado, garantindo que tanto treino quanto prova tivessem a mesma proporção de fraudes, com divisão fixa para repetir o experimento sempre igual caso houvesse repetição no código. Separar os dados simula o modelo vendo transações novas.

### 3. Padronizei a escala e treinei os 3 modelos

Apliquei o `StandardScaler` no treino e apenas apliquei no teste (`fit_transform` no treino e `transform` no teste) para evitar vazamentos. Depois treinei:

* **Regressão Logística**
* **Random Forest** (`class_weight='balanced'`)
* **XGBoost** (`scale_pos_weight`)

Modelos lineares funcionam melhor e mais rápido quando as variáveis estão em escalas parecidas, e dar peso maior às fraudes ajuda elas a não serem ignoradas, já que são o nosso foco e são raras.

### 4. Avaliei e comparei os resultados

Coloquei os números lado a lado e usei gráficos (como Curva ROC, Precisão vs. Recall e comparativos) para facilitar a troca entre pegar mais fraudes e gerar mais alarmes falsos.

### 5. Pipeline, ajuste de régua e busca de ajustes

* **Pipeline:** Juntei a padronização e a Regressão Logística num fluxo só para deixar tudo organizado e evitar mistura de dados entre treino e teste.
* **Ajuste do ponto de corte:** Testei baixar a régua de decisão de `0.5` para `0.3` para ver o que acontece quando tentamos pegar mais fraudes — ganha-se em alcance, mas aumentam os alarmes falsos.
* **Busca de ajustes (`GridSearchCV`):** Testei combinações de ajuste focando em não deixar fraudes passarem (`recall`), chegando a `C = 1.0` como melhor opção entre o que foi testado.

### 6. Olhando por dentro do modelo (explicabilidade)

Para não ficar só nos números gerais, fui ver o que mais pesou nas decisões do Random Forest:

* **O que mais pesou:** Variáveis como `V14`, `V10`, `V4`, `V12` e `V11` ficaram no topo. E o `Amount_log` ficou um pouquinho acima do valor bruto (`Amount`), mostrando que comprimir o valor ajudou.
* **SHAP:** Usei uma amostra pequena para não pesar o computador e gerei o gráfico resumo, que ajuda a enxergar não só quais colunas importam mais, mas também como elas empurram a decisão para um lado ou para o outro.

## Por que essas ferramentas?

* **VS Code + Notebook:** Tenho mais afinidade e posso rodar bloco por bloco com ele, verificando na hora qualquer imprevisto.
* **pandas:** Padrão para carregar, inspecionar e separar colunas com clareza.
* **numpy:** Operações matemáticas rápidas.
* **scikit-learn (`sklearn`):** Principal biblioteca de Machine Learning em Python, com uso simples e consistente.
* **matplotlib:** Simples e direta para gerar gráficos.
* **Regressão Logística:** Simples, rápida e mostra o quanto um limite mais simples consegue fazer antes de usar modelos mais complexos.
* **Random Forest:** Captura relações mais variadas entre colunas e lida bem com dados em tabela, com ótimo equilíbrio no teste.
* **XGBoost:** Muito forte em dados em tabela, com alto recall para pegar fraudes reais.

## O que fiz de diferente em relação às aulas

Segui o caminho proposto pela Expert, mas desenvolvi meu notebook no VS Code, onde tenho mais afinidade.

Durante o desenvolvimento, também fiz algumas adaptações:

* Calculei automaticamente as métricas dos modelos e criei um gráfico comparativo.
* Comparei os pontos de corte utilizando também a validação cruzada.
* Utilizei uma amostra balanceada para a análise SHAP e investiguei individualmente uma fraude detectada.
* Analisei as consequências dos falsos positivos e falsos negativos.
* Registrei as limitações encontradas e possíveis melhorias relacionadas à segurança, validação e confiabilidade dos resultados.

## Conclusão

No fluxo do projeto, visualizei os dados, defini X e y, separei em treino e prova, treinei, testei, avaliei e comparei os resultados de forma visual. Depois disso, testei diferentes pontos de corte (0.5 e 0.3) para ajustar como o modelo lida com as fraudes — equilibrando pegar mais casos reais com o cuidado de não gerar alarmes falsos demais.

No final, aprendi que cada modelo e cada ajuste faz uma escolha diferente entre precisão e recall — e cada escolha tem um impacto diferente na prática.

## Resultados no conjunto de teste

## Modelo                 Precision Recall    F1-score

Regressão Logística      83,12%      65,31%    73,14%

Random Forest             91,86%      80,61%    85,87%

XGBoost                     87,10%      82,65%    84,82%

Regressão Logística                                                            balanceada e otimizada    6,12%      91,84%   11,47%

O Random Forest apresentou o maior F1-score. A regressão balanceada e otimizada apresentou o maior recall (90 das 98 fraudes), mas muitos falsos alertas. Minha preferência é investigar formas de preservar o recall e reduzir esses alertas, sem bloquear compras automaticamente. A diferença em relação à regressão inicial não pode ser atribuída apenas ao GridSearchCV, pois também houve balanceamento de classes.

## Limitações e possibilidades futuras

* A seleção de hiperparâmetros e a comparação por validação cruzada utilizaram o mesmo conjunto de treinamento; o teste também já foi consultado. Uma avaliação independente seria necessária para confirmar a generalização.
* As curvas ROC e Precision-Recall e as explicações SHAP apresentadas referem-se ao Random Forest.
* Não foram avaliados custos financeiros dos erros nem capacidade operacional de investigação.
* Como possibilidades futuras, proponho ajustar pesos e limiares, investigar uma segunda camada de verificação, sinalizar divergências entre modelos e estabelecer regras de auditoria, monitoramento e revisão humana. Essas propostas não foram implementadas neste desafio.
