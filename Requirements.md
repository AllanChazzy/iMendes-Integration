# Requisitos - Integração Consulta Tributária Ganso iMendes
---
# Sumário
- [Sumário](#sumário)
- [Introdução](#introdução)
- [Requisitos Iniciais](#requisitos-iniciais)
  - [Cadastro de Empresas](#cadastro-de-empresas)
  - [Parâmetros do Sistema](#parâmetros-do-sistema)
  - [Protótipo de Configurações e Parâmetros:](#protótipo-de-configurações-e-parâmetros)
- [Requisitos Específicos](#requisitos-específicos)
  - [Consulta Tributos - Cadastro de Produtos](#consulta-tributos---cadastro-de-produtos)
    - [Fluxo de Consulta](#fluxo-de-consulta)
  - [Requisitos - Consulta iMendes Cadastro de Produtos](#requisitos---consulta-imendes-cadastro-de-produtos)
  - [Campos](#campos)
  - [Tipos de Consulta](#tipos-de-consulta)
  - [Fluxograma da Consulta Tributação](#fluxograma-da-consulta-tributação)
  - [Consulta Tributos - Gerenciador de Tributação](#consulta-tributos---gerenciador-de-tributação)
    - [Métodos Básicos](#métodos-básicos)
    - [Métodos Avançados](#métodos-avançados)
  - [Relatórios Gerenciais - Logs](#relatórios-gerenciais---logs)

# Introdução

- O presente documento objetiva descrever em detalhes os processos e meios para integração da Consulta Tributária iMendes no Sistema Ganso. 
- Nesta integração, o Sistema Ganso comunica-se com o portal tributário do Grupo iMendes através de uma **API**, e realiza a consulta da Tributação dos produtos e retorna as informações. 
- A consulta pode ocorrer de duas maneiras: **durante a realização de um novo Cadastro** ou **consultando a tributação de um produto Cadastrado** 
- Para que a integração seja bem sucedida, é necssário que o CNPJ do estabelecimento consulente (Cliente que realizará as consultas) esteja liberado mediante aquisição de licença com o Setor Comercial do Grupo iMendes. (Levando em consideração que a Ganso Sistemas seja um parceiro ativo).

# Requisitos Iniciais
Nesta seção, serão descritos os **Requisitos Iniciais e Obrigatórios** para a continuidade do processo de integração, e descreve os principais parâmetros de Configuração. 
- Para o correto funcionamento das Requisições à API iMendes, dados do Emitente da Consulta são necessários e a maioria deles estão disponíveis no Cadastro de Empresa e nos Parâmetros do Sistema Ganso. Contudo, há necessidade em criar Estruturas para as seguintes informações abaixo:

## Cadastro de Empresas

Tipo | Rotina/Recurso | Descritivo | Validações | Obrigatório
:------|:------|:------|:-------|:-----:
**Caixa de Combinação** | Regime Tributário | Campo para informar a Subclassificação do CRT que pode ser definida entre **Lucro Real - LR** ou **Lucro Presumido - LP**. | Deve ser ativado apenas se o CRT selecionado for igual a 3. | **Sim**

## Parâmetros do Sistema
Tipo de Elemento | Pai | Nome/Texto | Descritivo
:-----------|:------|:------|:------
**Caixa de Seleção** | - | Ativar Integração de Consulta Tributária iMendes | Parâmetro Global para ativação das configurações da API
**Grupo** | - | URLs da API | Organiza os campos de Configuração necessários para conectividade com o Servidor.
**Campo Texto** | **URLs da API** | Saneamento (Consulta Tributação) | Campo para infomrar a URL da API que retorna Dados da Tributação do produto consultado. Tamanho máximo de 255 caracteres
**Campo Texto** | **URLs da API** | Envia e Recebe Dados (Outros Métodos) | Campo para informar a URL da API que recebe os comandos extras e retorna produtos alterados. Tamanho máximo de 255 caracteres
**Campo Texto Mascarado** | **URLs da API** | Senha | Campo para Senha do Usuário Integrado. Tamanho máximo de 20 caracteres, sem validações. 
**Botão de Seleção** | **URLs da API** | Versão | Seleção da Versão da API contratada. Disponibilizar as versões 2.0 e 3.0 de seleção única (somente uma das opções pode ser selecionada)
**Campo Texto** | **URLs da API** | Tempo de Resposta | Timeout ou Tempo de Resposta máximo da API. Deve ser numérico e interpretado como "segundos". Valor padrão: 15 segundos
**Botão** | **URLs da API** | Verificar Status | Botão para verificação da Conectividade com as APIs. Deve retornar uma mensagem de Sucesso ou Falha, para ambas APIs.
**Grupo** | - | **Comportamento** | Organiza os parâmetros que definem as regras de funcionamento da integração e automatismos.
**Caixa de Seleção** | **Comportamento** | Atualizar Tributos dos Produtos Automaticamente | Permite atualização automatizada dos tributos de produtos ao consultar um Produto ou Receber atualizações em lote através do *Gerenciador Tributário*
**Caixa de Seleção** | **Comportamento** | Atualizar Grade Fiscal Ganso com Informações de Entrada Consultadas | Permite criar Regras Fiscais Ganso através das informações tributárias coletadas em consultas.
**Caixa de Seleção** | **Comportamento** | Permitir Consultar Tributos no Cadastro de Produtos | Permite ao usuário efetuar a consulta iMendes durante o Cadastramento de Novo Produto ou Atualização de um Produto Cadastrado.
**Caixa de Seleção** | **Comportamento** | Permitir Vincular Produtos utilizando Código iMendes | Permite ao usuário vincular um Produto Cadastrado com um Produto iMendes Consultado por Descrição.
**Caixa de Seleção** | **Comportamento** | Verificar Alterações de Produtos Automaticamente em: [ x ] dias. | Parâmetro para definir a periodicidade de consulta à atualizações de grades realizadas pelo iMendes. Deve ser numérico e interpretado como "dias". Valor padrão: 15 dias

## Protótipo de Configurações e Parâmetros: 
  
  ![Protótipo de Parâmetros](./parametros.png)

[Voltar ao Sumário](#sumario)

# Requisitos Específicos
Nesta seção, serão descritos os **Requisitos Obrigatórios** para atender à Homologação e os Recursos mais importantes desta integração, que são:
- Consulta via Cadastro de Produtos - Cadastrando um Novo Item
- Consulta via Cadastro de Produtos - Atualizando informações de um Produto Cadastrado
- Consulta em Lote para vários Produtos
- Controle de Acesso e Acessos Restritos (Operações Especiais).
- Verificação de Conformidade com a Estrutura MVP sugerida pela iMendes.

## Consulta Tributos - Cadastro de Produtos
### Fluxo de Consulta
Conforme documentação disponibilizada pela iMendes o Usuário deve conseguir efetuar Consulta de Tributos no Cadastro do Produto, em dois momentos:
   1. Durante o Cadastramento de um Novo Produto;
   2. Durante o processo de Atualização de um Produto já Cadastrado.

De modo simplificado, os passos são:
   1. Aciona o Botão de Consulta iMendes;
   2. Recebe o Retorno com a Tributação;
   3. Confere as informações e Confirma.

## Requisitos - Consulta iMendes Cadastro de Produtos
Para disponibilizar ao usuário o recurso, é necessário incluir uma **Função no Cadastro de Produtos** para que seja acionada quando o usuário desejar, e novos Campos, conforme descritivo abaixo:

## Campos
Tipo | Pai | Nome/Texto | Descritivo | Validações
:------|:------|:------|:------|:------
**Botão** | **Tributação / Fiscal** | Consultar iMendes | Botão para acionar a Consulta de Tributos na API iMendes.| Solicitar Chave de Acesso Restrito para realizar a Consulta.
**Campo Inteiro**|**Dados do Produto**|Código iMendes| Campo para armazenar o Código iMendes quando ocorrer o vínculo. Tipo Inteiro | Somente leitura.
**Caixa de Seleção** | - | **Não tributar pelo iMendes** | Parâmetro para restringir a atualização de Tributos do Produto pelo iMendes. Por decisão do Usuário, alguns produtos podem ser tributados seguindo a própria interpretação. | Solicitar Acesso Restrito para ativar o parâmetro.

## Tipos de Consulta
Os Métodos de consulta descritos abaixo identificam o fluxo a ser tomado para consumo das APIs. Todos os métodos são requeridos, e a API retornará a informação conforme método utilizado.

Método | Tipo | Descritivo | Validações | API | Tags de Envio Principais
:------|:------|:------|:------|:------|:------
**Método 1** | **Código de Barras EAN/GTIN e Descrição** | Consultar a Tributação do Produto utilizando o Código de Barras EAN/GTIN e a Descrição do Produto | Identificar se o Código é um EAN válido, considerando tamanho de 8, 12, 13 ou 14 dígitos. Se EAN válido, identificar se o Prefixo do Código de Barras inicia em 789 ou 790, e enviar origem igual a 0, caso contrário enviar origem igual a 8. Capturar o retorno dos dados.| **Saneamento** |"codigo":"EAN", "codInterno":"N", "codIMendes":"", "descricao":"DESCRICAO"
**Método 2** |**Apenas Descrição** | Consultar a Tributação de um Produto utilizando a Descrição. Capturar a lista de Produtos semelhantes, e permitir vincular um Produto iMendes com o Produto corrente. | Permitir vincular apenas um Produto iMendes com um Produto cadastrado. Criar acesso restrito para esta operação. Identificar se o Produto está sem Código de Barras e não vinculado a um Código iMendes.|**Envia/Recebe Dados**|"nomeservico":"DESCPRODUTOS", "dados":"CNPJ|DESCRICAO"
**Método 3** | **Código iMendes e Descrição** | Consultar a Tributação do Produto utilizando o Código iMendes previamente vinculado. Capturar o retorno normalmente como ocorre no Método 1 | Identificar se o Produto possui um Código iMendes, se sim, utilizar esta informação para localizar a tributação. | **Saneamento** | "codIMendes":"CODIGOVINCULADO", "descricao":"DESCRICAO"

Além do Método de Consulta, outras informações são necessárias para envio da Requisição, que compõem o **Perfil** a ser consultado. Estas informações foram classificadas e organizadas, e mesmo as não obrigatórias, devem ser enviadas com valores padrão. São:

Dado | Tipo | Descritivo | Obrigatório
:------|:------|:------|:-----:
UF | Lista de Dados | Lista de UFs para Consulta de Regras. Por padrão enviar a UF da Empresa Filial | **Sim**
CFOP | Código da Operação | Código da Operação a ser Realizada. Deve ser enviada uma operação coerente com os dados desejados, por exemplo, uma Operação de Venda deve conter um CFOP que indique operação de Venda, mesmo que este não seja o correto. Para Operações de Venda enviar por padrão "5102" | **Sim**
Característica Tributária | Lista de Códigos Inteiros | Indica o Tipo de Destinatário da Operação | **Sim**
Finalidade | Código | Indica a Destinação do Produto para a Operação informada. É importante para especificar a operação. | Não
Simples Nacional | Caractere | Indica se o Destinatário da Operação é Simples Nacional ou Não. Preenchido com 'S' ou 'N'. Se CRT da Empresa é igual a 1 ou 2, enviar 'S', senão enviar 'N'. | Não
Origem | Código | Indica a Origem da Mercadoria. Se Tipo de Consulta igual a Método 1, e Código de Barras não iniciar em 789 ou 790, enviar Código 8. | **Sim**
Substituição Tributária | Caractere | Indica se o destinatário é Substituto Tributário. Valor padrão igual a 'N' | Não

   
## Fluxograma da Consulta Tributação
- ![Fluxo de Consulta - Novo Produto](./Flow-Consulta-Produto.jpeg)

[Voltar ao Sumário](#sumario)

## Consulta Tributos - Gerenciador de Tributação
### Métodos Básicos
### Métodos Avançados

## Relatórios Gerenciais - Logs
