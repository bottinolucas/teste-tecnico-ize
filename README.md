# Teste técnico - IZE Gestão Empresarial

Este repositório é dedicado para a resolução da fase 2 do processo seletivo de Engenheiro(a) de Dados Júnior da IZE Gestão Empresarial. Esta etapa consiste em um pequejo projeto prático que envolve a resolução de dois desafios.

A empresa busca um Engenheiro de Dados para ajudar a otimizar a logística de entrega de produtos. O desafio foi preparado para avaliar a capacidade do candidato de resolver problemas e projetar soluções de dados.

## Desafio 1 

**CONTEXTO**: A empresa utiliza diferentes sistemas para registrar informações sobre envios e entregas. O desafio envolve o processamento de dados de um arquivo CSV que contém informações de rastreamento de pacotes. 

-   Arquivo recebido: rastreamento.csv
    - Colunas: id_pacote, origem, destino, status_rastreamento, data_atualizacao 

Perguntas sobre o desafio 1 

1.  Descreva uma pipeline de ingestão de dados para processar esse arquivo CSV. Qual ferramenta ou tecnologia você usaria em cada etapa (ingestão, processamento, armazenamento)?
2.  Proponha um modelo de dados para armazenar as informações de rastreamento de forma eficiente em um banco de dados relacional (SQL). Considere a possibilidade de um pacote ter múltiplos eventos de rastreamento ao longo do tempo (Ex: "EM TRANSITO", "EM ROTA", "ENTREGUE")
3.  Descreva a abordagem escolhida e justifique a escolha das tecnologias e o design do modelo de dados. 

## Desafio 2 

**CONTEXTO**: A equipe de análise de negócios precisa monitorar a eficiência das entregas para identificar gargalos e otimizar rotas. Para isso, precisamos de um sistema de relatórios que consuma os dados de rastreamento.

Perguntas sobre o desafio 2

1.  Proponha uma solução para criar um dashboard em tempo quase real que mostre o número de pacotes por status (EM TRÂNSITO, ENTREGUE, etc) e a média de tempo de entrega (do envio até a entrega final).
2.  Quais seriam as implicações se, no futuro, a empresa passar a receber dados de rastreament oem tempo real via um endpoint de API, em vez de um arquivo CSV diário? O que você precisa adaptar no seu pipeline e modelo de dados?

## SOLUÇÃO DO DESAFIO  1

A pipeline de ingestão de dados utilizada para resolver o desafio 1 é baseada em um processo de ingestão por lotes, onde o arquivo `rastreamento.csv` é a fonte de dados utilizada e o banco de dados SQL Postgres é o destino final. 

A arquitetura utilizada para construir essa pipeline foi a arquitetura de medalhão, bastante comum na construção de data warehouses e baseando-se nos estágios bronze, prata e ouro, com o objetivo de construir um Data Mart (subset do Data Warehouse para uma análise específica, neste caso, rastreamento de pacotes): 

![Pipeline de Ingestão de Dados](/assets/pipeline-ingestao-dados.png)

1.  Camada Bronze (DADOS BRUTOS)
    
    A camada bronze é o ponto de partida da pipeline, recebendo os dados da fonte original sem qualquer alteração.

    Arquivos presentes:

        rastreamento.csv

2.  Camada Silver/Prata (Limpeza e Transformação)

    A camada silver/prata é a etapa onde os dados presentes na camada bronze são limpos, padronizados e transformados para se adequarem ao modelo relacional de dados.

    Arquivos presentes:

        etl_bronze_to_silver.ipynb
        rastreamento_tratado.csv

3.  Camada Gold/Ouro (Modelagem de dados e armazenamento final)

    Por fim, a camada Gold/Ouro é a camada responsável por aplicar a regra de negócio, garantindo que os dados sejam armazenados de forma estruturada no banco de dados.

    Arquivos presentes:

        etl_silver_to_gold.ipynb

O fluxo de dados foi implementado por meio de dois scripts ETL que processam os dados de forma incremental e organizada, oferecendo rastreabilidade, garantia de qualidade e escalabilidade do pipeline. 

As tecnologias utilizadas foram a linguagem Python em conjunto com as bibliotecas Pandas e Pandera para manipulação da base de dados e validação da mesma com schemas. O SQLAlchemy permitiu mapear os objetos para tabelas do banco de dados, simplificando a persistência por não ser necessário utilizar strings para modelar as querys. O PostgreSQL foi o banco de dados relacional escolhido por ser robusto, open-source e surpotar tipos avançados, ideal para os dados estruturados e relacionamentos entre entidades. Por fim, o Docker foi a ferramenta que garantiu o isolamento do ambiente e a reprodutibilidade do projeto, executando o banco de dados de forma consistente em qualquer máquina.

O design do modelo de dados foi projetado para duas entidades principais:

![Diagrama Entidade-Relacionamento](/assets/diagrama-entidade-relacionamento.png)

1.  Pacote (id_pacote, origem, destino)
2.  Rastreamento (id_rastreio, status_rastreamento, data_atualizcao, id_pacote)

O relacionamento é 1:N, onde um pacote pode possuir vários eventos de rastreamento ao longo do seu processo de entrega. 

![Diagrama Lógico de Dados](/assets/diagrama-logico-dados.png)

## SOLUÇÃO DO DESAFIO 2

1. Proponha uma solução para criar um dashboard em tempo quase real
que mostre o número de pacotes por status (EM TRANSITO, ENTREGUE,
etc.) e a média de tempo de entrega (do envio até a entrega final).

Para a construção de um sistema de relatórios que consuma os dados de rastreamento, o ideal é que a modelagem do sistema esteja implementada como um Data Warehouse, ideal para construção de relatórios e BI. Essa estrutura permite consolidar informações históricas e facilitar a criação de relatórios e dashboards analíticos.

Como a eficiência no monitoramento e velocidade do dashboard são consideradas importantes regras de negócio, uma modelagem menos normalizada e otimizada para leitura é indicado, sendo o esquema de estrela (star schema) o mais adequado. Nesse modelo, a tabela fato armazena os dados de rastreamento e as tabelas dimensão fornecem o contexto necessário para as análises e agregações de dados. 

Devido à necessidade do dashboard precisar ser monitorado em tempo quase real, o Grafana ou Prometheus se apresentam como as ferramentas ideais para gerar as visualizações necessárias por consultar os dados diretamente da fonte e serem indicados para monitoramento contínuo de dados, métricas e atualizações em tempo quase real, diferentemente de ferramentas tradicionais de BI como o Power BI ou Tableau.

Caso ocorra a troca do recebimento de dados de um arquivo csv para um endpoint em tempo real via API, mudanças importantes deverão ocorrer para que a pipeline se mantenha estável e funcional. 

2. Quais seriam as implicações se, no futuro, a empresa passar a receber
dados de rastreamento em tempo real via um endpoint de API, em vez
de um arquivo CSV diário? O que você precisaria adaptar no seu pipeline
e modelo de dados?

    2.1. Como os dados são recebidos por um arquivo diário, isso significa que são blocos de dados que são passados para o banco de dados, ocorrendo o processamento por lotes (batch). Com a implementação da obtenção de dados via endpoint de uma api, os dados serão transmitidos de forma contínua à medida que os eventos ocorrerem, transformando o ambiente de ingestão de dados para um processamento em fluxo (streaming).

    2.2. A pipeline de ingestão deverá ser adaptada de um modelo que processa em lotes para um modelo que processa continuamente dados. Com isso, ao invés de ler o(s) arquivo(s) uma vez por dia, o sistema deve receber e processar cada evento à medida que ele ocorre, com as etapas de tratamento e cálculo das métricas ocorrendo de forma paralela e mantendo as agregações atualizadas.

    2.3. O modelo de dados precisaria evoluir para um modelo orientado a eventos pois a cada atualização de status de um pacote o registro deve ser persistido como um evento independente, com suas respectivas mudanças (status_rastreamento, horário), permitindo calcular as métricas de forma incremental e mantendo o histórico de movimentações.

    2.4. O sistema precisaria de camadas de validação mais complexas para lidar com o fluxo contínuo de dados e possíveis erros de duplicação de eventos.