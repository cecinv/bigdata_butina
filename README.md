# Big Data com Spark — Algoritmo de Clusterização Butina

Este projeto foi desenvolvido no contexto da disciplina **Big Data com MapReduce - Spark**, cursada durante a pós-graduação em Ciência de Dados e Machine Learning.

O objetivo do trabalho foi implementar o algoritmo de **clusterização Butina**, com base no artigo de referência disponibilizado na disciplina, e avaliar seu comportamento considerando diferentes volumes de dados.

---

## Objetivo do Projeto

- Compreender e implementar o algoritmo de **clusterização Butina**, conforme descrito no artigo de referência;
- Implementar o algoritmo utilizando fingerprints binários e o **coeficiente de similaridade de Tanimoto** para medir a similaridade entre os indivíduos;
- Validar o funcionamento do algoritmo em um **cenário reduzido e sequencial** (20 indivíduos, com 10 características cada);
- Investigar limitações de escalabilidade da abordagem sequencial;
- Explorar a **paralelização do cálculo de similaridade** com Apache Spark em um cenário de grande volume de dados (80.000 indivíduos, com 166 características cada);
- Analisar o impacto do volume de dados no desempenho e na formação dos clusters.

---

## Abordagem e Metodologia

O projeto foi desenvolvido em duas etapas complementares, com objetivos distintos:

- uma etapa **sequencial**, voltada ao entendimento do fluxo do algoritmo de Butina;
- uma etapa **distribuída**, focada na escalabilidade e no desempenho com grandes volumes de dados.

Em ambas as etapas, o fluxo lógico do algoritmo permanece o mesmo, diferindo apenas na forma como o cálculo de similaridade é executado. As etapas a seguir descrevem o processo geral adotado no projeto, com destaque para os pontos em que a paralelização com Apache Spark é aplicada.

1. **Geração dos indivíduos**
   - Criação de fingerprints binários sintéticos;
   - Cada indivíduo é representado por um vetor binário de tamanho fixo;
   - Uso de uma *seed* para garantir reprodutibilidade dos experimentos.

2. **Cálculo da similaridade**
   - Utilização do **coeficiente de Tanimoto** para fingerprints binários;
   - Consideração apenas da similaridade entre bits iguais a 1, conforme definido no trabalho original de Butina (1999);
   - Definição de um limiar mínimo de similaridade para a formação dos clusters.

3. **Paralelização com Apache Spark**
   - Implementação aplicada **exclusivamente no cenário ampliado (80.000 indivíduos)**;
   - Uso do **Spark Core (RDDs)** para distribuir os dados;
   - Criação de RDDs particionados;
   - Utilização de variáveis **broadcast** para compartilhamento eficiente dos indivíduos entre os executores;
   - Aplicação da operação `mapPartitions` para paralelizar o cálculo de similaridade.

4. **Formação dos clusters**
   - Ordenação dos indivíduos pelo número de vizinhos (mais similares);
   - Seleção iterativa dos centróides;
   - Construção das esferas de exclusão do algoritmo de Butina;
   - Identificação de clusters e singletons.

---

## Evolução da Implementação

A implementação do algoritmo foi realizada de forma incremental, permitindo identificar limitações e orientar decisões de projeto ao longo do desenvolvimento.

A versão inicial, com um conjunto reduzido de indivíduos, teve caráter didático e foi fundamental para a compreensão detalhada do fluxo do algoritmo de Butina, desde o cálculo da similaridade de Tanimoto até a formação dos clusters por meio das esferas de exclusão.

Durante essa etapa, observou-se que o cálculo das similaridades constitui o principal gargalo computacional do algoritmo, especialmente devido ao crescimento quadrático do número de comparações à medida que o volume de dados aumenta.

Para a escalabilidade, optou-se por uma abordagem distribuída com Apache Spark. Inicialmente, foi avaliada a utilização de operações baseadas em DataFrames e `crossJoin`, que, embora intuitivas, apresentaram custo computacional elevado e inviável para grandes volumes de dados.

Como alternativa, a implementação final adotou uma abordagem baseada em **RDDs**, utilizando `mapPartitions` para paralelizar o cálculo de similaridade e **variáveis broadcast** para o compartilhamento eficiente dos dados. Essa estratégia evitou a geração explícita de todos os pares possíveis e reduziu significativamente a sobrecarga de comunicação entre os executores.

Essa abordagem mostrou-se adequada para escalar a execução do algoritmo até o cenário de 80.000 indivíduos, sem a necessidade de alterações na lógica central do algoritmo de Butina.

---

## Definição do Limiar de Similaridade

O limiar de similaridade foi definido como **0.5**, um valor intermediário adotado neste trabalho como parâmetro inicial exploratório para a formação dos clusters.

Esse valor foi assim escolhido também por:
- favorecer a **coesão intra-cluster**, evitando a formação de clusters excessivamente heterogêneos;
- preservar a **diversidade estrutural**, evitando a fragmentação excessiva em singletons.

Considerando que os fingerprints utilizados neste trabalho são **sintéticos e de alta dimensionalidade**, valores mais elevados tenderiam a gerar um número excessivo de singletons, enquanto valores mais baixos resultariam em clusters demasiadamente grandes.

---

## Estrutura do Repositório

- `20_Butina.ipynb`  
  Implementação **sequencial** do algoritmo de Butina com um conjunto reduzido de 20 indivíduos, cada um com 10 características.  
  Esta versão tem caráter **didático e exploratório**, sendo utilizada para compreensão do fluxo do algoritmo conforme descrito no artigo, **sem uso de Apache Spark**.

- `80k_Butina.ipynb`  
  Implementação do algoritmo com 80.000 indivíduos, cada um com 166 características, utilizando **Apache Spark**, voltada à escalabilidade em ambiente distribuído.


- `README.md`  
  Documento principal do projeto, contendo a descrição metodológica, decisões de implementação e análise dos resultados.

---

## Tecnologias Utilizadas

- Python  
- Apache Spark (Spark Core)  
- PySpark  
- bitarray  
- NumPy  

---

## Execução

1. Configure um ambiente com Apache Spark e PySpark;
2. Execute o notebook desejado:
   - versão com 20 indivíduos;
   - versão com 80.000 indivíduos;
3. Ajuste, se necessário:
   - o número de partições do RDD;
   - o limiar de similaridade;
4. Analise os clusters formados e os resultados apresentados ao final da execução.

---

## Artigo de Referência

A implementação do algoritmo de clusterização Butina neste projeto baseia-se no artigo:

Butina, D. (1999). *Unsupervised data base clustering based on daylight's fingerprint and Tanimoto similarity: A fast and automated way to cluster small and large data sets*. Journal of Chemical Information and Computer Sciences.

---

## Observações

Este projeto possui caráter **acadêmico e exploratório**, com foco no entendimento do algoritmo de Butina e na aplicação de conceitos fundamentais de **processamento distribuído com Apache Spark**.

O objetivo não é apresentar uma implementação otimizada para produção, mas sim analisar o comportamento do algoritmo em diferentes escalas de dados e justificar as decisões metodológicas adotadas.
