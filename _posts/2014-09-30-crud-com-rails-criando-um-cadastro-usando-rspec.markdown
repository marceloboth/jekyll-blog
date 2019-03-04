---
layout: post
title: "CRUD com Rails - Criando um cadastro usando Rspec"
date: 2014-09-30 23:27:36 -0300
comments: true
categories:
---

O acrônimo CRUD (Create, Read, Update e Delete) é algo que todo iniciante em linguagens de programação ou frameworks necessita saber, pois significa nada menos que as operações básicas para a criação/manutenção de cadastros.

O nosso primeiro passo é a parte de criação (C do CRUD). Para realizar todos os passos vamos desenvolver usando as metodologias de BDD/TDD, que irá nos possibilitar um código estável e mais fácil de manter.

## Criando um cadastro

Nessa primeira história vamos criar um resource (recurso) que representa um produto, é somente para fins didáticos, o nome é o que menos interessa. Para o produto vamos ter dois atributos que são necessários: nome e descrição. O nosso primeiro passo é criar a nossa feature de spec, em `spec/features` (crie esse diretorios se não existirem), crie o diretório products e um arquivo com o nome de `creating_products_spec.rb`.

Uma dica, sempre use a nomenclatura em inglês em sua aplicação Rails, garanto que você evitará muitas dores de cabeça, pois o Rails se torna muito mais amigável na língua do tio Obama.

No arquivo criado vamos especificar o que desejamos que o nosso cadastro faça. Abaixo está o código da especificação:

spec/features/products/creating_products_spec.rb
{% highlight ruby %}
require ‘rails_helper’

feature 'Criando Produtos' do
scenario "posso criar um produto" do
visit '/'

    click_link 'Novo Produto'

    fill_in 'Nome', with: 'Produto 1'
    fill_in 'Descrição', with: 'Descrição do produto 1 (um)'
    click_button 'Criar produto'

    expect(page).to have_content('Produto foi criado.')

end
end
{% endhighlight %}

Execute o comando `rspec spec/features/products/creating_products_spec.rb`. Esse comando executará a nossa feature de teste e que apresentará o seguinte erro:

Erro no bash
{% highlight bash %}

Failures:

1. Criando Produtos posso criar um produto
   Failure/Error: click_link 'Novo Produto'
   Capybara::ElementNotFound:
   Unable to find link "Novo Produto"
   {% endhighlight %}

O teste falhará pois não temos o link “Novo Produto” na rota “/”, que é a nosso root path(página inicial). Então antes de seguir, vamos criar o link na página inicial. Coloque o seguinte código dentro do arquivo `_navigation_links.html.erb`, que está na pasta layouts de nossas views (`app/views/layouts/_navigation_links.html.erb`):

app/views/layouts/\_navigation_links.html.erb
{% highlight erb %}

  <li><%= link_to 'Novo Produto', new_product_path %></li>
{% endhighlight %}

Edite a página de layout para possuir esse link:

app/views/layouts/application.html.erb
{% highlight erb %}
<%= render "layouts/navigation" %>
{% endhighlight %}

Rode os testes novamente e ocorrera um erro pois não definimos uma rota para os nossos produtos:

Erro no bash
{% highlight bash %}
Failure/Error: visit '/'
ActionView::Template::Error:
undefined local variable or method 'new_product_path'
{% endhighlight %}

Adicione a rota no arquivo `config/routes.rb`:

config/routes.rb
{% highlight ruby %}
resources :products
{% endhighlight %}

Rode os testes novamente e acontecerá um nova quebra na nossa feature. Dessa vez é devido que a nossa suite de testes até conseguiu encontrar a rota, mas não encontrou nenhum controller que condize-se com o resource procurado. Veja o erro:

Erro no bash
{% highlight bash %}
Failure/Error: click_link 'Novo Produto'
ActionController::RoutingError:
uninitialized constant ProductsController
{% endhighlight %}

Essa ficou fácil. Crie o controller, em `app/controllers/products_controller.rb`. Nesse arquivo declare um classe ruby com o nome de `ProductsController` extendendo de `ApplicationController`:

app/controllers/products_controller.rb
{% highlight ruby %}
class ProductsController < ApplicationController

end
{% endhighlight %}

Rode novamente o teste. Problema resolvido, mas outro surgiu. Agora o rspec não conseguiu encontrar uma action no controller, que refletisse com a operação de novo cadastro. Veja:

Ação ainda não existe
{% highlight bash %}
Failure/Error: click_link 'Novo Produto'
AbstractController::ActionNotFound:
The action 'new' could not be found for ProductsController
{% endhighlight %}

Ao controller criado instantes atrás, adicione a action new:

Ação de novo cadastro
{% highlight ruby %}
class ProductsController < ApplicationController
def new
end
end
{% endhighlight %}

Se rodarmos novamente os testes, nossa ação é encontrada, porém o template erb não existe, ocasionando outra quebra:

Erro pois o template não encontrado
{% highlight bash %}
Failure/Error: click_link 'Novo Produto'
ActionView::MissingTemplate:
Missing template products/new, application/new with ….
{% endhighlight %}

Devemos criar o nosso template na camada de visão do projeto. Então na pasta `app/views`, crie uma pasta chamada products e nela um arquivo com o nome `new.html.erb`. Feito isso rode o teste mais uma vez. Agora temos um problema diferente de todos até agora. O Capybara na tentativa de preencher os campos do formulário (que ainda não existe) acaba gerando um novo erro:

Não encontrou o elemento Nome
{% highlight bash %}
Failure/Error: fill_in 'Nome', with: 'Produto 1'
Capybara::ElementNotFound:
Unable to find field "Nome"
{% endhighlight %}

Como podemos ver, o Capybara não conseguiu encontrar o elemento com o field “Nome”. Isso é lógico, nós não criamos ainda o formulário com os campos. Porém antes teremos que criar uma instância de Product em nossa controller e passar a nossa view para que ela possua os campos necessários:

Implementação da action new
{% highlight ruby %}
def new
@product = Product.new
end
{% endhighlight %}

A constante Product é o nosso model, que deve ser criado em app/models/product.rb. Mas antes de sair criando o arquivo, vamos acelerar as coisas usando um gerador de código do Rails.

Comando para gerar tudo relacionado ao model
{% highlight bash %}
rails g model Pproduct name:string description:string
{% endhighlight %}

Esse generator (gerador) nos poupa um tempo criando uma série de arquivos, como o arquivo de migração do banco de dados com os campos necessários, nosso model extendendo de ActiveRecord que contém um infinidade de funcionalidades para trabalhar com nossos dados e arquivos de testes unitários.

Um Model deve ser capaz de prover o acesso a camada de dados e é por isso que ele utiliza o Active Record. O Model também provê um local para a lógica de negócio, bem como, fazer validações e associações entre modelos.

Entre os arquivos gerados, está o arquivo de migration. Migration é um mecanismo eficiente de manter um controle de versão de nosso banco de dados, onde podemos evoluir e retrocer nosso schema quando necessário, sem trabalho adicional. Nosso arquivo de migração está é gerado dentro da pasta db/migrate e o nome leva em considereção o timestamp do momento de geração, para nunca ter o mesmo nome e o Rails conseguir controlar o historico de migrações. Nosso arquivo db/migrate/[data]\_create_product.rb apresenta o seguite código Ruby:

db/migrate/[data]\_create_product.rb
{% highlight ruby %}
class CreateProducts < ActiveRecord::Migration
def change
create_table :products do |t|
t.string :name
t.string :description

      t.timestamps
    end

end
end
{% endhighlight %}

Devemos rodar o comando abaixo para ter o nosso banco de dados atualizado:

Comando para atualizar nosso banco de dados
{% highlight bash %}
rake db:migrate
{% endhighlight %}

O esquema de nomenclatura é interessante, o Rails irá ter o model com o nome product (singular), no entanto a tabela no banco de dados será products(plural).

Com nosso model product criado e o nosso banco com a tabela necessária, vamos voltar ao formulário. Execute o teste novamente:

Rodando os testes
{% highlight bash %}
rspec spec/features/products/creating_products_spec.rb
{% endhighlight %}

E veremos que nada mudou desde a ultima execução, porém agora temos nosso banco de dados pronto, e um objeto do tipo Product que dará os atributos necessários para montar nosso formulário de cadastro. Abra o arquivo app/views/products/new.html.erb e adicione o código abaixo:

app/views/products/new.html.erb
{% highlight erb %}

<h2>New Product</h2>
<%= form_for(@product) do |f| %>
  <p>
    <%= f.label :name, "Nome" %><br />
    <%= f.text_field :name %>
  </p>
  <p>
    <%= f.label :description, "Descrição" %><br />
    <%= f.text_field :description %>
  </p>
  <p>
     <%= f.submit "Criar produto" %>
  </p>
<% end %>
{% endhighlight %}

Formulário criado, devemos rodar nosso teste novamente. Fazendo isso veremos que nesse ponto, o que ocorre é que o formulário é preenchido, porém ao ser submitido, nenhuma ação para criar (create) é encontrado no nosso controller:

A ação create não esta definida
{% highlight bash %}
Failure/Error: click_button 'Criar produto'
AbstractController::ActionNotFound:
The action 'create' could not be found for ProductsController
{% endhighlight %}

Então para resolver isso devemos criar o nosso método create dentro do ProductContoller:

Definindo a action create
{% highlight ruby %}
def create
@product = Product.new(product_params)

if @product.save
redirect_to @product, notice: "Produto foi criado."
else
flash[:alert] = "Produto não pode ser criado."

    render "new"

end
end
{% endhighlight %}

Também defina um método privado para permitir que nossos parametros sejam lidos, sem serem bloqueados pelo mecanismo de Strong Parameters do Rails 4:

Adicionando os campos permitidos
{% highlight ruby %}
private
def product_params
params.require(:product).permit(:name, :description)
end
{% endhighlight %}

Como vamos usar flash messages do Rails, é necessário em nosso application.html.erb definir um local para
fazer a exibição das mensagens. Crie um diretório denominado `shared` dentro da pasta views. Nessa nova pasta
crie o arquivo `_messages.html.erb`. Adicione o seguinte conteúdo:

app/views/shared/\_messages.html.erb
{% highlight erb %}
<% flash.each do |name, msg| %>
<% if msg.is\*a?(String) %>

    <div class="alert alert-<%= name.to_s == 'notice' ? 'success' : 'danger' %>">
      <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
      <%= content_tag :div, msg, :id => "flash*#{name}" %>
    </div>

<% end %>
<% end %>
{% endhighlight %}

Com o partial criado, adicione a chamada de renderização no arquivo de layout, logo acima da
instrução `yield`:

app/views/layouts/application.html.erb
{% highlight erb %}

  <div class="container">
    <%= render 'shared/messages' %>
    <%= yield %>
  </div>
{% endhighlight %}

Rode seu teste e ele ira dizer que não encontrou a action show. Isso quer dizer que o nosso cadastro está sendo salvo, porém não temos uma ação para fazer a exibição de cadastro. Adicione a ação show ao seu ProductController:

Nossa ação para busca e exibir o usuário
{% highlight ruby %}
def show
@product = Product.find(params[:id])
end
{% endhighlight %}

Se rodarmos os testes vamos ter um erro por que o template correspondente a action, não pode ser encontrada, crie um o seguinte arquivo `app/views/products/show.html.erb`, e adicione o código abaixo:

app/views/products/show.html.erb
{% highlight ruby %}

  <h2><%= @product.name %></h2>
  <p><%= @product.description %></p>
{% endhighlight %}

Rodandos os testes, nossa feature vai passar. Se quiser testar manualmente, inicie o servidor com o comando `rails s` e realize um cadastro.

## Adicionando validações aos nossos campos

Como o nosso cadastro criado e funcionando, devemos adicionar algumas validações
para os nossos campos. Ao campo `nome` vamos verificar se ele foi informado,
se atende o tamanho minímo de 3 caracteres e o máximo de 50 e também, o nome deve
ser único. Já para o campo `descrição` verificáremos se foi informado e se possue
pelo menos 15 carácteres de texto. Nessa etapa, vamos usar testes unitários e testes
de integração. Os testes unitários vão garantir que as validações estão sendo exigidas
e o teste integrado deverá validar a operação de cadastro completo.

### Testando unitáriamente nosso model

Vamos iniciar testando no nosso model `Product` se algum valor a propriedade name foi
informado, assim impediremos que o nosso campo tenha um valor em branco. Vamos usar
a validação de tamanho para impedir o não preenchimento do campo. Adicione o seguinte código de ao arquivo:

spec/models/product_spec.rb
{% highlight ruby %}
require 'rails_helper'

describe Product do
before do
@product = Product.new(name: "Cadastro Exemplo",
description: "Descrição do cadastro Exemplo")
end

    describe "quando o nome não foi informado" do
      before { @product.name = ""}
      it { should_not be_valid }
    end

    describe "quando o nome é muito curto" do
      before { @product.name = "na"}
      it { should_not be_valid }
    end

    describe "quando o nome é muito longo" do
      before { @product.name = "n" * 50}
      it { should_not be_valid }
    end

end
{% endhighlight %}

Rode o comando `rspec spec/models/product_spec.rb` e o nosso teste deverá falhar,
pois estamos esperando que algo errado aconteça com o uso dos métodos `should_not be_valid`,
mas como não definimos as validações no nosso model, nada de errado ocorre e o nosso teste
falha. Adicione o código abaixo e rode o comando em seguida, nosso testes deve passar:

app/models/product.rb
{% highlight ruby %}
class Product < ActiveRecord::Base
validates_length_of :name, minimum: 5, maximum: 50, allow_blank: false
end
{% endhighlight %}

A validação do nome está quase completa, para finalizar adicione ao teste á baixo do último
bloco describe:

spec/models/product_spec.rb
{% highlight ruby %}
describe "quando o nome de produto já está sendo usado" do
before do
product_with_same_name = @product.dup
product_with_same_name.name = @product.name
product_with_same_name.save
end

    it { should_not be_valid }

end
{% endhighlight %}

Usamos o método dup quando queremos duplicar um módelo. Aqui queremos que o nome que já
exista, não seja duplicado, e é isso que o teste nos assegura. Ao model adicione:

app/models/product.rb
{% highlight ruby %}
validates_uniqueness_of :name
{% endhighlight %}

E rode o teste novamente. Tudo verde, o campo nome está validado, agora vamos
validar o campo `description`. Ao teste insira abaixo do ultimo bloco de describe
o seguinte código:

spec/models/product_spec.rb
{% highlight ruby %}
describe "quando a descrição não foi informada " do
before { @product.description = "" }
it { should_not be_valid }
end

describe "quando a descrição é muito curta" do
before { @product.name = "n" \* 15}
it { should_not be_valid }
end

describe "quando a descrição é muito longa" do
before { @product.name = "n" \* 255}
it { should_not be_valid }
end
{% endhighlight %}

E ao nosso modelo insira as validações:

app/models/product.rb
{% highlight ruby %}
validates_length_of :description, minimum: 15, maximum: 255, allow_blank: false
{% endhighlight %}

Mas que beleza! Nosso modelo está validando nossos dados e temos a garantia disso
com os testes unitários, hora de testar integrado e ajustar nossas mensagens.

## Testando de maneira integrada

Antes de continuar a implementação, vamos refatorar a nossa spec feature `spec/features/products/creating_products_spec.rb`. Vamos adicionar um bloco `before`,
e nele vamos colocar o que estará repetindo nos outros cenários, nossa feature deve ser
semelhante a isso:

spec/features/products/creating_products_spec.rb
{% highlight ruby %}
require 'rails_helper'

feature 'Criando Produtos' do
before do
visit '/'
click_link 'Novo Produto'
end

    scenario "posso criar um produto" do
      fill_in 'Nome', with: 'Produto 1'
      fill_in 'Descrição', with: 'Produto 1 (um)'
      click_button 'Criar produto'

      expect(page).to have_content('Produto foi criado.')
    end

end
{% endhighlight %}

Rode a spec com o comando `rspec spec/features/products/creating_products_spec.rb`, seu primeiro
cenário deverá falhar. Agora estamos validando nossos campos, e a nossa descrição não está
no tamanho correto, corrija, colocando uma descrição com pelo menos 15 caracteres, rode o teste
novamente e tudo estará correto.

Vamos agora focar nos cenários de teste negativo. Primeiro para valores do campo nome.
Crie um novo cenário abaixo do último criado, como o seguinte código:

insira em spec/features/products/creating_products_spec.rb
{% highlight ruby %}
scenario "com nome inválido não posso criar um produto" do
fill_in 'Nome', with: ''
fill_in 'Descrição', with: 'Produto com uma bela descrição de teste'
click_button 'Criar produto'

    expect(page).to have_content('Nome é muito curto (mínimo: 5 caracteres)')

end
{% endhighlight %}

Se rodar o teste, irá falhar, pois essa mensagem não está sendo exibida. Vamos criar
um arquivo, na pasta shared de nossas views, com o nome de \_errors.html.erb
(app/views/shared/\_errors.html.erb). Será um `partial`, que irá exibir todos os
erros de validação.

insira em app/views/shared/\_errors.html.erb
{% highlight ruby %}
<% if object.errors.any? %>

  <div id="error_explanation">
    <div class="alert alert-error">
      O formulário contém <%= pluralize(object.errors.count, "erro") %>.
    </div>
    <ul>
      <% object.errors.full_messages.each do |msg| %>
      <li>* <%= msg %></li>
      <% end %>
    </ul>
  </div>
  <% end %>
{% endhighlight %}

Também deve-se incluir esse arquivo no formulário de cadastro:

altere em app/views/products/new.html.erb
{% highlight ruby %}

  <h2>New Product</h2>

<%= form_for(@product) do |f| %>
<%= render 'shared/errors', object: f.object %>

<p>
<%= f.label :name, "Nome" %><br />
<%= f.text_field :name %>
</p>
<p>
<%= f.label :description, "Descrição" %><br />
<%= f.text_field :description %>
</p>
<p>
<%= f.submit "Criar produto" %>
</p>
<% end %>
{% endhighlight %}

Podemos rodar nosso teste novamente, e teremos ainda algum erro, mas estamos quase.
Precisamos ajustar para que nossos atributos (name e description) sejam reconhecidos
como nome e descrição. Basta mudar no nosso arquivo de tradução em `config/locales/pt-BR/pt-BR.yml`

adicione em config/locales/pt-BR/pt-BR.yml
{% highlight yaml %}
activerecord:
models:
product: Produto
attributes:
product:
name: Nome
description: Descrição
{% endhighlight %}

E mude a configuração padrão, para reconhecer a tradução bem como torna-la default
para o nosso sistema:

ajuste a tradução em config/application.rb
{% highlight ruby %}
module CrudRspec
class Application < Rails::Application
config.time_zone = 'Brasilia'
config.i18n.default_locale = :'pt-BR'
config.i18n.load_path += Dir["#{Rails.root}/config/locales/**/*.{rb,yml}"]
config.encoding = 'utf-8'
I18n.enforce_available_locales = false
end
end
{% endhighlight %}

E também baixe a tradução das mensagens. Salve no diretorio `config/locales/pt-BR` com
o nome de rails.pt-BR.yml. Baixe desse repositório:

https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/pt-BR.yml

Dessa maneira podemos traduzir tudo que o active record gera automáticamente, é uma maneira
muito flexível de se traduzir o nosso sistema. Rode novamente os testes e dessa vez, nossa
spec passou. Hora de refatorar. Agora que colocamos a tradução dos campos do active record,
não precisamos mais a tradução que está fixa nos campos do formulário de cadastro. Remova
a tradução:

remova a tradução em app/views/products/new.html.erb
{% highlight ruby %}

  <h2>New Product</h2>

<%= form_for(@product) do |f| %>
<%= render 'shared/errors', object: f.object %>

<p>
<%= f.label :name %><br />
<%= f.text_field :name %>
</p>
<p>
<%= f.label :description %><br />
<%= f.text_field :description %>
</p>
<p>
<%= f.submit "Criar produto" %>
</p>
<% end %>
{% endhighlight %}

Complete nossa spec com teste para descrição:

insira em spec/features/products/creating_products_spec.rb
{% highlight ruby %}
scenario "com descrição inválida não posso criar um produto" do
fill_in 'Nome', with: 'Meu produto'
fill_in 'Descrição', with: ''
click_button 'Criar produto'

    expect(page).to have_content('Descrição é muito curto (mínimo: 15 caracteres)')

end
{% endhighlight %}

Poderíamos colocar mais testes, mas não é esse o intuíto do post (Fica de incentivo para você).

O código fonte pode ser baixado aqui: https://github.com/marceloboth/crud-rspec. Baixando o código
faça checkout para o branch criando_cadastro: `git chechout criando_cadastro`.
