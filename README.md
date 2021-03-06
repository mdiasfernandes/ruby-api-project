# README

- Create a rails project only for APIs:
    - `rails new ruby-api-project --api`

- Up the server with rails app:
    - `rails s`

- Create a scaffold:
    - `rails g scaffold contact name:string email:string birthdate:string value:type`

- Run:
    - `rails db:migrate` - Alteração da estrutura do BD e maném um log de alterações realizadas

- Generate a rake:
    - `rails generate task dev setup`

- Run rake:
    - `rails dev:setup` - Executar uma operação que não necessariamente está associada às regras de negócio

- Recursos:
    - config/routes - descreve os recursos existentes na minha aplicação
    - o recurso(rota) contacts foi criado, ou seja, URLs disponíveis para solicitar ao servidor algum serviço.
    - os recursos são sempre escritos no PLURAL.
    - recursos são sempre SUBSTANTIVOS e nunca VERBOS

- curl:
    - Com o '-v' na chamada com curl, é possível identificar toda a estrutura da chamada e não apenas a resposta
    - Com o '-X' na chamada com curl, é possível definir qual verbo HTTP será utilizado na chamada (GET, PUT, POST, PATCH)
    - Com o '-i' na chamada com curl, é possível identificar todo o cabeçalho da resposta à chamada HTTP
    - Com o '-H' na chamada com curl, é possível indicar que uma parâmetro no Header será enviado, por exemplo, `-H Content-Type: application/json`
    - Com o '-d' na chamada com curl, é possível enviar um body para a chamada (POST, PUT e PATCH), por exemplo, `-d '{"name": "value", "data": 500}'`
        
- Cabeçalho de uma resposta HTTP:
    - Start-line: Versão HTTP usada + Status da chamada
    - Header-Fields: Representação dos metadados, ex. Content-type: application/json
    - Empty-line: Separa o cabeçalho da resposta propriamente dita
    - Message-Body: Corpo da resposta à chamada HTTP (resposta do servidor)

- REST e RESTful
    - Formalização de um conjunto de melhores práticas denominadas CONSTRAINTS
    - CONSTRAINTS:
        - Client/Server, ou seja, separação das responsabilidades entre Client e Server
        - Stateless, cada requisição é única e deve ser suficientemente completa em infos necessárias para ser tratada com sucesso pelo servidor
        - Cache, respostas passíveis de cache
        - Interface Uniforme, deve haver padronização, por exemplo, Recursos ou até mesmo as mensagens que devem ser auto-explicativas
        - Sistemas em camadas, uso de um load balancer, por exemplo, pois isso permite a escalabilidade da aplicação sem impactá-la
    - REST é um conjunto de boas práticas denominadas CONSTRAINTS
    - Uma API que não segue os princípios REST é apenas uma API HTTP
    - Uma API que segue os princípios REST, temos uma API RESTful

- HTTP status code
    - 1xx - informacional
    - 2xx - sucesso entre client e server
    - 3xx - redirecional (passo adicional)
    - 4xx - erro do lado do CLIENT
    - 5xx - erro do lado do SERVER
    - No rails >>> status: `:rails_code`

- Map/Collect
    - map >>> constrói um novo objeto, um novo array, por exemplo
    - each >>> não cria um novo objeto, fica restrito à execução da sentença

- Render JSON
    - Active Support JSON
        - Possui `ActiveSupport::JSON.encode(hash)` para transformar um hash em um json, e `ActiveSupport::JSON.decode(json)` para reverter um json para hash
        - Elemento raíz, `root: true`, vem com o nome da origem dos dados
        - Com o `only` e o `except` é possível manipular o que será exibido usando o `render json:`
        - Com o `as_json` sobrescrevo o método original e passo via parâmetros quais personalizações quero para as responses de todas as APIs de um determinado recurso

- Novo CRUD com tabelas relacionadas
    - `rails g scaffold Kind description:string` - Criação de um CRUD padrão, nome da tabela é `Kind`
    - `rails g migration add_kind_to_contact kind:references` - Cria uma chave estrangeira 'referência' dentro da tabela contacts
    - `rails db:migrate`
    - `rails db:drop db:create db:migrate dev:setup`
    - `belongs_to`, `has_one`, `has_many`, `has_and_belongs_to_many`

- Trazer outros dados de uma entidade relacionada - inclusão da lógica em `models/contact.rb`
    - Exibição aninhada: `include: {kind: {only: :description}}`
    - Exibição na estrutura inicial:
        - Criação de um método:
            `def kind_description`
                `self.kind.description`
            `end`
        - Chamada desse método sobrescrevendo o método `as_json`:
            `methods: [:kind_description]`
        - Para a não exibição de dados:
            `except: :kind_id`

- PERSONALIZAÇÃO da exibição de atributos quando um determinado recurso é consumido:
    - Sobrescrita de método `as_json` para todos os recursos
        - No model eu posso sobrescrever o método originalmente implementado para que a exibição personalizada seja padrão para todos os recursos
    - Personalizar a exibição especificamente em um recurso
        - Definir na estrutura do `Controller`

- Registro de dados na API
    - No método `contact_params` é necessário que os atributos que serão registrados na base sejam explicitamente identificados
    - Ou não utilizar o parâmetro `optional`, como por exemplo `belongs_to :kind, optional: true`

- Usar I18n para internacionalização de dados, como por exemplo datas.
    - `I18n.l(Date.today)` => 01/01/2000

- Criação de mais relacionamentos
    - `rails g model Phone number:string contact:references` - Modelo que possui uma referência para a entidade contact
    - `rails db:migrate`
    - `rails db:drop db:create db:migrate dev:setup`
    - `rails dev:set_phones`

- Quando um `Model` é criado, nesse caso `Phones`, só é possível atualizá-lo na criação ou atualização de um contato

- Relacionamento `has_one`
    - `rails g model Address street:string city:string contact:references`
    - Mesmo com uma associação `has_one` é possível criar mais de um endereço para o mesmo contato, 
    entretanto, somente o último endereço cadastrado é válido, ao fazer uma busca pela relação de endereço(s) desse determinado contato
    nota-se que exite mais de um endereço, mas toda vez que o endereço associado ao contato é buscado, apenas o último é exibido.

- CRUD `has_one`
    - `update_only` - Só permite atualização dos registros e não a criação de novos
    - Do que foi executado no rails 6, quando é solicitado a atualização dos dados em uma relação `has_one` sem a indicação do `id`
    o seguinte erro ocorreu: 
        `"status": 422,`
        `"error": "Unprocessable Entity",`
        `"exception": "#<ActiveRecord::RecordNotSaved: Failed to remove the existing associated address. The record failed to save after its foreign key was set to nil.>",`

- CORS: Cross-Origin Resource Sharing
    - config/initializers/cors.rb
    - Permite que outros sites 'servidores' acessem/compartilhem dos recursos existentes na minha aplicação
    - **SERIA POSSÍVEL CONSTRUIR AS APIs GERADORAS DE CONTEXTO COM RAILS E FAZER COM QUE ELAS SE COMUNIQUEM COM O _CORE_ VIA _CORS_???**

- Active Model Serializers (AMS)
    - Serialização é o processo de tradução de estruturas de dados ou estado de objeto em um formato que possa ser armazenado e reconstruído posteriormente no mesmo ou em outro ambiente computacional.
    - {json:api} É uma especificação para construir APIs em JSON
    - Para criar um serializer para os models que você deseja trabalhar: `rails g serializer MODEL` -> contact, kind, etc.
    - Com o serializer construído, as _responses_ são 'fornecidas' pelo próprio serializer durante a chamada, por exemplo `render json: @contacts`
    - **O serializer é usado para 'exibição' dos dados, ou seja, através dele é possível editar quais atributos exibir ou não**
    - Serializer: _Descrevem QUAIS atributos e relacionamentos devem ser serializados._
    - Adapters: _Descrevem COMO os atributos e relacionamentos devem ser serializados._
    - Para formatar usando o json_api como modelo: `ActiveModel::Serializer.config.adapter = :json_api` + ams
    - Para trabalhar com as associações, basta informá-las assim como no model `belongs_to`, `has_many`, `has_one`
    - Com o uso de `meta: {}` é possível adicionar metadados para complementar os dados renderizados na response da chamada

- Links (HATEOAS)
    - Hypermedia - links dinâmicos que disponibilizam acesso ao meu servidor relativos àquele recurso

- Foreman (Procfile)
    - Usado para definir todas as ações que eu desejo executar no momento em que estou subindo o servidor da minha aplicação.

- Media Type
    - Definição do formato do dado e como ele deve ser interpretado pela máquina, ou seja, diferenciar entre JSON e XML.
        - application/json
        - application/xml
        - multipart/form-data
        - texthtml
    - Para informar o media type, usamos o header field **Accept** no momento da requisição
    - Mime Type é a mesma coisa que Media Type

- Desserialização
    - Faz com que o objeto JSON que segue as regras json:api seja interpretado corretamente pelo rails

**PORTANTO**

- **Models**: for handling _data_ and _business logic_
- **Controllers**: for _handling the user interface_ and application
- **AMS - _Active Model Serializer_**: Tradução (_serialization_ and _deserialization_) para estruturação e manipulação de dados no lugar dos _Models_

- Relacionamentos e Rotas
    - Singular Resources: `resource :resource_name`
    - Namespace and Routing: `resources :articles, path: '/admin/articles'`
    - **PASSO A PASSO**
        - Construa a rota em `/ruby-api-project/config/routes.rb`
        - Defina o `link` no `serializer` correspondente ao `controller` de manipulação dos dados que você vai chamar
        - _(Se necessário)_ construa o controller de manipulação dos dados
    
- Nomenclatura dos `controllers`
    - Todo novo controller deve ser nomeado no plural, independentemente se o tipo de relacionamento seja `has_one`

- CRUD, Nested Attributes e AMS
    - **Segundo a documentação do json:api, não é possível cadastar, por exemplo, um contato com os seus respectivos tipos, telefones e endereço (`relationships`), portanto, é necessário que, nesse caso, 3 outras chamadas sejam feitas além da própria chamada POST de um novo contato.**

- CRUD AMS
    - **Melhor exemplo de controller do projeto é no arquivo `app/controllers/addresses_controller.rb`**

- Autenticação HTTP
    - Tipos de autenticação
        - Basic (básica - `base64`)
            - **Encode64**
                - `require 'base64'` -> `Base64.encode64('user:pass')` -> Tem um `\n` no final
                - `require 'base64'` -> `Base64.strict64('user:pass')` -> NÃO tem um `\n` no final
        - Digest (resumida - `MD5`)
            - **MD5**
                - `require 'digest/md5'` -> `Digest::MD5.hexdigest('user:pass')`
    - Trabalhando com serviços é necessário enviar um `Header` com o parâmetro `Authorization: senha-encriptada`

- Autenticação baseada em TOKEN (Comumnte usada para autorizar chamadas às APIs)
    - `Authorization: Token 6afc7f...`
    - Autenticação baseada em `token` é `Stateful`, ou seja, quando o servidor precisa conhecer os meus dados
    - Autenticação em que não é preciso que o servidor me "conheça" é `Stateless`

- JWT - JSON Web Tokens
    - Guarda no próprio `Token` as informações que o servidor precisa **validar**
    - No servidor existe uma `chave` que consegue verificar se o Token é valido ou não
    - Estrutura
        - Basicamente um TOKEN dividido em 3 partes -> `HEADER`, `PAYLOAD` e `SIGNATURE`
        - Exemplo
            - `browser:` POST/login with username+password  ->  `server:` Cria JWT com segredo
            - `browser:` Retorna JWT para o browser         <-  `server:` Cria JWT com segredo
            - `browser:` Envia JWT no header Authorization  ->  `server:` Checa assinatura JWT com segredo. Pega dados do usuário do JWT
            - `browser:` Envia resposta para o cliente      <-  `server:` Checa assinatura JWT com segredo. Pega dados do usuário do JWT
        - Para definir o tempo de expiração, com Ruby, use o atributo `exp:`

- Versionamento de APIs
    - Mais comumente utilizado é **Segmento de URL**: `v1/users/permissions`
    - Com **Query Parameter**:
        - `localhost:3000/users?version=2`
        - `if params[:version] == '1'  \n  @contacts = Contact.all`
    - Com versionamento de `controllers`:
        - Criação de `modules` diferenciando `v1` de `v2`
        - Criação de pastas que diferenciem as versões e em cada classe dentro dessas pastas é necessário associá-las ao módulo da versão correspondente, ou seja, `module v1`, `module v2`, etc.
    - `constraints`: Mecanismo que permite especificar a rota a ser chamada com base em um parâmetro, declarado no arquivo de definição de `rotas`
    - _Maiores detalhes de como documentar e versionar uma API será realizado em outro repo_

- Diff entre `Accept` e `Content-Type`:
    - `Accept`: informa ao servidor o que eu espero receber, ou seja, qual formato de resposta será enviado na resposta à chamada
        - Espero receber um `json`
    - `Content-Type`: quando vou enviar dados ao servidor, preciso indicar qual o formato que estou enviando
        - Estou enviando um `xml`

- Paginação
    - `kaminari` - `Gem` utilizada para paginação de APIs

- Caching
    - Baseado em `Headers`
    - Qualquer valor que é difícil e computacionalmente custoso de obter deve ser cacheado;
    - Influencia inclusive na demora de resposta aos dados consultados;
    - Objetivos:
        - 1º Eliminar o envio de requisições o máximo possível;
        - 2º Reduzir os dados de resposta;
    - O primeiro objetivo pode ser alcançado usando `Cache-Control` - _método utilizado `expires-in`_
        - Com o tempo do `Cache-Control` configurado, não é necessário realizar uma nova chamada ao servidor
        - `Max-Age`: É especificado em quanto tempo o recurso pode ser cacheado, essa definição também pode ser feita por intermediários e não necessariamente pelo navegador
        - `Public/Private`: **Public**:Qualquer `client` pode realizar cache, **Private**: Somente o `browser` pode realizar cache
        - `No-Cache/No-Store`: **No-Cache**: A resposta pode ser cacheada mas não pode ser reutilizada sem sem consultada no servidor, **No-Store**: Sem cache _at all_, ou seja, em lugar nenhum
    - O segundo objetivo pode ser alcançado usando o mecanismo de validação `ETag` ou `Last-Modified` - _método utilizado `fresh_when`_
        - Na primeira chamada um `ETag` é devolvido na resposta
        - Na segunda requisição esse mesmo `ETag` é enviado ao servidor através de um parâmetro `Header` e o servidor avalia se aquele token está associado é o mesmo da versão dos dados contidos no servidor, e só devolve ao `client` os dados novos ou atualizados

- Rack e Rack Middleware
    - `Rack` é um pacote Ruby que provê uma interface para o servidor web se comunicar com a aplicação
    - `Middleware` é qualquer componente de software que auxilia, mas não está diretamente envolvido na execução de uma tarefa
    - `Rack-Middleware` é um componente situado entre o cliente e o servidor e que processa requisições de entrada e respostas
        - _Ex: No `CORE` o `Newrellic` é uma implementação de `middleware` que intercepta os dados da `request/response` e passa adiante sem influenciá-los_
        - Realizam funções bem específicas, podendo adicionr ou remover aspectos da `request/response`

- TDD para API
    - `MiniTest` - framework padrão do Rails, mas o `Rspec` será usado como `gem` de testes, está no diretório `/ruby-api-project/test`
    - Gerar a pasta de testes: `rails generata rspec:install`
    - Comando para execução dos testes: `bundle exec rspec`
    - Construir uma estrutura de pastas e arquivos que se assemelhe a estrutura de pastas e arquivos da minha aplicação e, com isso, testar cada parte dela independentemente do que seja, portanto, testes de `controllers`, `models`, `rack middlewares` etc.
    - O rspec irá buscar por todos os arquivos teste que terminaram com `_spec.rb`
    - Com o `require 'rails_helper'` estou chamando também o `spec_helper.rb`
    - Trabalhando com HASHES `h[:a]` == `h.fetch(:a)`
    - **REPLICA o meu banco de dados do ambiente _development_ para o ambiente _test_: `rails db:migrate RAILS_ENV=test`**


    