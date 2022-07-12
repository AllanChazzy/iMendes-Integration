# Requisitos - Integração Consulta Tributária Ganso iMendes
---
Índice

- [Introdução](#introdução)
- [Requisitos Iniciais](#requisitos-iniciais)
  - [Cadastro de Empresas](#cadastro-de-empresas)
  - [Parâmetros de Comunicação](#parâmetros-de-comunicação)
  - [Regras Parametrizáveis](#regras-parametrizáveis)
  - [Finalidade da Operação](#finalidade-da-operação)
  - [Perfil Fiscal](#perfil-fiscal)
- [Requisitos Específicos](#requisitos-específicos)
  - [Recurso 1 - Consulta Tributos - Cadastro de Produtos](#recurso-1---consulta-tributos---cadastro-de-produtos)
    - [Fluxo de Consulta](#fluxo-de-consulta)
    - [Requisitos da Operação](#requisitos-da-operação)
      - [Funções Necessárias](#funções-necessárias)
      - [Campos Necessários](#campos-necessários)
      - [Tipos de Consulta](#tipos-de-consulta)
      - [Fluxo de Decisão do Método de Consulta](#fluxo-de-decisão-do-método-de-consulta)
      - [Composição da Tag do Emitente ```"emit"```](#composição-da-tag-do-emitente-emit)
      - [Composição da Tag Perfil ```"perfil"```](#composição-da-tag-perfil-perfil)
      - [Composição da Tag de Produtos ```"produtos"```](#composição-da-tag-de-produtos-produtos)
      - [Composição da Requisição - Resumo da Operação de Consulta](#composição-da-requisição---resumo-da-operação-de-consulta)
  - [Captura do Retorno das Informações](#captura-do-retorno-das-informações)
    - [Histórico de Consultas](#histórico-de-consultas)
      - [Atualizações Tributárias - Exibição de Dados](#atualizações-tributárias---exibição-de-dados)
    - [Relacionamento de Dados - Retorno iMendes x Ganso](#relacionamento-de-dados---retorno-imendes-x-ganso)
      - [Destino: Tabela Produto](#destino-tabela-produto)
      - [Destino: Tabela Produto Parâmetros](#destino-tabela-produto-parâmetros)
  - [Recurso 2 - Consulta Tributos em Lote - Gerenciador de Tributação](#recurso-2---consulta-tributos-em-lote---gerenciador-de-tributação)
    - [Regras da Consulta em Lote](#regras-da-consulta-em-lote)
    - [Métodos Padrão](#métodos-padrão)
      - [Interface do Usuário](#interface-do-usuário)
      - [Validações da Interface do Usuário](#validações-da-interface-do-usuário)
    - [Fluxo de Consulta em Lote](#fluxo-de-consulta-em-lote)
    - [Métodos Avançados](#métodos-avançados)
      - [Consulta Produtos Pendentes ou Devolvidos](#consulta-produtos-pendentes-ou-devolvidos)
      - [Remover Produtos da Base iMendes](#remover-produtos-da-base-imendes)
  - [Controle de Acesso e Acessos Restritos](#controle-de-acesso-e-acessos-restritos)
  - [Tabela de Validações do MVP](#tabela-de-validações-do-mvp)
  - [Relatórios Gerenciais - Logs](#relatórios-gerenciais---logs)
- [Simulações](#simulações)
  - [Consulta Cadastro de Produtos](#consulta-cadastro-de-produtos)
  - [Consulta em Lote - Regras de Entrada](#consulta-em-lote---regras-de-entrada)
  - [Consulta em Lote - Regras de Saída](#consulta-em-lote---regras-de-saída)

# Introdução

- O presente documento objetiva descrever em detalhes os processos e meios para integração da Consulta Tributária iMendes no Sistema Ganso. 
- Nesta integração, o Sistema Ganso comunica-se com o portal tributário do Grupo iMendes através de uma **API**, e realiza a consulta da Tributação dos produtos e retorna as informações. 
- A consulta pode ocorrer de duas maneiras: **durante a realização de um novo Cadastro** ou **consultando a tributação de um produto Cadastrado** 
- Para que a integração seja bem sucedida, é necessário que o CNPJ do estabelecimento consulente (Cliente que realizará as consultas) esteja liberado mediante aquisição de licença com o Setor Comercial do Grupo iMendes. (Levando em consideração que a Ganso Sistemas seja um parceiro ativo).

# Requisitos Iniciais
Nesta seção, serão descritos os **Requisitos Iniciais e Obrigatórios** para a continuidade do processo de integração, e descreve os principais parâmetros de Configuração. 
- Para o correto funcionamento das Requisições à API iMendes, dados do Emitente da Consulta são necessários e a maioria deles estão disponíveis no Cadastro de Empresa do Sistema Ganso. Contudo, há necessidade em criar estruturas complementares para as seguintes informações abaixo:

## Cadastro de Empresas
Nome/Texto | Descritivo | Validações | Obrigatório
|:------|:------|:-------|:-----:
Regime Tributário | Campo para informar a Subclassificação do CRT Regime Normal que pode ser definida entre **Lucro Real - LR** ou **Lucro Presumido - LP**. | Deve ser ativado apenas se o CRT selecionado for igual a 3 - Regime Normal. | **Sim**

## Parâmetros de Comunicação
São dados necessários para efetuar a comunicação com os Servidores da iMendes. Abaixo estão descritos quais informações devem ser criadas e suas funções.

Tipo de Elemento | Pai | Nome/Texto | Descritivo
:-----------|:------|:------|:------
**Caixa de Seleção** | - | Ativar Integração de Consulta Tributária iMendes | Parâmetro Global para ativação das configurações da Integração com o Sistema
**Grupo** | - | URLs da API | Organiza os campos de Configuração necessários para conectividade com o Servidor.
**Campo Texto** | **URLs da API** | Saneamento (Consulta Tributação) | Campo para informar a URL da API que retorna Dados da Tributação do produto consultado. Tamanho máximo de 255 caracteres
**Campo Texto** | **URLs da API** | Envia e Recebe Dados (Outros Métodos) | Campo para informar a URL da API que recebe os comandos extras e retorna produtos alterados. Tamanho máximo de 255 caracteres
**Campo Texto Mascarado** | **URLs da API** | Senha | Campo para Senha do Usuário Integrado. Tamanho máximo de 20 caracteres, sem validações. 
**Botão de Seleção** | **URLs da API** | Versão | Seleção da Versão da API contratada. Disponibilizar as versões 2.0 e 3.0 de seleção única (somente uma das opções pode ser selecionada)
**Campo Texto** | **URLs da API** | Tempo de Resposta | Timeout ou Tempo de Resposta máximo da API. Deve ser numérico e interpretado como "segundos". Valor padrão: 15 segundos
**Botão** | **URLs da API** | Verificar Status | Botão para testar a Conectividade com as APIs utilizando os dados informados. Deve retornar uma mensagem de Sucesso ou Falha, para ambas APIs.
**Botão de Seleção** | **URLs da API** | Ambiente | Seleção do Ambiente de Comunicação ativado. Pode ser definido entre **1 = Homologação e 2 = Produção**

## Regras Parametrizáveis
Tipo de Elemento | Pai | Nome/Texto | Descritivo
:--------|:------|:------|:------
**Grupo** | - | **Comportamento** | Organiza os parâmetros que definem as regras de funcionamento da integração e automatismos.
**Caixa de Seleção** | **Comportamento** | Atualizar Tributos dos Produtos Automaticamente | Permite atualização automatizada dos tributos de produtos ao consultar um Produto ou Receber atualizações em lote através do *Gerenciador Tributário*
**Caixa de Seleção** | **Comportamento** | Atualizar Grade Fiscal Ganso com Informações de Entrada Consultadas | Permite criar Regras Fiscais Ganso através das informações tributárias de Entrada coletadas em consultas.
**Caixa de Seleção** | **Comportamento** | Permitir Consultar Tributos no Cadastro de Produtos | Permite ao usuário efetuar a consulta iMendes durante o Cadastramento de Novo Produto ou Atualização de um Produto Cadastrado.
**Caixa de Seleção** | **Comportamento** | Permitir Vincular Produtos utilizando Código iMendes | Permite ao usuário vincular um Produto Cadastrado com um Produto iMendes Consultado por Descrição.
**Caixa de Seleção** | **Comportamento** | Verificar Alterações de Produtos Automaticamente em: [ x ] dias. | Parâmetro para definir a periodicidade de consulta à atualizações de grades realizadas pelo iMendes. Deve ser numérico e interpretado como "dias". Valor padrão: 15 dias
**Caixa de Seleção** | **Comportamento** | Permitir Atualização Parcial das Informações Tributárias do Produto | Parâmetro para definir se o usuário terá permissão em aceitar apenas campos específicos da Tributação consultada. Quando esta opção está desativada, não será permitido desmarcar campos na Tela exibição dos Tributos consultados
**Caixa de Seleção** | **Comportamento** | Permitir consultar mais de uma UF no iMendes | Parâmetro para definir se o usuário terá permissão em consultar mais de uma UF de destino em uma Requisição. Requer tratamento específico para armazenar os dados, e efetuar o correto tratamento e aplicação da regra.
**Caixa de Seleção** | **Comportamento** | Permitir consultar mais de uma Característica Tributária no iMendes | Parâmetro para definir se o usuário terá permissão em consultar mais de uma Característica Tributária de destino em uma Requisição. Requer tratamento específico para armazenar os dados, e efetuar o correto tratamento e aplicação da regra.
**Campo** | **Comportamento** | Limite de Produtos por Consulta em Lote | Parâmetro para definir um Limite de Produtos a serem enviados em Lote durante uma Consulta Tributária. O Limite máximo da iMendes é de 1000 produtos, contudo deve ser permitido configurar envios menores de acordo a conectividade do Usuário.

## Finalidade da Operação
A Consulta iMendes requer que uma Operação seja definida durante a composição da Consulta Tributária, que deve indicar um **Código CFOP** (correto ou não, mas válido e coeso com a operação). Deste modo, é necessário que o **Cadastro de Finalidade de Operações** seja adequado para compor a estrutura da requisição, e para que a criação de Regras Fiscais de Entrada e Saída seja possível. As alterações estão descritas abaixo:

Tipo de Elemento | Nome/Texto | Descritivo | Validações
:---|:---|:---|:---
Campo | Tipo de Movimentação | Campo para informar se a Finalidade da Operação é **Entrada ou Saída** | Obrigatório preenchimento/definição
Campo | CFOP Padrão Estadual | Campo para informar o Código do CFOP Padrão para a Operação Estadual cadastrada | Validar se CFOP é válido, se existe na Tabela de CFOPs do Sistema Ganso definido como **Entrada ou Saída**, e se o primeiro dígito é compatível com o Tipo de Movimentação: 1 para **Entrada** / 5 para **Saída**
Campo | CFOP Padrão Interestadual | Campo para informar o Código do CFOP Padrão para a Operação cadastrada | Validar se CFOP é válido, se existe na Tabela de CFOPs do Sistema Ganso definido como **Entrada ou Saída**, e se o primeiro dígito é compatível com o Tipo de Movimentação: 2 para **Entrada** / 6 para **Saída**
Campo | Finalidade do Produto | Campo do tipo código para informar se a Finalidade do Produto se enquadra nas opções: 0 = Revenda, 1 = Insumo ou 2 = Uso e Consumo ou Ativo Imobilizado | Informar pelo menos um dos códigos da lista 0 = Revenda, 1 = Insumo ou 2 = Uso e Consumo ou Ativo Imobilizado. Informação Obrigatória para realizar a consulta.

## Perfil Fiscal
Além de uma Operação bem definida, a iMendes requer que seja enviada uma **Característica Tributária** que no Sistema Ganso é definido como **Perfil Fiscal**. A partir desta integração, esta informação é fundamental para que o processo de Consulta ocorra de maneira correta e os dados tributários sejam retornados de acordo com a Operação que será realizada. Portanto, deve ser obedecida a Lista fornecida pela iMendes, conforme tabela abaixo:
Código | Descritivo
:---|:---
0 | Industrial 
1 | Distribuidor 
2 | Atacadista 
3 | Varejista
4 | Produtor Rural Pessoa Jurídica 
6 | Produtor Rural Pessoa Física 
7 | Pessoa Jurídica não Contribuinte do ICMS 
8 | Pessoa Física não Contribuinte do ICMS

[Voltar ao Sumário](#sumario)

# Requisitos Específicos
Nesta seção, são descritos os **Requisitos Obrigatórios** para atender à Homologação e os Recursos mais importantes desta integração, que são:
- Consulta via Cadastro de Produtos - Cadastrando um Novo Item
- Consulta via Cadastro de Produtos - Atualizando informações de um Produto Cadastrado
- Consulta em Lote para vários Produtos e Envio do Cadastro Completo para iMendes
- Controle de Acesso e Acessos Restritos (Operações Especiais).
- Tabela de Verificação de Conformidade com a Estrutura MVP sugerida pela iMendes.

## Recurso 1 - Consulta Tributos - Cadastro de Produtos
### Fluxo de Consulta
Conforme documentação disponibilizada pela iMendes o Usuário deve conseguir efetuar Consulta de Tributos no Cadastro do Produto, em dois momentos:
   1. Durante o Cadastramento de um Novo Produto;
   2. Durante o processo de Atualização de um Produto já Cadastrado.

De modo simplificado, os passos são:
   1. Iniciar o Cadastro ou Consultar um Produto Cadastrado;
   2. Acionar a função para Consultar Tributação iMendes;
   3. Receber e visualizar o Retorno com a Tributação antiga e a Nova;
   4. Confirmar ou não as alterações.

### Requisitos da Operação
Para disponibilizar ao usuário o recurso, é necessário incluir **Funções no Cadastro de Produtos** para que sejam acionadas quando o usuário desejar, e novos Campos, conforme descritivo abaixo:

#### Funções Necessárias
Nome | Descritivo | Validações
:-----|:-----|:------
Consultar Tributação iMendes | Acionar a Consulta de Tributos na API iMendes | Solicitar Chave de Acesso Restrito para realizar a Consulta.
Consultar Histórico de Alterações Tributárias | Acionar a visualização do Histórico de Alterações do Produto | -
Desfazer Alterações Tributárias | Acionar função para usuário reverter dados Tributários alterados pelo iMendes consultando o Histórico e com possibilidade de escolha do ponto de reversão desejado | Solicitar Chave de Acesso Restrito

#### Campos Necessários
Tipo | Posicionamento | Nome/Texto | Descritivo | Validações
:------|:------|:------|:------|:------
**Campo**|**Grupo Dados do Produto**|Código iMendes| Campo para armazenar e exibir o Código iMendes quando ocorrer o vínculo efetuado pelo Usuário | Deve ser do Tipo Inteiro e Somente leitura.
**Campo** | A Definir |Auditado por iMendes| Campo para armazenar e exibir a informação de que o Produto teve a tributação auditada/atualizada pela iMendes. Complementar esta informação com a Data e Hora da última atualização tributária | Somente leitura e visualmente destacado
**Campo** | A Definir |Enviado para Saneamento| Campo para armazenar e exibir a informação quando o Produto foi enviado para a iMendes classificar e tributar. Será utilizado para identificar que produtos estão pendentes de tributação na iMendes, e como *flag* para apontar que precisa receber atualização. Complementar esta informação com a Data e Hora do envio | Somente leitura e visualmente destacado
**Caixa de Seleção** | A Definir | **Não tributar pelo iMendes** | Parâmetro para restringir a atualização de Tributos do Produto pelo iMendes. Por decisão do Usuário, alguns produtos podem ser tributados seguindo a sua própria interpretação. | Solicitar Acesso Restrito para ativar e desativar o parâmetro.

#### Tipos de Consulta
Além dos campos e funções, é necessário criar Métodos de Consulta que trata o **Comportamento do Usuário** e direciona a Consulta para o fluxo correto de consumo das APIs. Todos os métodos são requeridos, e a API retorna informações distintas conforme método utilizado que são:

Método | Tipo de Consulta | Descritivo | Validações | API a Consumir | Tags de Envio Principais
:------|:------|:------|:------|:------|:------
**Método 1** | **Código de Barras EAN/GTIN e Descrição** | Consultar a Tributação do Produto utilizando o Código de Barras EAN/GTIN e a Descrição do Produto | Identificar se o Código é um EAN válido, considerando tamanho de 8, 12, 13 ou 14 dígitos. Considerar sempre 14 dígitos para envio, preenchendo com **Zeros** à esquerda caso EAN diferente de tamanho 14. Se EAN válido, identificar se o Prefixo do Código de Barras inicia em "789" ou "790", ou "1789" ou "1790" e enviar origem igual a 0, caso contrário enviar origem igual a 8. Capturar o retorno dos dados pela Tag ```"prodEAN"```.| **Saneamento** |```"codigo":"EAN", "codInterno":"N", "codIMendes":"", "descricao":"DESCRICAO"```
**Método 2** |**Apenas Descrição** | Consultar a Tributação de um Produto utilizando a Descrição. Capturar a lista de Produtos semelhantes, e permitir vincular um Produto iMendes com o Produto corrente. | Permitir vincular apenas um Produto iMendes com um Produto cadastrado. Criar acesso restrito para esta operação. Identificar se o Produto está sem Código de Barras e não vinculado a um Código iMendes.|**Envia/Recebe Dados**| ```"nomeservico":"DESCPRODUTOS", "dados":"CNPJ\|DESCRICAO" ```
**Método 3** | **Código iMendes e Descrição** | Consultar a Tributação do Produto utilizando o Código iMendes previamente vinculado. Capturar o retorno normalmente como ocorre no Método 1 | Identificar se o Produto possui um Código iMendes, se sim, utilizar esta informação para localizar a tributação. | **Saneamento** | ```"codIMendes":"CODIGOVINCULADO", "descricao":"DESCRICAO"```
**Método 4** | **Código Interno** | Consultar a Tributação do Produto utilizando o Código Interno do Produto previamento classificado pela iMendes |Identificar se o possui o campo "Enviado para Saneamento" igual a "Sim" |**Saneamento**| ```"codInterno":"CODIGOINTERNO"```

#### Fluxo de Decisão do Método de Consulta
- ![Fluxo de Decisão do Método de Consulta](./Flow-Decision-Method.jpeg)

Além dos Métodos de Consulta, para realização da Consulta, é necessário coletar as informações para geração da Requisição ```JSON```, iniciando pela Tag ```"emit"```:

#### Composição da Tag do Emitente ```"emit"```
Dado | Tag | Tipo | Descritivo | Origem dos Dados | Obrigatório
:---|:---|:---|:---|:---|:---
Ambiente | ```"amb"``` | Código | Tipo de Ambiente de Envio da Requisição. 1 = Homologação. 2 = Produção | **Parâmetros do Sistema/Aba iMendes** | **Sim**
CNPJ | ```"cnpj"``` | Caractere | CNPJ do Emitente da Consulta (Cliente) | **Cadastro de Empresas/CNPJ** | **Sim**
CRT | ```"crt"``` | Código | Código do CRT da Empresa. 1 = Simples Nacional, 2 = Simples Nacional com excesso de sublimite ou 3 = Regime Normal | **Cadastro de Empresas/CRT** | **Sim**
Regime Tributário | ```"regimeTrib"``` | Caractere | Regime Tributário da Empresa. Complemento do CRT 3. Pode ser definido entre 'LR - Lucro Real' e 'LP - Lucro Presumido' | **Cadastro de Empresas/Regime Tributário** | **Sim**
UF | ```"uf"``` | Caractere | UF do Emitente | **Cadastro de Empresas/UF** | **Sim**
CNAE | ```"cnae"``` | Caractere | CNAE do Emitente | **Cadastro de Empresas/CNAE** | Não
Substuto Tributário | ```"substICMS"``` | Caractere | Indicativo de Emitente Substituto Tributário. Se houver uma Inscrição Estadual de Substituto Tributário informada no Cadastro de Empresas, deve ser enviado esta informação tanto na tag ```"emit"``` quando na Tag ```"produtos"``` | **Cadastro de Empresas/Inscrição Substituto Tributário** | Não
Dia | ```"Dia"``` | Número | Dia da Vigência combinada com Mês e Ano para consultas específicas por Data | - | Não
Mês | ```"Mês"``` | Número | Mês da Vigência combinada com Dia e Ano para consultas específicas por Data | - | Não
Ano | ```"Ano"``` | Número | Ano da Vigência combinada com Dia e Mês para consultas específicas por Data | - | Não
Interdependente | ```"interdependente"``` | Caractere | Informação específica. Enviar sempre como "N" | - | Não

Coletados os dados do Emitente, é necessário coletar o **Perfil** a ser consultado. Os dados necessários foram classificadas e organizadas, e mesmo os não obrigatórios, devem ser enviadas com valores padrão. Conforme análise, o **objetivo de Consulta através do Cadastro** é obter dados para **Saída na Operação de Venda ao Consumidor (NFC-e)**, portanto os parâmetros definidos abaixo devem ser considerados em seus **Valores Padrão**:

#### Composição da Tag Perfil ```"perfil"```
Dado | Tag | Tipo | Descritivo | Valor Padrão | Obrigatório
:----|:---|:------|:------|:-------:|:-------:
UF | ```"uf"``` | Lista de Dados | Lista de UFs para Consulta de Regras. | UF da Empresa Filial Logada |**Sim**
CFOP | ```"cfop"```| Código da Operação | Código da Operação a ser Realizada. Deve ser enviada uma operação coerente com os dados desejados, por exemplo, uma Operação de Venda deve conter um CFOP que indique operação de Venda, mesmo que este não seja o correto. | "5102" |**Sim**
Característica Tributária | ```"caracTrib"``` | Lista de Códigos Inteiros | Indica o Tipo de Destinatário da Operação | Código "8" |**Sim**
Finalidade | ```"finalidade"```| Código | Indica a Destinação do Produto para a Operação informada. É importante para especificar a operação. | Código "0" | Não
Simples Nacional | ```"simplesN"```| Caractere | Indica se o Destinatário da Operação é Simples Nacional ou Não. Preenchido com "S" ou "N". Se CRT da Empresa é igual a 1 ou 2, enviar "S", senão enviar "N". | "N" | Não
Origem |```"origem"``` | Código | Indica a Origem da Mercadoria. Se Tipo de Consulta igual a Método 1, e Código de Barras não iniciar em 789 ou 790, enviar Código 8. | Código "0" | **Sim**
Substituição Tributária | ```"substICMS"```| Caractere | Indica se o destinatário é Substituto Tributário. | "N" | Não

Por fim, deve ser gerada a Tag ```"produtos"``` que contém os Produtos a serem Consultados, conforme descrito abaixo:

#### Composição da Tag de Produtos ```"produtos"```
Dado | Tag | Tipo | Descritivo | Origem dos Dados | Validações | Obrigatório
:---|:---|:---|:---|:---|:---|:---
Código | ```"codigo"``` | Código | Código de Barras EAN/GTIN ou Código Interno do Produto quando o mesmo foi enviado previamente para Saneamento pela iMendes | **Cadastro de Produtos/Código** | Verificar o Método de Consulta utilizado para preenchimento deste campo. [Ver Métodos de Consulta](#tipos-de-consulta) | **Sim**
Código Interno | ```"codInterno"``` | Caractere | Indicativo "Sim" ou "Não" para consulta via Código Interno, quando o Produto foi enviado previamente para Saneamento pela iMendes | "S" ou "N" | Verificar se o Produto está marcado como "Enviado para Saneamento" e o Método de Consulta utilizado. [Ver Métodos de Consulta](#tipos-de-consulta) | **Sim** 
Descrição | ```"descricao"```| Caractere | Descrição Completa do Produto | **Cadastro de Produtos/Descrição** | Enviar a Desrição Completa do Produto para que a resposta da API seja eficiente. | **Sim**
Código iMendes | ```"codImendes"```| Código | Código Único fornecido pela iMendes. Quando um produto é vinculado ao código iMendes esta informação poderá ser utilizada para consulta. | **Cadastro de Produtos/Codigo iMendes** | Verificar o preenchimento do Código iMendes que indica que houve a vinculação anterior. Utilizar o método de consulta que trata utiliza este código. [Ver Métodos de Consulta](#tipos-de-consulta) | **Não**
NCM | ```"ncm"```| Caractere | Nomenclatura Comum do Mercosul | **Cadastro de Produtos/NCM** | Verificar se NCM está preenchido no Cadastro do Produto. Esta informação não é obrigatória e a API retornará o Dado correto | **Não**

#### Composição da Requisição - Resumo da Operação de Consulta
1. Coleta dados do **Cadastro de Empresa** para gerar a gera a Tag ```"emit"```
3. Coleta dados do **Perfil de Envio** para gerar a Tag ```"perfil"```
4. Coleta dados do **Produto**, aplicando um dos **Métodos de Consulta** definidos em **Tipo de Consulta** para gerar a Tag ```"produtos"```
5. Enviar a Estrutura JSON de Requisição utilizando a **API** apontada no **Método de Consulta** utilizado. [Ver Métodos de Consulta](#tipos-de-consulta)

Após a obtenção dos Dados Necessários para envio da Requisição, o **Fluxo de Consulta** pode ser visualizado através do Esquema abaixo:   

- ![Fluxo de Consulta - Novo Produto](./Flow-Consulta-Produto.jpeg)

[Voltar ao Sumário](#sumario)
## Captura do Retorno das Informações
Após efetuar a Consulta e obter o retorno com os Dados Tributários, é necessário efetuar a Gravação de Histórico de Consulta. Abaixo são descritos quais são necessários:

### Histórico de Consultas
Nome do Histórico | Descritivo | Dados Principais
:----|:-----|:----
Consumo da API | Armanezenar todas as Consultas Realizadas por Usuário | Usuário, Data e Hora, CNPJ, Método de Consulta e Produtos Consultados
Retorno de Consulta | Armazenar os Retornos de cada Produto Consultado | Código de Produto, Data e Hora, Dados Completos Retornados da API, e informações invidivuais relacionadas com campos existentes no Sistema Ganso
Alterações de Produtos | Armazenar as Alterações Tributárias Promovidas pela iMendes relacionadas aos Produtos do Usuário | Código do Produto, Data e Hora, Dados Alterados para Consumo 
Dados Ignorados| Armazenar os Campos e Dados que foram ignorados pelo Usuário durante a Confirmação de Retorno | Armazenar por Produto e Usuário, Todos os campos e dados não selecionados pelo Usuário

Após captura e armazenamento dos dados consultados, um dos Requisitos para Homologação descreve que o Usuário precisa ter liberdade em **Aceitar** as atualizações Tributárias para o Produto total ou parcial, para gravá-las no Cadastro do Produto Consultado. Deste modo, é necessário exibir em Tela este retorno contendo todos os Dados **Antes e Depois**, destacando claramente quais apresentam divergências. Os elementos necessários nesta exibição estão descritas abaixo:

#### Atualizações Tributárias - Exibição de Dados
Campo | Informação de exibição | Validações
:---|:---|:---
Código Interno | Código do Produto Ganso | Somente Leitura
Descrição | Descrição do Produto Ganso | Somente Leitura
Código de Barras | GTIN/EAN do Produto Consultado | Somente Leitura
Código iMendes | Código iMendes Vinculado ao Produto Consultado | Somente Leitura
Grade de Dados "Tributação Atual" | Exibir todos os campos relacionados em [Destino: Produtos](#destino-tabela-produto) e [Destino: Produto Parâmetros](#destino-tabela-produto-parâmetros) com informações atuais gravadas no Produto no Sistema Ganso | Somente Leitura
Grade de Dados "Tributação Nova"| Exibir todos os campos relacionados em [Destino: Produtos](#destino-tabela-produto) e [Destino: Produto Parâmetros](#destino-tabela-produto-parâmetros) com informações **que a Consulta à API retornou** | Opção para Selecionar individualmente cada campo
Botão de Confirmação | Exibir uma Caixa de Diálogo com texto de confirmação de alteração de dados. Havendo campos parciais, incluí-los na mensagem para deixar claro ao usuário as alterações que serão realizadas | Solicitar Acesso Restrito para Gravar todos os Dados ou Dados Parciais

Abaixo estão descritas as informações Retornadas por Grupo e qual o destino da mesma no Sistema Ganso.

### Relacionamento de Dados - Retorno iMendes x Ganso
Nesta seção, são descritos os **Relacionamentos de Informações** retornadas pela API iMendes com a Estrutura do Sistema Ganso, indicando em Campo Destino onde deverão ser Gravadas as respectivas informações retornadas pela API.                  

#### Destino: Tabela Produto
Tag Pai | Campo Retornado | Campo Destino | Descritivo | Tratamento
:------|:-----|:------|:------|:------
Grupos | nCM | ncm | NCM do Produto | -
Grupos | cEST | cest | CEST do Produto | Remover a máscara retornada pelo iMendes
Grupos | codAnp | prod_esp_com_codigo_anp | Código da ANP | Gravar somente quando o produto é especifico do tipo Combustível
iPI | ex | ex_tipi | Exclusão da TIPI | -

#### Destino: Tabela Produto Parâmetros
Tag Pai | Campo Retornado | Campo Destino | Descritivo | Tratamento
:------|:-----|:------|:------|:------
```pisCofins``` |```cstEnt``` | cst_pis_entrada e cst_cofins_entrada | CST de Pis e Cofins de Entrada | Informar o mesmo retorno para ambos os Campos Destino. O CST de PIS e Cofins de Entrada são sempre iguais
```pisCofins``` | ```cstSai``` | cst_pis e cst_cofins | CST de Pis e Cofins de Saída | Informar o mesmo retorno para ambos os Campos Destino. O CST de PIS e Cofins de Saída são sempre iguais
```pisCofins``` | ```aliqPis``` | codigo_tributo_pis_entrada e codigo_tributo_pis_saida / f_pis_compra e f_pis_venda | Alíquota de Pis de Entrada e Saída | Gravar o Código do Tributo, resultante de uma consulta na Tabela **tributos** onde o campo "tipo" seja igual a 'PIS' e "aliquota" seja igual ao valor retornado. Gravar em ambos Campos Destino o "codigo" encontrado e Gravar o valor retornado nos campos f_pis_compra e f_pis_venda.
```pisCofins``` | ```aliqCofins``` | codigo_tributo_cofins_entrada e codigo_tributo_cofins_saida / f_cofins_compra e f_cofins_venda | Alíquota de Cofins de Entrada e Saída | Gravar o Código do Tributo, resultante de uma consulta na Tabela **tributos** onde o campo "tipo" seja igual a 'COF' e "aliquota" seja igual ao valor retornado. Gravar em ambos Campos Destino o "codigo" encontrado e gravar o valor retornado nos campos f_cofins_compra e f_cofins_venda.
```pisCofins``` | ```nri``` | cst_natureza_receita_piscofins | CST da Natureza da Receita de PIS e Cofins | Verificar a existência do Código de Natureza da Receita ("codigo_natureza_receita") na tabela **natureza_receita** onde os campos "cst_pis" e "cst_cofins" contém o 'cstEnt' ou 'cstSai', se existir, gravar no campo destino, se não existir, incluir esta nova natureza de receita na tabela **natureza_receita** preenchendo o campo "natureza_receita" com uma descrição genérica e os campos "cst_pis" e "cst_cofins" com as informações dos campos 'cstEnt' e 'cstSai' separados por vírgula.
```iPI``` | ```cstEnt``` | cst_ipi_entrada | CST de IPI de Entrada | Informar no campo de destino
```iPI``` | ```cstSai``` | cst_ipi | CST de IPI de Saída | Informar no campo de destino
```iPI``` | ```aliqipi``` | codigo_tributo_ipi | Alíquota de IPI de Saída | Gravar o Código do Tributo, resultante de uma consulta na Tabela **produto_tributo** onde o campo "situacao" seja igual a 1 e "cst" seja igual ao valor retornado e "situacao_tributaria" seja igual a 'T'
```CaracTrib``` | ```cFOP``` | codigo_cfop_nfc | Código da Operação CFOP de Saída | Informar no campo de destino
```CaracTrib``` | ```cST``` | cst e cst_nfc | CST de Saída | Concatenar o dígito 0 como prefixo e informar o mesmo retorno em ambos os campos relativos. A informação é utilizada tanto para NFC-e quanto para NF-e
```CaracTrib``` | ```cSOSN``` | csosn e csosn_nfc | CSOSN de Saída | Informar o mesmo retorno em ambos os campos relativos. Esta informação só deve ser gravada quando o CRT da Empresa for igual a 1 ou 2.
```CaracTrib``` | ```reducaoBcIcms``` | f_rbc_icms_sai_estadual | Redução de Base de Cálculo do ICMS Estadual | Informar no campo de destino o valor retornado
```CaracTrib``` | ```reducaoBcIcmsSt``` | f_st_rbc_icms_sai_estadual | Redução de Base de Cálculo do ICMS ST Estadual | Informar no campo de destino o valor retornado
```CaracTrib``` | ```reducaoBcIcmsStInterestadual``` | f_st_rbc_icms_sai_interestadual | Redução de Base de Cálculo do ICMS ST Interestadual | Informar no campo de destino o valor retornado
```CaracTrib``` | ```iVA``` | f_st_mva_saida | Margem de Valor Agregado de Saída Estadual | Informar no campo de destino o valor retornado
```CaracTrib``` | ```iVAAjust``` | *f_st_mva** | Margem de Valor Agregado Interestadual | Informar no campo de destino o valor retornado. 
```CaracTrib``` | ```fCP``` | *percentual_fcp** | Percentual de Fundo de Combate à Pobreza de Entrada | Informar no campo de destino o valor retornado
```CaracTrib``` | ```codBenef``` | cod_beneficio_fiscal | Código do Benefício Fiscal | Informar no campo de destino o valor retornado
```CaracTrib``` | ```pDifer``` | *percentual_diferimento** | Percentual de Diferimento de Entrada | Informar no campo de destino o valor retornado
```infPDV``` | ```pICMSPDV``` | codigo_tributo | Alíquota do ICMS de Saída ECF/NFC-e | Gravar o Código do Tributo, resultante de uma consulta na Tabela **produto_tributo** onde o campo "situacao" seja igual a 0 e "cst" seja igual ao valor retornado e "situacao_tributaria" seja igual ao campo **simbPDV** da Tag Pai **infPDV** e gravar no campo relativo o "codigo" encontrado.


* Os campos sinalizados deverão ser discutidos em reunião de planejamento. Será necessário criar versões para saída estadual e interestadual, se houver desenvolvimento de uma Regra Fiscal de Saída.

[Voltar ao Sumário](#sumario)

## Recurso 2 - Consulta Tributos em Lote - Gerenciador de Tributação
Nesta Seção, são descritos os Requisitos para Consulta de Tributação de vários produtos em Lote, que atende aos requisitos da iMendes para homologação. Este recurso também permitirá ao usuário enviar o próprio Cadastro de Produtos para revisão tributária pela iMendes.

### Regras da Consulta em Lote
Regra | Descritivo | Limite Máximo | Limite Recomendado | Validações
:---|:---|:---:|:---:|:---
R001 | Limitação de Quantidade de Produtos em Lote | 1000 | 100 | Limitar o Envio de Produtos em Lote por Consulta
R002 | Limitação de Quantidade de UFs de Envio | 26 | 5 | Limitar o Envio de UFs em Lote por Consulta
R003 | Limitação de Tempo de Resposta da API | 5 min. | 3 min. | Limitar o Tempo de Resposta para efetuar nova Consulta

### Métodos Padrão
A integração iMendes oferece opção para usuário consultar a tributação de vários produtos em uma única requisição, observando as limitações da API e Regras definidas. Para que o recurso seja oferecido ao usuário, é necessário criar uma **Tela de Consulta em Lotes** que processará os Produtos selecionados pelo Usuário, conforme especificações a seguir.

#### Interface do Usuário
Os elementos necessários em uma interface de consulta em lote são:

Elemento |  Nome/Texto | Descritivo | Conjunto de dados | Validações
:---|:---|:---|:---|:---
Grupo de Filtros | Filtros | Deve conter os campos necessários para realizar filtros e localizar Produtos para compor um Lote | Empresa Filial, Descrição do Produto, Marca, Seção, Grupo, Subgrupo, Ambiente de Utilização, Fornecedor Padrão, Agrupamento de Preços, Referência Fabricante e Auxiliar, Localização, Status do Produto, Produtos com Código iMendes, Produtos Enviados para Saneamento | Validar: Código da Empresa, Hierarquia de Segmentação, Parâmetro "Não Tributar pelo iMendes" e Enviado para iMendes [Ver Campos do Produto](#campos-necessários)
Campo | Limite por Lote | Campo para usuário informar quantos produtos serão enviados no Lote de Consulta. [Ver Regra 001 Consulta em Lote](#regras-da-consulta-em-lote) | Parâmetro do Limite de envio de Produtos em Lote | Restringir envio de Lote de Produtos acima do Limite de 1000 ou obedecer o Parâmetro Configurado [Ver Parâmetros](#regras-parametrizáveis)
Botão de Ação | Pesquisar Produtos | Ação para Usuário acionar a pesquisa de produtos com os critérios informados no Grupo de Filtros | Todos os dados informados | Se nenhum filtro for preenchido, informar ao usuário que a Consulta retornará todos os Produtos, contudo apenas 1000 produtos poderão ser enviados de uma só vez devido ao Limite de Envio por lote.
Grade com Caixa de Seleção | Relação de Produtos Filtrados | Grade que exibe todos os Produtos encontrados com os filtros informados pelo Usuário permitindo selecionar apenas 1000 por vez. | Código do Produto, Código de Barras, NCM, CEST, Descrição, Marca, Seção, Grupo e Subgrupo | Selecionar apenas 1000 Produtos por vez. Informar ao Usuário o Limite quando atingido.
Texto Informativo | Total Selecionado e Total de Produtos Listados | Texto que exibe a contagem de Produtos selecionados pelo Usuário, e o Total Listado na pesquisa realizada pelo mesmo utilizando os filtros estabelecidos no topo da tela. | Total Selecionado / Total Listado | - 
Caixa de Combinação | Finalidade da Operação | Campo para definir a Natureza da Operação do Perfil a ser Consultado para os Respectivos Produtos. A operação trata-se do ```"cfop"``` e estas operações deverão ser definidas a partir da integração, o usuário não poderá definir este código manualmente e sim, o Sistema oferecer as Finalidades Cadastradas para Consulta. A iMendes requer uma Operação por Requisição, caso houver necessidade em Consultar mais de uma Operação para os mesmos produtos, é necessário criar uma Requisição para cada Operação | Cadastro de Finalidade da Operação. [Ver Seção Finalidade da Operação](#finalidade-da-operação) | Informar obrigatoriamente uma Finalidade de Operação.
Caixa de Combinação | Característica Tributária | Campo para definir a Característica Tributária do Perfil Consultado | Cadastro de Perfil Fiscal. [Ver Seção Perfil Fiscal](#perfil-fiscal) | Informar obrigatoriamente uma Característica Tributária
Campo | UFs de Origem/Destino | Campo para definir as UFs das quais o usuário deseja obter a tributação para a Operação definida | Campo Livre com validações | Validar a digitação do Usuário, verificando se é uma UF válida, e não permitir repetição. Limitar Preenchimento a 5 UFs. [Ver Regra R002 da Consulta em Lote](#regras-da-consulta-em-lote)
Botão de Ação | Consultar Tributação | Botão de Ação para ativar a Consulta em Lote dos produtos selecionados conforme os parâmetros definidos pelo Usuário | Todos os dados informados | -
Botão de Ação | Verificar Pendentes/Devolvidos | Botão para acionar a API **Envia/Recebe Dados** para verificar os Produtos alterados pela iMendes quando o cadastro de Produtos foi enviado para Saneamento ou para Consultar as Alterações Tributárias | Produtos marcados como "Enviado para Saneamento" [Ver Seção Cadastro de Produtos/Campos Necessários](#campos-necessários) | Verificar Operações Especiais para acionar a função

#### Validações da Interface do Usuário
Neste tópico, são descritos os comportamentos e interações que as Rotinas da **Consulta em Lote** devem realizar conforme as **Ações do Usuário**.

Ação | Validação | Resposta ao Usuário
:---|:---|:---
Ação **Consultar Tributos** | Verificar Produtos Marcados pelo Usuário que não possuam Código de Barras GTIN, Código iMendes e que Não foram enviados para Saneamento para iMendes |  Exibir uma mensagem de confirmação informando que "Alguns produtos poderão não obter retorno de tributação", exibir a quantidade de Produtos e Salvar estes para verificação posterior.


### Fluxo de Consulta em Lote
![Fluxo de Consulta em Lote](./Batch-Search-Flow.jpeg)
  

### Métodos Avançados
#### Consulta Produtos Pendentes ou Devolvidos
#### Remover Produtos da Base iMendes

## Controle de Acesso e Acessos Restritos
Item | Controle | Recurso | Descritivo | Grau de Importância
:---|:---|:---|:---|:---
1 | Alterar Configurações da Integração iMendes | Chave de Acesso | Solicitar Chave de Operação Especial para evitar que o Usuário ou Técnico faça alterações nas URLs de Comunicação, Senha e outras configurações da integração iMendes | **Alto**
2 | Ativar ou Desativar o parâmetro para Atualizar Tributos dos Produtos Automaticamente | Chave de Acesso | Solicitar Chave de Operação Especial para evitar que o Usuário Ative ou Desative a atualização automática de Tributação quando houver mudanças | **Médio**
3 | Permitir Consultar Tributos via Cadastro de Produtos | Chave de Acesso | Solicitar Chave de Acesso para Usuário realizar a Consulta Tributária através do Cadastro de Produtos | **Alto**
4 | Permitir Gravar Atualização de Tributação no Cadastro de Produtos | Chave de Acesso | Solicitar Chave de Acesso para evitar que um Usuário não autorizado e com acesso ao Cadastro grave uma Tributação através do Cadastro de Produtos | **Alto**
5 | Permitir Gravar Atualização **Parcial** de Tributação no Cadastro de Produtos | Chave de Acesso | Solicitar Chave de Acesso para evitar que um Usuário autorizado grave Campos Específicos de tributação (não todos) através do Cadastro de Produtos | **Alto**
6 | Permitir Reverter, a um estado anterior, Dados Tributários alterados pela iMendes | Chave de Acesso | Solicitar Chave de Acesso para restringir e registrar que o Usuário reverteu dados de tributação atuais para uma versão do histórico do Produto | **Alto**
7 | Permitir vincular um Código iMendes a um Produto sem Código de Barras | Chave de Acesso | Solicitar Chave de Acesso para restringir e registrar que o usuário está vinculando um Produto Cadastrado com um Código iMendes através de uma Pesquisa de Produto por Descrição. Esta operação assegura ciência da operação pelo usuário | **Alto**


## Tabela de Validações do MVP

Item | Requisito iMendes | Requisito Ganso | Grau de Importância | Implementado
:---|:---|:---|:---:|:---:
MVP 1 | Opção para Usuário definir que o Produto Cadastrado não deve ser enviado à iMendes | [Campos Necessários](#campos-necessários) do Produto | **Alto** | -
MVP 2 | Opção para Usuário enviar informações manualmente | [Recurso 2](#recurso-2---consulta-tributos-em-lote---gerenciador-de-tributação) - Consulta em Lote | **Alto** | -
MVP 3 | Captura de Retorno controlado por Log e **Reversão** de tributação atualizada de qualquer Produto em qualquer ponto do histórico | [Funções Necessárias](#funções-necessárias) do Produto | **Alto** | -
MVP 4 | Consultar a Tributação de um único produto através do Cadastro do Produto e receber as atualizações após consulta | [Consultar Tributação iMendes](#funções-necessárias) | **Alto** | -
VF 1 | Opção para Usuário vincular um Produto próprio com o Código iMendes | [Consultar Tributação iMendes/Método 2](#tipos-de-consulta) | **Alto** | -
VF 2 | Gravar e exibir no Cadastro do Produto um selo indicativo de auditoria da tributação pela IMendes | [Campos Necessários / Auditado por iMendes](#campos-necessários) | **Alto** | -
VF 3 | Gravar e exibir o Retorno da Consulta por Produto contendo todos os campos retornados pela API e permitir a seleção de campos individuais para atualização. Apresentar a relação **Antes e Depois** e um Aceite do Usuário | [Tela/Atualizações Tributárias](#atualizações-tributárias) | **Alto** | -
VF 4 | Efetuar verificação períodica de Atualizações Tributárias dos Produtos auditados por iMendes através do Método **Alterados** | [Regras Parametrizáveis/Atualizar Tributos Automaticamente/Consulta em Lote](#regras-parametrizáveis)| **Alto** | -
VF 5 | Criar um Relatório Gerencial para acompanhamento do Histórico de Mudanças Tributárias ocorridas em determinado período | [Relatórios Gerenciais - Logs](#relatórios-gerenciais---logs) |**Alto** | -
VF 6 | Implementar o **Simulador Tributário** que efetua uma verificação no Cadastro de Produtos e aponta os problemas tributários em Clientes ainda não integrados | - | **Médio** | - 
VF 7 | Implementar mensagem de **Sugestão de Contratação da iMendes** para captura de novos clientes | - | **Baixo** | -

## Relatórios Gerenciais - Logs
Construir Relatório de Auditoria dos Produtos iMendes, para exibir o Histórico de Alterações.

# Simulações

## Consulta Cadastro de Produtos
## Consulta em Lote - Regras de Entrada
## Consulta em Lote - Regras de Saída
