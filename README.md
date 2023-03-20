# AZURE---DataPipeline---OlistBrazilian-E_Commerce #
## Escopo do projeto ##
Este projeto visa a cronstrução de um pipeline completo para automação das etapas de ETL, utilizando das mais diversas funções disponibilizadas pela cloud AZURE.

## Base de dados ##
A Olist é uma empresa brasileira de tecnologia que oferece soluções para e-commerce e marketplace, ajudando pequenos e médios empreendedores a venderem seus produtos online. Sua plataforma permite que os empreendedores criem uma loja virtual integrada a um marketplace, ampliando as possibilidades de venda. Além disso, a empresa oferece uma solução completa de logística e suporte ao cliente para seus parceiros.
A base de dados escolhida foi a da OLIST, disponível no kaggle.
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce?resource=download

<center>Segue descrição da arquitetura da base de dados. </center>

![image](https://user-images.githubusercontent.com/120025497/226148405-da9eb15a-7c8f-48ea-9193-d4fe3465e550.png)

## Arquitetura do projeto ##
O projeto utiliza a arquitetura em nuvem da Microsoft, <b>Azure</b>, que é baseada em serviços em nuvem. O Azure oferece recursos e ferramentas para ajudar no desenvolvimento e implantação de aplicativos, além de garantir alta disponibilidade, segurança e desempenho.

![image](https://user-images.githubusercontent.com/120025497/226148620-60539001-41e1-488f-8de7-55f5684eaa8a.png)

Alguns dos serviços utilizados foram:
  - Azure Data Factory
  - Azure Key Vault
  - Azure Storage Account(Data Lakes)
  - Azure Databricks Workspace
  - Azure Synapse Workspace
  - SQL Server
  - SQL Database
  - Azure Active Directory
  
Além do gerenciamento de assinaturas, permissões, regiões, urls, tokens, linked services, clusters, pipelines, triggers, pools, cost estimations, dentre outros.

## Descrição do projeto ## 

- Nesta seção irei destacar alguns passos e desafios enfrentados ao decorrer deste projeto.
O primeiro passo do projeto foi a importação da base de dados do keagle(link na Seção Base de Dados).
![image](https://user-images.githubusercontent.com/120025497/226148984-2429edc2-4115-4244-a911-8de2e6de23f3.png)

- Feito isto, e após ser criada uma conta na plataforma da Azure, a primeira coisa a ser feita é a criação de um grupo de recursos, que nada mais é que um contêiner lógico que agrupa recursos(meio óbvio, não!?) relacionados a um projeto ou solução. Essa prática traz diversos benefícios, como a possibilidade de gerenciar e monitorar vários recursos ao mesmo tempo e a facilidade na implantação e remoção de recursos.
Em seguida foi criado um servidor SQL de modo a simular uma situação real em que os CSV's estariam disponiveis para EXTRAÇÃO (o E de E-T-L), é neste local que que os dados extraídos do KAGGLE(Olist) serão armazenados!!

- Uma vez que a base de dados está preparada, agora é hora de começar a subir os recursos necessários para o nosso Data Lake, mas o que é isso? 
Data Lake é um repositório centralizado de dados em grande escala, que armazena dados brutos de várias fontes em seu formato original. O objetivo é possibilitar o acesso rápido e fácil a grandes volumes de dados para análise e tomada de decisão, sem a necessidade de transformá-los em um formato estruturado e organizado. 
E então é criada uma **conta de armazenamento(Storage Account)** para hospedar nosso Data Lake, e dentro dela, criam-se os containeres landing, processing e curated para armazenamento dos dados de maneira organizada.

![image](https://user-images.githubusercontent.com/120025497/226149492-b34d71d8-2e3d-4487-8414-ee93fdcd778f.png)

- Data Lake OK, Base de dados OK.. Chega a hora de iniciar a construção do pipeline! Mas afinal, o que é Pipeline de dados?
- É um conjunto de processos automatizados que permitem a transferência de dados de uma ou mais fontes para um ou mais destinos!! Resumindo: É o CORE do nosso projeto!
- E para tal tarefa, o AZURE disponibiliza uma ferramente INCRÍVEL chamada **Data Factory**, responsável por orquestrar todo fluxo de dados! A primeira tarefa a ser realizada por este pipeline
é a Extração dos dados do SQL Server para a LANDING de nosso Data Lake. A imagem que segue ilustra a configuração do ForEach que permite que o Data Factory itere sobre uma lista de itens, no nosso caso os CSV's, e realize uma determinada operação
,no caso copiar de um local, para o outro.

![2 -pipeline   for each todos de uma vez](https://user-images.githubusercontent.com/120025497/226150021-f2afeca9-7050-4abe-9e24-7f09aff4c8ac.jpg)

- Após a execução da parte de Extração do pipeline, é verificado na LANDING que de fato a operação foi realizada com sucesso!
![4 - apos execucacao do pipeline_verificado o povoamento de nossa landing](https://user-images.githubusercontent.com/120025497/226150161-7d7443fd-a373-4a66-ac26-ca20332ef508.jpg)

- O próximo passo é iniciar a realização da fase de PROCESSING. E para isto, é criado um workspace no DATABRICKS. Mas afinal o que é isto? 
- O DATABRICKS é uma plataforma de análise de dados poderosa e flexível que oferece recursos avançados para processamento de dados MASSIVO, de maneira distribuída, o que o torna altamente escalável e elástico.
![6 - criando cluster no databriks, o mais basico possivel](https://user-images.githubusercontent.com/120025497/226150379-ac34a39f-9fed-4a4c-97a3-99ca9229d3af.jpg)
Na imagem acima é criado um cluster no Databricks! Ps: com as configurações mais básicas possíveis(e mais baratas!!), pois a carga de processamento de nosso trabalho não requer maiores configurações.
Ps: Tive um certo problema ao tentar realizar a criação do cluster a priori(imagem do erro abaixo), mas depois de várias tentativas frustradas descobri que o problema estava na região que eu havia criado o workspace do databricks(CENTRAL US). Havia escolhido
esta devido ao custo ser menor. Porém devido ao erro, tive de escolher outra região mais cara... não teve jeito!
![ERRO criação do cluster databriks](https://user-images.githubusercontent.com/120025497/226232437-633ca8eb-fab5-41df-b5d4-fc165eee705d.jpg)


- Nesta etapa são realizados algumas operações utilizando de PySpark como transformação dos dados para formato PARQUET,criação de um servidor SQL para disponibilização dos dados através de tabelas e databases.(O que facilita o trabalho para quem se utiliza de soluções para DataViz como Power BI).
- Porém, antes de mais nada, é necessário realizar a montagem de um sistema de arquivos externo para que nós consigamos ter acesso aos contêirenes que foramm criados no Data Lake. No Databricks isso é possível através da chamada do método **dbutils.fs.mount**, que faz parte da API do Databricks Utilities(DBUtils). Vale ressaultar que para termos acesso a esse sistema de arquivos externo ao Databricks, é necessário um mecanismo de autenticação bem criterioso com a criação de escopo secreto e um segredo do app(segue a imagem da parte do código PySpark que trata desse processo de autenticalçao e dos segredos mencionados)
![autenticação databricks - azure](https://user-images.githubusercontent.com/120025497/226233011-df0e06d3-9d94-41e7-9cdd-9db1cef4f093.jpg)
![8 - criacao do segredo do app](https://user-images.githubusercontent.com/120025497/226233029-e34d0dfa-9a87-458e-a1af-7c5258e1728b.jpg)
![7 - criacao do escopo secreto](https://user-images.githubusercontent.com/120025497/226233037-83d313fa-bc21-4fd1-92c7-bed11a6eda0c.jpg)


Uma vez que o Databricks agora tem acesso ao nosso Data Lake, é hora de realizar a integração desse junto ao Data Factory para, por fim, finalizarmos o pipeline de dados. Para recapitular: O pipeline de dados é uma solução automatizada para mover, transformar e processar grandes volumes de dados de diferentes fontes. Nesse contexto, a Data Factory extrai os dados e envia para o Databricks realizar o processamento e transformação, e depois disponibiliza os dados para outras aplicações ou sistemas de armazenamento de dados, tudo isso orquestrado pela Data Factory. É impressionante a versatilidade que o Databricks oferece para tratamento dos dados! Se quiseres pode trabalhar através de consultas SQL puro! Ou se preferir, pode transformar os CSV's em parquet e guardá-los em um Dataframe! Segue imagem de uma parte do processo de conexão entre o Data Factory e o Databricks, que é a geração de um token no Databricks.
![9 - gerando Token para conexão do datafactory com databricks, relacionado ao pipeline](https://user-images.githubusercontent.com/120025497/226235371-bab7607a-8bd6-4426-ac8e-93e11b2fcdbf.jpg)


- No nosso exemplo, todos os CSV's foram transformados em parquet e alocados na zona PROCESSING do Data Lake. De modo a ilustrar uma situação, filtrei da table CUSTOMER, apenas os que residiam na cidade do Rio de Janeiro(dentro de customers, os registros que possuiam atributo customer_state = 'RJ'), através de uma simples consulta SQL. De posse de um dataframe filtrado, foi realizado a transformação deste em um CSV e disponibilizado no conteider CURATED. 


- Segue imagem que ilustra o pipeline final, no qual de maneira automatizada e integrada, são realizadas todas as fases do famoso ETL: Dados CV's extraídos do banco de dados, processados em parquet, e por fim, disponibilizados novamente em CSV e prontos para consumo.
![10 - pipeline rodou de ponta a ponta com sucesso](https://user-images.githubusercontent.com/120025497/226236720-183eeedf-07aa-4987-baca-be89b9721544.jpg)

### Maneira alternativa de realizar construção de pipeline ###
- E se em seu ambiente de trabalho, por algum motivo, você não tiver acesso a um ambiente de computação distribuída como o Azure Databricks, ou até mesmo uma arquitetura de banco de dados madura de modo a viabilizar o trabalho com estes dados? A plataforma Azure, ainda contém uma outra ferramenta fantástica para trabalharmos com Big Data e Análise de dados que é o **Azure Synapse**.
- Azure Synapse é uma plataforma de análise de dados que combina recursos de data warehousing e big data. Ele permite que as empresas integrem e analisem grandes volumes de dados de diferentes fontes, realizem análises avançadas e tomem decisões informadas em tempo real. Possui grande similaridade com o databricks, porém com um GRANDE diferencial: O Synapse tem seu próprio mecanismo de processamento de consulta e armazenamento de dados distribuídos, chamado **SQL on-demand**.
- O SQL on-demand é um serviço de análise de dados do Azure Synapse que permite executar consultas SQL sob demanda em grandes volumes de dados armazenados em formatos como Parquet e CSV diretamente no armazenamento do Azure, sem a necessidade de provisionar ou gerenciar infraestrutura de banco de dados.
É uma abordagem "serverless", onde o processamento de consultas é gerenciado automaticamente pela plataforma, com escalabilidade automática sob demanda, sem que seja necessário gerenciar clusters de banco de dados ou provisionar capacidade de armazenamento. Isso permite que os usuários do Azure Synapse executem análises ad-hoc em grande escala em seus dados sem preocupações com a complexidade da infraestrutura de banco de dados.

- Na imagem abaixo é realizada a leitura dos nossos arquivos diretamente na LANDING, através de uma simples consulta SQL
![11 - Utilizando SYNAPSE para recuperar dados diretamente do data lake](https://user-images.githubusercontent.com/120025497/226238950-09f7e390-b046-4283-986a-597cc7962691.jpg)

- Se tenho acesso aos arquivos diretamente dos conteineres, e acesso ao SQL, é possivel automatizar as ações de um pipeline somente com Synapse e Data Factory? A resposta é: Sim! Através de Stored Procedures! As Stored Procedures podem ser usadas para executar qualquer operação de transformação de dados ou consulta SQL. No caso deste projeto, foi criado uma SP que executa a criação de uma view com o conteúdo da table Orders.
![12 - criação de VIEW ATRAVES de Stored Procedure](https://user-images.githubusercontent.com/120025497/226240450-759a0459-ff28-4815-a680-5e2cdbe28099.jpg)

- No Data Factory, é feita a integração do serviço de Extract já previamente configurado, com a execução da Stored Procedure, finalizando assim, uma outra abordagem de construção de pipeline de dados, dentro da mesma plataforma!
![13 - criação de pipeline alternativa via SYNAPSE totalmetne serverless](https://user-images.githubusercontent.com/120025497/226240858-5c0d0fa8-c7a9-45f7-8207-66e8b43b5c7c.jpg)


# Conclusão #

Em suma, este projeto apresentou a construção de um pipeline completo que proporcionou a automação de todas as etapas do processo de ETL. A utilização de ferramentas e serviços disponibilizados pela **cloud AZURE** permitiu a implementação de soluções escaláveis e eficientes, além de garantir a segurança dos dados. A possibilidade de integração entre as diferentes ferramentas também facilitou a configuração do pipeline. Foram apresentadas duas possíveis soluções para o problema proposto, e ambas se apresentaram muito eficientes.
