# Proposta de solução 

A seguir será apresentada a solução proposta para resolução de um desafio. Que basicamente consiste em:
- Elaborar uma solução que ofereça armazenamento, processamento e disponibilização de dados. Levando em consideração que a solução precisa suportar um grande volume de dados complexos e sensíveis.

#### Armazenamento

As informações são organizadas em 3 bases externas, com as seguintes características:

- **Base A**: Armazena informações extremamente sensíveis é deve ser protegida com os maiores níveis de segurança, entretando o acesso aos dados não precisa ser performático.
- **Base B**: Possui dados críticos, mas o acesso as informações deve ser um pouco mais rápido. Permite a extração de dados por meio de algoritmos de aprendizado de máquina.
- **Base C**: Não possui dados críticos, mas o acesso precisa ser extremamente rápido.

Nesse ponto ficou vago se nós acessaríamos diretamente as informações, ou se manteríamos os dados. Mas de qualquer forma acredito que uma opção ideal seria mantermos e gerenciarmos o armazenamento dessas 3 bases. Então considerando estas características, foram definidas as tecnologias a serem utilizadas em cada uma das bases e o motivo:

- **Base A**: 
Para esta situaçao a proposta e utilizar o **Amazon RDS for PostgreSQL**. Sobre o item de segurança, o **PostgreSQL** tem suporte SSL nativo para conexoes e para criptografar as comunicaçoes cliente/servidos.
Além disso **O Amazon RDS** permite criptografar o banco de dados usando chaves gerenciadas pelo [**KMS**](https://aws.amazon.com/pt/kms/). Em uma instância de banco de dados em execução com a criptografia do **Amazon RDS**, os dados ociosos mantidos no armazenamento subjacente são criptografados, bem como os backups automáticos, as réplicas de leitura e os snapshots desses dados.
O uso do [VPC](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html), também permite o isolamento das instancias na própria rede virtual e conectá-las a infraestrutura de TI existente utilizando uma **VPN IPsec** criptografada padrão do setor. Outro ponto é a possibilidade de configurar as definições do firewall e controlar o acesso de rede às suas instâncias de banco de dados.

- **Base B e Base C**:
Mais um serviço da AWS para conta, e nesse case que sugiro o uso do **Amazon Elasticsearch Service**. O Elasticsearch foi escolhido nesses 2 casos pelos seguintes motivos:
  - Por ter confiabilidade e segurança como o PostgreSQl. (E o AWS por todo o gerenciamento, escalabilidade, segurança). (Base B)
  - Por trabalhar muito bem com [Machine Learning](https://www.elastic.co/pt/what-is/elasticsearch-machine-learning). (Base B)
  - Por ter um tempo de resposta muito rápido, já que internamente ele mantém os dados em cache para deixar o resultado ainda mais performático. (Base C)

Ainda sobre o armazenamento, fiz algumas pesquisas, e considerei um cenário onde disponibilizamos apis para registro desses dados. No desafio foram citados alguns dos dados que as bases armazenam:
Base A : CPF, Nome, Endereço, Lista de dívidas
Base B: Idade, Lista de Bens (imóveis, etc), Endereço, Fonte de renda
Base C: Movimentações financeiras, dados de compras em cartões, consultas em outros serviços similares ao do detentos do desafio, tudo filtrando por CPF.

Considerando esse cenário, acredito ser necessário disponibilizar apis para registro dessas informações.
Uma sugestão seria: 
- Expor uma api RESTful que permitisse o registro de todas as informações cadastrais e financeiras. Quando um CPF nunca foi registrado, e quando vem apenas atualizações de dívidas, movimentações bancárias e etc. Desenvolveria essa api em Spring Boot, devido a facilidade de integração com as duas stacks de banco citadas acima.
- Criar um nano-serviço, podendo ser um lambda que dispare uma prévia do calculo do score, após o registro/atualização das informações utilizas no cálculo.

#### Tráfego

Sobre este item, no desafio foi comentado que os dados são acessados através de micro-serviços e nano-serviços, e foram elencados os dados que são acessados em cada base de dados.
Isso auxiliou na definição das tecnologias para as bases de dados elencadas no item anterior.

Base A acessa os seguintes dados:
- CPF
- Nome
- Endereço
- Lista de dívidas
Base B acessa os seguintes dados:
- Idade
- Lista de Bens (imóveis, etc)
- Endereço
- Fonte de renda
Base C acessa os seguintes dados: 
- Rastreia dados pelo CPF
   - Ultima consulta do CPF em outros serviços similares ao detentor do desafio
   - Movimentações financeiras
   - Dados relacionados a ultima compra com cartão de crédito.

Sobre o payload de consulta, eu acrescentaria algumas informações no acesso aos dados:
- **Base A**. Na lista de dívidas, deixaria registrado dívidas que já foram quitadas, porém que foram registrada no sistema anteriormente.
- **Base B**. Informações referentes a financiamentos imobiliários e veiculares que a pessoa possui.

Como eu citei anteriormente sobre o cálculo do score, tentaria armazenar as informações na base do ES já com uma prévia do cálculo do score, ou seja, alguns dados já pré-processados. Eu desconheço como o cálculo é feito, mas acredito que de para armazenar algums informações pré-processadas.
Assim dependendo de como o cálculo funciona, eu poderia utilizar uma função lambda em NodeJS para calcular o score final e apresentar ao cliente.
Acredito que este cálculo não deva demorar muito para ser executado, mas caso seja, a api poderia ser desenvolvida em outra tecnologia, como NodeJS, ou GO.

#### Disponibilização dos Dados

Nesse ponto, acredito que deveria existir duas visões, uma para empresas consultarem os dados, e outra para a pessoa dona do CPF.
Desenvolveria uma única interface para ambas as visões, utilizando basicamente ReactJS. Com estas funcionalidade:

- Visão Empresa:
    - Consulta de devedores da empresa
    - Consulta da movimentações financeiras
    - Consulta do score de crédito

- Visão Pessoa Física:
    - Consulta das informações registradas
    - Consulta das dividas em aberto
    - Consulta da situação das negociações
    - Consulta do score de crédito (se for permitido)
    
#### Considerações Finais

Sobre as API's é possível fazer o uso do [localstack](https://localstack.cloud/) que fornece uma estrutura para simular o ambiente AWS. E isso facilita o desenvolvimento, e testes locais.
Utilizando o PostgreSql é possivel utilizar o [liquibase](https://www.liquibase.org/), para rastrear, gerenciar a aplicar alterações na estrutura do bando de dados.
Também sugiro fazer uso do [docker-compose](https://docs.docker.com/compose/), onde todos os serviços utilizados pelas apis podem ser configurados.
Uso do Gitlab CI/CD, configurando o pipeline para publicação, entre outras vantagens.
