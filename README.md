<div align="center">

# RotaCloud
### Arquitetura conceitual de uma rede de postos de combustíveis na AWS

**Da chegada do cliente ao fechamento do turno: uma visão integrada da operação física e dos recursos em nuvem.**

![Status](https://img.shields.io/badge/status-estudo%20conceitual-blue)
![Formação](https://img.shields.io/badge/DIO-AWS%20Cloud%20Foundations-orange)
![Serviços](https://img.shields.io/badge/AWS-EC2%20%7C%20EBS%20%7C%20S3%20%7C%20Lambda-232F3E)

</div>

---

## Visão geral

O **RotaCloud** é um exercício de arquitetura conceitual desenvolvido durante a formação **AWS Cloud Foundations**, com foco nos fundamentos do Amazon EC2, armazenamento em nuvem e gerenciamento de recursos.

O cenário escolhido é uma rede de postos de combustíveis com operação 24 horas, atendimento a veículos leves e pesados, restaurante, conveniência e equipes organizadas em três turnos.

A proposta é relacionar esse contexto de negócio aos serviços estudados, representando como instâncias EC2 podem executar aplicações, como volumes EBS podem fornecer armazenamento em blocos, como o S3 pode armazenar documentos e relatórios e como o Lambda pode apoiar rotinas automatizadas.

> **Natureza do projeto:** estudo educacional e arquitetura conceitual. Nenhum recurso foi implantado em uma conta AWS. As integrações com bombas, sensores, sistemas de pagamento e aplicações do posto são ilustrativas; não foram desenvolvidas nem testadas.

## O cenário de negócio

O complexo rodoviário considerado no estudo reúne diferentes jornadas e rotinas operacionais:

| Área | Atividades representadas |
|---|---|
| Chegada e direcionamento | Identificação do tipo de veículo e encaminhamento para a pista adequada |
| Abastecimento | Atendimento a carros, motos, caminhões e ônibus, com diferentes combustíveis |
| Caixa e pagamento | Registro do abastecimento, meios de pagamento e emissão de NFC-e |
| Restaurante | Buffet por quilo, balança, serviço à la carte, cozinha, estoque e comanda |
| Conveniência e serviços | Loja, troca de óleo, calibragem e banho para caminhoneiros |
| Equipe e turnos | Operação 24 horas, registro de ponto e fechamento de caixa |

O cenário foi escolhido para dar contexto às decisões de arquitetura. Os setores físicos representam o domínio do negócio; não são, por si só, componentes ou serviços da AWS.

## Diagrama da arquitetura

O diagrama reúne duas perspectivas complementares:

1. **Operação física:** jornada do cliente e rotina da equipe no posto.
2. **Infraestrutura em nuvem:** recursos AWS propostos para apoiar aplicações, armazenamento e rotinas administrativas.

<div align="center">

![Diagrama da arquitetura RotaCloud](images/rotacloud-arquitetura.png)

*Figura 1 — Visão conceitual da operação do posto e dos recursos AWS.*

</div>

**Arquivo-fonte editável:** [Abrir o diagrama no Draw.io](diagrams/rotacloud-arquitetura-detalhada.drawio).

## Arquitetura proposta

### 1. Amazon EC2 — capacidade computacional

O **Amazon Elastic Compute Cloud (Amazon EC2)** fornece capacidade computacional por meio de instâncias, que funcionam como máquinas virtuais na AWS.

Na proposta, foram representadas duas instâncias com responsabilidades distintas:

- **EC2 — Operação:** aplicações relacionadas ao atendimento, vendas, abastecimento e conveniência.
- **EC2 — Equipe:** aplicações de apoio aos funcionários, registro de ponto, turnos e relatórios.

A separação é uma decisão de modelagem utilizada para ilustrar diferentes cargas de trabalho. Não significa que essa seja a única arquitetura possível ou que a divisão tenha sido validada por dimensionamento.

A escolha de uma instância deve considerar as necessidades da aplicação, como CPU, memória, armazenamento, rede e sistema operacional. A **Amazon Machine Image (AMI)** fornece a imagem de referência utilizada para iniciar uma instância.

Como o EC2 é um serviço de infraestrutura como serviço (IaaS), o projeto também considera a responsabilidade sobre os aplicativos, dados e conexões configurados no ambiente.

### 2. Amazon EBS — armazenamento em blocos associado à EC2

O **Amazon Elastic Block Store (Amazon EBS)** fornece volumes de armazenamento em blocos que podem ser anexados às instâncias EC2.

No RotaCloud, os volumes EBS representam o armazenamento associado às aplicações de operação e de equipe, para dados e arquivos utilizados pelas instâncias.

Uma forma de compreender esse recurso é imaginá-lo como um disco conectado a uma máquina virtual. O volume pode ser utilizado pelo sistema operacional e pelas aplicações como armazenamento em blocos.

**Distinção importante:** EBS não é apresentado como banco de dados nem como armazenamento de objetos. Ele fornece volumes em blocos para uso pela EC2. A modelagem de um banco de dados transacional não faz parte do escopo deste primeiro exercício.

Também é importante distinguir a instância do volume: interromper uma instância EC2 não significa excluir automaticamente o volume EBS associado. Um volume que permanece provisionado pode continuar gerando custos.

### 3. Amazon S3 — armazenamento de objetos

O **Amazon Simple Storage Service (Amazon S3)** é o serviço de armazenamento de objetos utilizado na proposta para guardar arquivos gerados pelas rotinas do posto.

O desenho separa os usos por finalidade:

- **Notas fiscais:** arquivos relacionados à documentação fiscal.
- **Relatórios:** relatórios operacionais e de fechamento de turno.
- **Backups:** área representada para armazenar cópias de arquivos previstas na estratégia conceitual.

A separação por finalidade facilita a organização e a compreensão do cenário. Em uma implementação real, seria necessário definir buckets, prefixos, permissões, retenção, criptografia e requisitos legais.

O S3 oferece diferentes classes de armazenamento e regras de ciclo de vida (*Lifecycle*). Essas regras podem automatizar a transição de objetos entre classes, de acordo com critérios de acesso e retenção, contribuindo para a otimização de custos.

### 4. AWS Lambda — automação de rotinas

O **AWS Lambda** permite executar código em resposta a eventos ou invocações, sem que seja necessário administrar diretamente servidores para a execução da função.

No diagrama, o Lambda aparece associado a possibilidades de automação:

- Organização de documentos fiscais.
- Apoio ao fechamento de turno.
- Geração de alerta de estoque baixo.
- Coordenação de uma rotina relacionada a snapshots.

Esses itens são **casos de uso propostos**, não funções implementadas ou testadas. Para cada automação, ainda seria necessário definir a origem dos dados, o evento de acionamento, as permissões e a integração com os demais recursos.

**Sobre snapshots:** a representação da rotina noturna deve ser entendida como uma ideia de automação ou coordenação do processo. Snapshots de volumes EBS são recursos gerenciados pela AWS; não devem ser interpretados como arquivos que uma função Lambda simplesmente copia para um bucket S3 comum.

O detalhamento de gatilhos, código e mecanismos de integração fica fora do escopo introdutório deste projeto.

## Fluxo operacional ilustrativo

O fluxo abaixo conecta a jornada do negócio às responsabilidades atribuídas aos recursos:

1. O cliente chega e é direcionado conforme o tipo de veículo.
2. O abastecimento é realizado na pista adequada.
3. A operação é registrada no caixa; o cenário prevê diferentes meios de pagamento e emissão de NFC-e.
4. O cliente pode utilizar o restaurante, a conveniência ou outros serviços.
5. Os funcionários registram seus turnos e realizam as rotinas de fechamento.
6. As aplicações representadas nas instâncias EC2 apoiam as atividades operacionais.
7. Os volumes EBS fornecem armazenamento em blocos às instâncias.
8. Documentos e relatórios são organizados conceitualmente no Amazon S3.
9. As rotinas atribuídas ao Lambda representam possibilidades de automação e geração de alertas.

Esse fluxo descreve o funcionamento pretendido do cenário. Não especifica protocolos nem comprova integração com bombas, terminais de pagamento ou equipamentos físicos.

## Gerenciamento e otimização de recursos

O conteúdo da formação aborda otimização principalmente como uma forma de utilizar os recursos de maneira eficiente e evitar custos desnecessários. No cenário RotaCloud, os conceitos podem ser relacionados da seguinte forma:

- **Desligamento de instâncias não utilizadas:** em ambientes de desenvolvimento, testes ou treinamento, instâncias sem uso fora do horário de trabalho podem ser interrompidas, quando isso for compatível com a finalidade do ambiente.
- **Remoção de recursos ociosos:** recursos que não são mais necessários devem ser identificados para evitar gastos desnecessários.
- **Escalabilidade vertical:** aumentar ou reduzir a capacidade de um recurso, como CPU, memória, armazenamento ou rede de uma instância.
- **Escalabilidade horizontal:** aumentar a quantidade de recursos ou instâncias para atender à demanda.
- **Sob demanda (On-Demand):** opção que pode ser adequada a cargas variáveis e usos de curto prazo que não podem ser interrompidos.
- **Instâncias reservadas:** modalidade de compromisso de uso que pode oferecer redução de custo em cenários compatíveis com o período contratado.
- **Spot:** capacidade com desconto, sujeita a interrupções; deve ser considerada apenas para cargas que tolerem esse comportamento.

A escolha de modalidade de compra e estratégia de escala dependeria de métricas reais de utilização, previsibilidade da demanda, criticidade e orçamento. Nenhuma modalidade foi contratada ou simulada neste projeto.

## Snapshots e continuidade dos dados

Snapshots de volumes EBS são considerados como conceito relacionado à proteção e recuperação de dados.

A rotina noturna indicada no diagrama é uma representação conceitual. Uma implementação real precisaria definir frequência de criação, retenção, permissões, recuperação e responsabilidades operacionais. A automação poderia apoiar o agendamento ou a coordenação dessas tarefas, mas o snapshot continua sendo um recurso gerenciado pela AWS.

O material de estudo também destaca a diferença entre interromper e encerrar uma instância. Dados armazenados em volumes EBS podem permanecer após a interrupção da instância; já dados em armazenamento temporário da instância podem ter outro comportamento. Volumes EBS mantidos provisionados podem continuar gerando custos.

## Segurança — premissas iniciais

O diagrama inclui uma representação introdutória de **Região AWS, VPC, sub-rede e Security Group**, para contextualizar a organização do ambiente de rede das instâncias.

As premissas registradas são:

- O Security Group deve liberar somente o tráfego necessário às aplicações.
- O acesso administrativo deve ser restrito à origem autorizada, evitando exposição indiscriminada.
- A instância deve utilizar uma função IAM associada por um perfil de instância para acessar recursos AWS autorizados, evitando armazenar credenciais diretamente no servidor.
- O acesso ao S3 deve seguir o princípio do menor privilégio.

Esses pontos são diretrizes conceituais. A configuração efetiva de rede, identidade, criptografia, auditoria e proteção de dados não foi realizada.

## Expansão para múltiplas unidades

O desenho ilustra a possibilidade de expansão para outros postos, identificados como **Posto 02 ... N**.

A representação inicial sugere a repetição de recursos de computação e armazenamento por unidade e a organização dos arquivos no S3. A intenção é demonstrar como o cenário poderia ser replicado em outras unidades da rede.

Essa representação não constitui um projeto validado de arquitetura multiunidade. Em uma solução real, seria necessário estudar isolamento entre unidades, governança central, conectividade, administração, custos e disponibilidade antes de decidir entre ambientes separados ou compartilhados.

## Melhorias realizadas no projeto

Durante a elaboração do exercício, foram realizadas melhorias para tornar a proposta mais clara e coerente com os conceitos estudados:

- Escolha de um cenário de negócio com diferentes jornadas operacionais, em vez de um exemplo genérico.
- Separação visual entre a operação física do posto e os recursos de infraestrutura em nuvem.
- Organização das responsabilidades atribuídas a EC2, EBS, S3 e Lambda.
- Diferenciação entre armazenamento em blocos (EBS) e armazenamento de objetos (S3).
- Inclusão de conceitos de AMI, Security Group, VPC, sub-rede e perfil IAM no contexto da arquitetura.
- Relacionamento dos conceitos de otimização, escalabilidade, interrupção de instâncias e snapshots ao cenário proposto.
- Registro explícito das premissas, limitações e integrações que ainda não foram implementadas.

## O que aprendi

Este exercício ajudou a consolidar e relacionar os seguintes conceitos:

- O papel do Amazon EC2 como serviço de computação e execução de instâncias.
- A importância de considerar CPU, memória, armazenamento, rede e sistema operacional ao escolher uma instância.
- O uso de AMIs como imagens de referência para iniciar instâncias.
- A função dos Security Groups no controle do tráfego de rede.
- A relação entre instâncias EC2 e volumes EBS.
- A diferença entre armazenamento em blocos (EBS) e armazenamento de objetos (S3).
- O uso de classes de armazenamento e regras de ciclo de vida no S3.
- O conceito introdutório de automação orientada a eventos com Lambda.
- Noções de snapshots, persistência de dados e otimização de recursos.
- A importância de documentar decisões, premissas e limitações em um diagrama de arquitetura.

## Limites do projeto e oportunidades de evolução

Para manter o trabalho coerente com o estágio atual da formação, alguns temas foram deliberadamente mantidos fora do escopo:

- Implantação de recursos em uma conta AWS.
- Dimensionamento de tipos e quantidades de instâncias.
- Estimativa monetária baseada em região, consumo e preços atuais.
- Implementação das funções Lambda e configuração de seus gatilhos.
- Integração real com bombas, sensores de tanques, PDV, adquirentes ou sistemas fiscais.
- Definição de banco de dados transacional.
- Projeto detalhado de alta disponibilidade, recuperação de desastres e conectividade entre unidades.
- Execução de testes na AWS.

Esses pontos representam oportunidades de evolução conforme novos conteúdos forem estudados na formação.

## Estrutura do repositório

```text
roudcloud-arquitetura/
├── README.md
├── diagrams/
│   └── rotacloud-arquitetura-detalhada.drawio
└── images/
    └── rotacloud-arquitetura.png
```

- **README.md:** documentação do cenário, dos conceitos e das decisões da proposta.
- **diagrams/:** arquivo-fonte editável do diagrama.
- **images/:** imagem exportada para visualização direta no GitHub.

## Referências de estudo

Materiais utilizados como base para este exercício:

- Formação **AWS Cloud Foundations — DIO**, módulo de computação em nuvem com EC2.
- Slides da formação sobre Amazon EC2, Amazon EBS, Amazon S3 e otimização de recursos.
- Material de apoio do desafio: **Gerenciar instâncias do Amazon EC2**.
- [Documentação AWS — Gerenciar instâncias do Amazon EC2 (AWS Toolkit for Visual Studio)](https://docs.aws.amazon.com/pt_br/toolkit-for-visual-studio/latest/user-guide/tkv-ec2-ami.html)
- [Amazon EC2 — página oficial](https://aws.amazon.com/pt/ec2/)
- [Amazon EBS — página oficial](https://aws.amazon.com/pt/ebs/)
- [Amazon S3 — página oficial](https://aws.amazon.com/pt/s3/)
- [AWS Lambda — página oficial](https://aws.amazon.com/pt/lambda/)

> A documentação do AWS Toolkit for Visual Studio é uma referência específica para o fluxo de gerenciamento pelo Visual Studio. Os nomes de telas e procedimentos desse material não devem ser interpretados como um tutorial universal da interface atual do Console AWS.

## Sobre o projeto

Projeto educacional desenvolvido para consolidar os primeiros conceitos de arquitetura e gerenciamento de recursos AWS durante a formação AWS Cloud Foundations.

**Autoria:** [valquiria87](https://github.com/valquiria87)  
**Repositório:** [roudcloud-arquitetura](https://github.com/valquiria87/roudcloud-arquitetura)

---

<div align="center">

**RotaCloud — uma arquitetura em evolução, acompanhando a jornada de aprendizado em AWS.**

</div>
