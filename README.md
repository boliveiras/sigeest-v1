# SIGEEST — Sistema de Gerenciamento de Estruturas

> Trabalho de Conclusão de Curso — Tecnólogo em Análise e Desenvolvimento de Sistemas
> Apresentado à banca em **28/06/2018** · Conceito **9 (nove)** · Publicado no GitHub em **2019**

---

## Apresentação

O **SIGEEST** (Sistema de Gerenciamento de Estruturas) nasceu como o meu Trabalho de Conclusão de Curso de Análise e Desenvolvimento de Sistemas, em 2018. Na época, eu estava dando meus primeiros passos sérios com o ecossistema Java e o Spring, e este projeto foi onde muitas ideias de "sala de aula" finalmente viraram código que rodava de verdade.

Ele é um sistema web para gerenciar a **infraestrutura física de um provedor de internet / rede de telecomunicações**: as torres e pontos de transmissão espalhados pela cidade, os equipamentos de rede instalados neles, as baterias que mantêm tudo funcionando quando falta energia, os fabricantes desses itens e o estoque de onde cada peça sai e para onde volta.

Não é um produto comercial e nunca pretendeu ser. É o registro honesto de um estudante aprendendo a costurar, de ponta a ponta, uma aplicação real: modelagem de domínio, persistência, segurança, telas, validação e regras de negócio. Se você chegou aqui anos depois, seja muito bem-vindo(a) — este README foi escrito para que você entenda tanto **o que o sistema faz** quanto **a história por trás dele**.

---

## Sobre o Projeto

O SIGEEST organiza o ciclo de vida dos ativos de uma rede de telecomunicações. A análise do código revela um domínio bem específico e coerente:

- Uma operadora possui **estruturas** (torres/pontos) classificadas como de **Transmissão**, **Recepção** ou **Repetição**, cada uma com endereço, coordenadas geográficas (latitude/longitude), elevação, telefone de contato, proprietário e valor de aluguel.
- Nessas estruturas são instalados **equipamentos** de rede — CPE, switch, roteador, conversor, modem — cada um identificado por um **endereço MAC** único.
- O fornecimento de energia de backup é feito por **baterias** (VRLA, no-break, estacionária), identificadas por **número de série**.
- Equipamentos e baterias são comprados de **fabricantes** e ficam guardados em **estoques** (almoxarifados, que também têm localização geográfica) enquanto não estão alocados em campo.
- Tudo é operado por **usuários** organizados em **grupos**, e cada grupo carrega um conjunto de **permissões** que define o que aquele usuário pode fazer.

O coração da aplicação é o **movimento dos ativos**: um equipamento sai do estoque, é alocado em uma estrutura, eventualmente vai para manutenção e depois retorna. O SIGEEST acompanha esse vai-e-vem através de um conjunto de estados bem definidos.

---

## Contexto Acadêmico

Este sistema foi desenvolvido e defendido como **Trabalho de Conclusão de Curso (TCC)** do curso Tecnólogo em **Análise e Desenvolvimento de Sistemas**. Foi apresentado à banca em **28 de junho de 2018**, recebeu **conceito 9** e me qualificou como graduado.

Mais do que uma nota, ele representa o momento em que vários conteúdos estudados de forma isolada — orientação a objetos, banco de dados, modelagem, web — precisaram conversar entre si dentro de uma única aplicação que tinha que compilar, subir e funcionar na frente de uma banca avaliadora. Foi o projeto em que aprendi, na prática, o que significa **entregar um software inteiro**, e não apenas um exercício.

Em 2019 ele foi publicado no GitHub, e desde então cumpre um segundo papel: o de marco da minha trajetória.

---

## O Problema que o Projeto Resolve

Provedores de internet de pequeno e médio porte lidam com um inventário que vive em movimento. Equipamentos e baterias têm custo, número de série, fabricante e um histórico de onde estiveram. Eles saem do estoque, são instalados em uma torre, quebram, vão para manutenção, voltam. Sem um controle central, esse patrimônio se perde em planilhas, na memória das pessoas ou em anotações de campo.

O SIGEEST se propõe a responder, a qualquer momento, perguntas como:

- **Onde** está fisicamente cada equipamento e cada bateria?
- Esse ativo está **em uso**, **em estoque** ou **em manutenção**?
- Quais equipamentos estão instalados **nesta estrutura** específica?
- De qual **fabricante** veio determinado item e quanto ele custou?
- **Quem** no sistema pode cadastrar, editar, excluir ou apenas consultar essas informações?

É um sistema de **inventário e controle de ativos de rede** com consciência de localização geográfica — daí a integração com mapas para visualizar onde cada estrutura está.

---

## Funcionalidades

As funcionalidades estão organizadas por entidade, e para cada uma o sistema oferece um conjunto consistente de operações. Essa simetria é uma das características mais marcantes do projeto.

### Módulo de Estruturas (torres / pontos de rede)
- Cadastro, edição, exclusão, exibição e pesquisa de estruturas.
- Classificação por tipo: Transmissão, Recepção, Repetição.
- Registro de coordenadas geográficas, com **visualização em mapa** (Google Maps).
- **Alocação e remoção** de equipamentos e baterias na estrutura.

### Módulo de Equipamentos
- Cadastro, edição, exclusão, exibição e pesquisa de equipamentos.
- Validação de **endereço MAC** por expressão regular.
- Tipos: CPE, Switch, Roteador, Conversor, Modem.
- Envio para manutenção e recebimento de volta da manutenção.

### Módulo de Baterias
- Cadastro, edição, exclusão, exibição e pesquisa de baterias.
- Controle por **número de série**.
- Tipos: VRLA, No-Break, Estacionária.
- Envio para manutenção e recebimento de volta da manutenção.

### Módulo de Fabricantes
- Cadastro, edição, exclusão, exibição e pesquisa de fabricantes.
- Vínculo com os equipamentos e baterias fornecidos.

### Módulo de Estoques
- Cadastro, edição, exclusão, exibição e pesquisa de estoques (almoxarifados).
- Localização geográfica do estoque.
- Origem e destino dos ativos durante a alocação/remoção.

### Módulo de Usuários e Controle de Acesso
- Cadastro, edição, exclusão, exibição e pesquisa de usuários.
- Vínculo de cada usuário a um **setor** (NOC, Atendimento, Infraestrutura, Projetos, Gerência, Direção).
- Autenticação por login/senha e **autorização baseada em papéis** (grupos e permissões).
- Ativação/desativação de usuários (usuário inativo não consegue autenticar).

### Ciclo de vida dos ativos
Equipamentos e baterias compartilham um conjunto de estados e transições, implementados diretamente no modelo de domínio:

| Estado | Significado | Como se chega nele |
|---|---|---|
| `INDEFINIDO` | Recém-cadastrado, ainda não movimentado | Estado inicial |
| `DESALOCADO` | Em estoque, disponível | Ao ser estocado ou ao voltar de campo/manutenção |
| `ALOCADO` | Em uso em uma estrutura | Ao ser alocado em uma estrutura |
| `MANUTENCAO` | Em reparo, inativo | Ao ser enviado para manutenção |

---

## Arquitetura

O SIGEEST segue uma arquitetura **MVC em camadas**, típica das aplicações Spring da época, com renderização de páginas no servidor.

```
Navegador
   │  (HTTP)
   ▼
Controllers (control)  ──►  Services (service)  ──►  Repositories (repository)  ──►  Banco de dados (MySQL)
   │                                                                                      │
   ▼                                                                                      │
Templates Thymeleaf  ◄───────────────── Modelos de domínio (model) ◄──────────────────────┘
```

### Organização em camadas

- **`control`** — Controllers Spring MVC. Recebem as requisições, validam os dados, chamam os serviços e devolvem o nome da view (template Thymeleaf) ou um redirecionamento.
- **`service`** — Camada de serviço. Concentra as regras de negócio e a orquestração entre múltiplos repositórios (por exemplo, alocar um equipamento mexe em três entidades ao mesmo tempo: estrutura, estoque e equipamento).
- **`repository`** — Interfaces Spring Data JPA. Cada uma estende `JpaRepository`, ganhando CRUD pronto e métodos de consulta derivados do nome (como `findByLogin`).
- **`model`** — Entidades JPA que representam o domínio, com herança via `@MappedSuperclass`.
- **`enums`** — Tipos enumerados que dão semântica ao domínio (tipos, status, setores).
- **`security`** — Integração com Spring Security (carregamento de usuário e autoridades).
- **`config`** — Configuração de segurança da aplicação.
- **`converters`** — Conversores JPA para tipos de data do Java 8 (`LocalDate`/`LocalDateTime`), que ainda não eram suportados nativamente nessa versão.

### Um detalhe de estilo: um controller por ação

Uma escolha que chama atenção — e que conta a história de um desenvolvedor em formação — é que **cada ação tem sua própria classe de controller**: `CadastrarEstrutura`, `EditarEstrutura`, `ExcluirEstrutura`, `ExibirEstrutura`, `PesquisarEstrutura`, e assim por diante para cada entidade. Em vez de um único `EstruturaController` com vários métodos, o projeto decompõe o sistema em **uma classe por caso de uso**.

Hoje o mais comum seria agrupar as ações de uma entidade em um só controller. Mas essa decomposição tão granular revela algo positivo: o autor estava pensando o sistema em termos de **funcionalidades/casos de uso** — exatamente como se modela um TCC a partir de um documento de requisitos. Cada "história" virou um arquivo. É uma decisão datada, sim, mas honesta e didática.

### Herança no domínio

O modelo usa herança para evitar repetição:

- **`Componente`** (`@MappedSuperclass`) é a base de **`Equipamento`** e **`Bateria`** — concentra nome, corrente, voltagem, preço, status, data de compra e o ciclo de vida (`estocar`, `desestocar`, `emManutencao`, `semManutencao`).
- **`Localidade`** (`@MappedSuperclass`) é a base de **`Estrutura`** e **`Estoque`** — concentra endereço, latitude, longitude, elevação e telefone, porque ambos existem em um ponto físico do mapa.

### Fluxo de execução (exemplo real)

Quando um operador aloca um equipamento em uma estrutura:

1. O `AdicionarEquipamento` (controller) recebe os códigos da estrutura, do equipamento e do estoque pela URL.
2. Chama `EstruturaService.adicionarEquipamento(...)`.
3. O serviço orquestra a regra de negócio: muda o status do equipamento (`desestocar`), vincula-o à estrutura, remove-o do estoque e **salva as três entidades**.
4. O usuário é redirecionado para a tela de exibição da estrutura, agora com o equipamento listado.

---

## Stack Tecnológica

| Tecnologia | Versão | Papel no projeto |
|---|---|---|
| **Java** | 8 (1.8) | Linguagem base. O uso de `LocalDate`/`LocalDateTime` mostra a adoção da então nova API de datas do Java 8. |
| **Spring Boot** | 1.5.9.RELEASE | Espinha dorsal da aplicação: autoconfiguração, servidor embarcado e empacotamento. Reduziu drasticamente o "boilerplate" de configurar o Spring na mão. |
| **Spring MVC** | (via Boot) | Controllers, roteamento de URLs e o padrão Model-View-Controller. |
| **Spring Data JPA** | (via Boot) | Persistência sem escrever SQL na maioria dos casos — repositórios como interfaces. |
| **Hibernate** | (via JPA) | Implementação de ORM por trás do JPA. Mapeia objetos Java para tabelas MySQL. |
| **Spring Security** | (via Boot) | Autenticação por formulário e autorização por papéis (roles). |
| **Thymeleaf** | (via Boot) | Motor de templates server-side. As telas são HTML renderizado no servidor, com `thymeleaf-extras-springsecurity4` para controlar o que cada papel vê e `thymeleaf-extras-java8time` para formatar datas. |
| **MySQL** | (connector runtime) | Banco de dados relacional onde tudo é persistido. |
| **Bootstrap** | 3.x | Layout e componentes visuais responsivos. |
| **jQuery** | 3.2.1 / 2.1.1 | Interatividade no navegador e base para os plugins. |
| **Plugins jQuery** | — | DataTables (tabelas), Datepicker (datas em pt-BR), Masked Input / maskMoney (máscaras de MAC, telefone e dinheiro), autocomplete, validator e Chart.js. |
| **Google Maps API** | — | Exibição da localização geográfica das estruturas em mapa. |
| **Maven** | (wrapper incluído) | Build, gestão de dependências e empacotamento. O `mvnw`/`mvnw.cmd` permite buildar sem ter o Maven instalado. |
| **WebJars** | — | jQuery, Bootstrap e Font Awesome distribuídos como dependências Maven. |

O projeto pode ser executado de duas formas, graças ao `ServletInitializer`: como **JAR executável** (servidor Tomcat embarcado) ou como **WAR** implantado em um servidor de aplicação externo — no repositório há, inclusive, configuração para o *Pivotal tc Server*, usado no Spring Tool Suite da época.

---

## Estrutura do Projeto

```
sigeest-v1/
├── README.md
├── LICENSE
├── Servers/                         # Configuração do servidor (Pivotal tc Server / STS)
└── sigeest/                         # A aplicação Spring Boot
    ├── pom.xml                      # Dependências e build (Maven)
    ├── mvnw / mvnw.cmd              # Maven Wrapper
    └── src/
        ├── main/
        │   ├── java/br/com/sigeest/
        │   │   ├── SigeestApplication.java      # Ponto de entrada (main) + locale pt-BR
        │   │   ├── ServletInitializer.java      # Suporte a deploy como WAR
        │   │   ├── config/        # SecurityConfig (regras de acesso)
        │   │   ├── control/       # Controllers — uma classe por ação
        │   │   ├── service/       # Regras de negócio
        │   │   ├── repository/    # Interfaces Spring Data JPA
        │   │   ├── model/         # Entidades JPA (domínio)
        │   │   ├── enums/         # Tipos, status, setores
        │   │   ├── security/      # UserDetailsService e usuário do sistema
        │   │   └── converters/    # Conversores de data Java 8 <-> SQL
        │   └── resources/
        │       ├── application.properties       # Datasource e JPA
        │       ├── static/        # CSS, JS, imagens (Bootstrap, jQuery, mapa…)
        │       └── templates/     # Telas Thymeleaf (.html)
        └── test/
            └── java/br/com/sigeest/SigeestApplicationTests.java
```

> Observação histórica: o repositório também carrega uma pasta `.metadata/` e configurações de workspace do Eclipse/STS. São resquícios do ambiente de desenvolvimento da época — um retrato fiel de como o projeto foi versionado em 2019, direto da IDE.

---

## Modelo de Dados

### Principais entidades

- **Estrutura** — torre/ponto de rede. Herda de `Localidade`. Possui nome, proprietário, aluguel, tipo, e listas de equipamentos e baterias instalados.
- **Equipamento** — ativo de rede com MAC único. Herda de `Componente`. Pertence a um fabricante e está em uma estrutura **ou** em um estoque.
- **Bateria** — fonte de energia com serial único. Herda de `Componente`. Mesma relação com fabricante, estrutura e estoque.
- **Fabricante** — fornece equipamentos e baterias.
- **Estoque** — almoxarifado com localização. Herda de `Localidade`. Guarda equipamentos e baterias desalocados.
- **Usuario** — operador do sistema. Pertence a um grupo e a um setor.
- **Grupo** — agrupa usuários e carrega uma lista de permissões.
- **Permissao** — uma capacidade do sistema (PESQUISAR, CADASTRAR, EDITAR, EXCLUIR, ADICIONAR, REMOVER).

### Relacionamentos

```
Fabricante 1 ──── N Equipamento N ──── 1 Estrutura
     │                                      │
     └──────── N Bateria ───────────────────┘
                   │
Estoque 1 ──── N Equipamento / Bateria   (quando desalocados)

Usuario N ──── 1 Grupo N ──── N Permissao   (grupos_permissoes)
```

- Um **Fabricante** tem muitos equipamentos e baterias (`@OneToMany`).
- Uma **Estrutura** e um **Estoque** têm, cada um, muitos equipamentos e baterias (`@OneToMany`) — um ativo está sempre de um lado ou do outro.
- Um **Equipamento**/**Bateria** aponta para seu fabricante, sua estrutura e seu estoque (`@ManyToOne`).
- A relação **Grupo ↔ Permissão** é muitos-para-muitos (`@ManyToMany`), materializada na tabela de junção `grupos_permissoes`.

### Regras de negócio identificadas

- **Unicidade**: MAC de equipamento, serial de bateria, nome de estrutura/fabricante/grupo/permissão, descrição de estoque e login de usuário são únicos no banco (constraints declaradas nas entidades).
- **Validações de domínio** (Bean Validation): MAC precisa seguir o formato `xx:xx:xx:xx:xx:xx`; voltagem entre 5 V e 227 V; aluguel entre 0,01 e 9.999.999,99; data de compra não pode ser futura; senha entre 8 e 15 caracteres; e-mail em formato válido.
- **Ciclo de vida**: as transições de status são guardadas no próprio modelo (`Componente`), impedindo, por exemplo, mandar para manutenção um item que não esteja desalocado.
- **Acesso**: um usuário desativado não autentica; cada operação exige o papel correspondente.

> O schema do banco é gerado automaticamente pelo Hibernate (`spring.jpa.hibernate.ddl-auto=update`) a partir das anotações nas entidades — não há scripts SQL versionados no projeto.

---

## Fluxo de Funcionamento

1. **Autenticação** — o usuário acessa `/login`. O `ImplUserDetailsService` busca o usuário pelo login, verifica se está ativo e monta suas autoridades a partir dos grupos e permissões.
2. **Autorização** — a cada requisição, o `SecurityConfig` confere se o papel do usuário permite a URL acessada. O menu (Thymeleaf + Spring Security) só mostra as opções que aquele usuário tem direito de ver.
3. **Operação** — o usuário navega pelos módulos (Pesquisar, Cadastrar, Editar, Excluir). Cada ação cai em um controller específico.
4. **Validação** — os dados do formulário passam por Bean Validation. Se houver erro, a tela é reexibida com as mensagens.
5. **Negócio e persistência** — o controller chama o serviço, que aplica as regras e usa os repositórios para gravar no MySQL via Hibernate.
6. **Resposta** — o sistema redireciona ou reexibe uma view Thymeleaf, frequentemente com uma mensagem de sucesso (padrão *Post/Redirect/Get*).

---

## Como Executar

> ⚠️ Este é um projeto de 2018/2019. Rodá-lo hoje é mais um exercício de "viagem no tempo" do que uma instalação de produção. As instruções abaixo refletem o que o código realmente espera.

### Pré-requisitos

- **JDK 8** (o projeto declara `java.version` 1.8).
- **MySQL** em execução local.
- **Maven** — opcional, já que o projeto inclui o Maven Wrapper (`mvnw`).

### Banco de dados

A aplicação espera um banco chamado **`SIGEESTBD`** em `localhost`. Por padrão, o Hibernate cria/atualiza as tabelas sozinho (`ddl-auto=update`), então basta o banco existir:

```sql
CREATE DATABASE SIGEESTBD;
```

As credenciais ficam em [`sigeest/src/main/resources/application.properties`](sigeest/src/main/resources/application.properties). No commit original elas apontam para `root` com uma senha fixa — **ajuste para o seu ambiente** antes de subir.

### Build e execução

A partir da pasta `sigeest/`:

```bash
# Build
./mvnw clean package        # Linux/Mac
mvnw.cmd clean package      # Windows

# Executar
./mvnw spring-boot:run      # Linux/Mac
mvnw.cmd spring-boot:run    # Windows
```

A aplicação sobe em `http://localhost:8080` (porta padrão do Spring Boot, já que nenhuma outra é configurada).

> Para conseguir entrar, é preciso ter ao menos um usuário, um grupo e suas permissões cadastrados no banco — o projeto não inclui um script de carga inicial (*seed*). Esse era um passo manual na época da apresentação.

---

## Aprendizados Obtidos

Olhando para este código anos depois, é possível ver um desenvolvedor aprendendo em tempo real — e isso é bonito.

Foi neste projeto que eu entendi, de verdade, como as peças do Spring se encaixam: o que é injeção de dependência quando você precisa que um serviço converse com três repositórios; por que separar controller, serviço e repositório torna o código mais fácil de raciocinar; como o Spring Security transforma "grupos e permissões" do banco em regras de acesso reais nas telas e nas URLs.

Aprendi a **modelar um domínio** a partir de um problema do mundo real — perceber que "estrutura" e "estoque" compartilham a ideia de *localização*, que "equipamento" e "bateria" compartilham a ideia de *componente*, e transformar isso em herança. Aprendi que regras de negócio têm lugar (a camada de serviço e o próprio modelo), e que validação não é firula: é o que protege a integridade dos dados.

E aprendi coisas que só a prática ensina: que um ciclo de vida de estados precisa ser pensado com cuidado, que datas em Java são um capítulo à parte, que fazer o deploy e ver tudo funcionando na frente de uma banca é uma sensação que nenhum tutorial entrega.

Sim, há decisões que eu faria diferente hoje — e a seção de [Sugestões de Evolução](#sugestões-de-evolução) é honesta sobre isso. Mas é exatamente essa distância entre o "eu de 2018" e o "eu de hoje" que mede o quanto se evoluiu. Projetos acadêmicos como o SIGEEST são a fundação invisível sobre a qual a carreira é construída: não pelo código que sobrou, mas pelo desenvolvedor que se formou escrevendo-o.

---

## Valor Histórico

Este repositório permanece publicado de propósito.

Ele não está aqui porque é o melhor código que já escrevi — está aqui porque é um dos mais **importantes**. É o ponto de partida documentado de uma jornada. Cada escolha datada, cada padrão que hoje eu refinaria, cada linha de validação caprichada conta um pedaço de como eu pensava quando estava começando.

Manter o SIGEEST no ar é uma forma de respeitar a própria trajetória — de poder voltar, anos depois, e dizer "foi aqui que começou". Para quem está estudando agora, ele pode servir de exemplo de um TCC Spring completo e funcional. Para mim, é um marco: a prova de que toda evolução tem um começo, e que vale a pena preservar esse começo.

---

## Licença

Este projeto é distribuído sob a **GNU General Public License v3.0 (GPL-3.0)**. O texto completo está em [LICENSE](LICENSE).

### Por que GPL-3.0?

A escolha reflete o espírito com que o SIGEEST foi criado e mantido: **conhecimento que se compartilha e melhorias que voltam para a comunidade**. A GPL-3.0 é uma licença *copyleft* forte — ela garante que o código permaneça livre não só agora, mas em qualquer trabalho derivado dele.

Entre as opções consideradas:

- **MIT / BSD 3-Clause / Apache 2.0** são licenças permissivas: qualquer um pode pegar o código, fechá-lo e redistribuir sem devolver nada. Ótimas para máxima adoção, mas não combinam com a intenção de manter as melhorias abertas.
- **GPL-3.0** é *copyleft*: quem distribui uma versão modificada **precisa** distribuir também o código-fonte, sob a mesma licença. As melhorias permanecem públicas.

Para um projeto acadêmico e de portfólio, cujo maior valor é **servir de estudo e ser melhorado abertamente**, a GPL-3.0 é a que melhor preserva esse propósito.

> **Nota:** Por se tratar de uma aplicação web, existe ainda a **AGPL-3.0**, irmã da GPL que estende o copyleft também a quem oferece o software como serviço pela rede (sem distribuir o binário). Caso, no futuro, você queira esse alcance — e manter consistência com outros projetos seus que já usam AGPL —, a migração é natural. Para um TCC publicado como referência de estudo, a GPL-3.0 já cumpre muito bem o papel.

### Direitos concedidos pela GPL-3.0

- ✅ **Usar** o software para qualquer finalidade, inclusive comercial.
- ✅ **Estudar** o código-fonte e adaptá-lo às suas necessidades.
- ✅ **Modificar** o software livremente.
- ✅ **Distribuir** cópias, originais ou modificadas.

### Obrigações impostas pela GPL-3.0

- 📋 **Disponibilizar o código-fonte** ao distribuir o software (original ou modificado).
- 📋 **Manter a mesma licença** (GPL-3.0) nos trabalhos derivados — é o efeito *copyleft*.
- 📋 **Preservar os avisos** de copyright e licença.
- 📋 **Declarar as alterações** significativas feitas no código.
- 📋 **Sem garantia**: o software é fornecido "como está", sem qualquer garantia (cláusula padrão da licença).

---

## Considerações Finais

Tecnologias envelhecem, versões ficam para trás, e o jeito de escrever software muda a cada poucos anos. O SIGEEST é um retrato fiel de um momento — e está tudo bem que seja datado, porque era exatamente o que precisava ser quando foi criado.

O que não envelhece é o aprendizado. Cada projeto, por mais simples que pareça depois, deixou uma camada de entendimento que sustentou o próximo. Preservar este repositório é preservar essa história: a de alguém que continuou estudando, continuou evoluindo, e que olha para onde começou com gratidão, não com vergonha.

Se você chegou até aqui, obrigado por dedicar um tempo a conhecer não só um sistema, mas uma etapa de uma jornada. Que o seu próprio começo, seja ele qual for, também valha a pena ser guardado. 🚀

---

## Sugestões de Evolução

> Esta seção é um exercício de "e se o SIGEEST fosse retomado hoje?". Nada aqui é uma crítica ao projeto de 2018 — é uma forma de mapear, com carinho, o caminho entre o que ele foi e o que poderia se tornar com o ferramental atual.

### Arquitetura
- **Agrupar controllers por entidade**: consolidar `CadastrarEstrutura`, `EditarEstrutura`, etc. em um `EstruturaController` coeso reduziria a quantidade de classes e facilitaria a navegação.
- **DTOs e camada de apresentação**: hoje as entidades JPA são usadas diretamente nos formulários. Separar entrada/saída em DTOs evitaria expor o modelo de persistência na web e problemas como *over-posting*.
- **Tratamento transacional explícito**: anotar os métodos de serviço que tocam múltiplos repositórios com `@Transactional` garantiria atomicidade nas operações de alocação/remoção.
- **API REST opcional**: expor uma camada REST (além do MVC) abriria caminho para um front-end moderno (React/Angular/Vue) ou integrações.

### Segurança
- **Hash de senhas**: o ponto mais importante. As senhas são persistidas em texto puro; o padrão atual é usar `BCryptPasswordEncoder` (ou Argon2). Essencial antes de qualquer uso real.
- **Credenciais fora do código**: mover usuário/senha do banco do `application.properties` para variáveis de ambiente ou um cofre de segredos, e nunca versioná-las.
- **Proteção CSRF e HTTPS**: revisar a configuração do Spring Security para as práticas atuais e forçar TLS.
- **Política de senha e bloqueio**: limitar tentativas de login e exigir senhas mais fortes.

### Manutenção
- **Atualizar a stack**: Spring Boot 1.5 chegou ao fim de vida; uma migração para uma versão atual (e Java LTS — 17/21) traz correções de segurança e produtividade. O salto `javax` → `jakarta` faz parte dessa jornada.
- **Migrações de banco versionadas**: adotar Flyway ou Liquibase em vez de `ddl-auto=update` torna o schema reproduzível e auditável.
- **Remover artefatos de IDE**: tirar a pasta `.metadata/` e configurações de workspace do versionamento (mantendo-as como registro histórico, se desejado).

### Observabilidade
- **Logging estruturado**: padronizar logs (SLF4J + Logback) com níveis adequados, em vez de `show-sql` no console.
- **Spring Boot Actuator**: expor *health checks* e métricas.
- **Métricas e tracing**: Micrometer + Prometheus/Grafana dariam visibilidade sobre o uso e a saúde da aplicação.

### Testes
- **Cobertura real**: hoje há apenas o teste de contexto gerado pelo Spring Initializr. Valeria cobrir as regras de negócio dos serviços (alocação, ciclo de vida) com testes unitários (JUnit 5 + Mockito).
- **Testes de integração**: `@DataJpaTest` para repositórios e `@WebMvcTest` para controllers, com banco em memória ou *Testcontainers*.

### Performance
- **Paginação nas listagens**: `findAll()` carrega tudo de uma vez; trocar por `Pageable` evita problemas quando o inventário cresce.
- **Evitar N+1**: revisar os carregamentos `LAZY`/`EAGER` e usar *fetch joins* onde fizer sentido.

### Modernização
- **Front-end**: Bootstrap 3 e jQuery cumpriram seu papel; uma reescrita da UI com um framework atual (ou ao menos Bootstrap 5) melhoraria a experiência.
- **Containerização**: um `Dockerfile` + `docker-compose` (app + MySQL) tornaria a execução em qualquer máquina trivial — resolvendo, de quebra, o "como rodar isso hoje".
- **CI/CD**: um workflow de GitHub Actions para build e testes a cada push.

### Experiência do Desenvolvedor
- **Script de *seed***: um carregamento inicial de usuário/grupo/permissões permitiria subir e logar sem passos manuais no banco.
- **Documentação de setup**: complementar este README com um guia passo a passo de ambiente.
- **Padronização de código**: aplicar um formatador (Spotless) e análise estática (Checkstyle/SonarLint) para consistência.
- **Internacionalização das mensagens**: extrair as mensagens de validação para arquivos de *messages* facilitaria manutenção e tradução.

---

<p align="center">
  <i>SIGEEST — onde uma carreira começou a ser construída, uma linha de código por vez.</i>
</p>
