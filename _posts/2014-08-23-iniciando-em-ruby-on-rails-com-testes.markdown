---
layout: post
title: "Iniciando em Ruby on Rails com testes"
date: 2014-08-23 10:25:13 -0300
comments: true
---

Esse post é o primeiro de uma série que irão servir como um próximo passo a quem estiver
iniciando com Ruby on Rails, algo além de "construa um blog em quinze minutos" e
sim algo mais útil ao mundo real.

Nesse post veremos sobre como gerar nossa aplicação, configurar nossos testes,
instalar uma biblioteca de front-end e deixar tudo pronto para iniciar o desenvolvimento.

# Criando nossa aplicação

Criar a aplicação é um passo simples, onde usaremos os geradores do Rails para
fazer o trabalho árduo de montar a estrutura e deixar tudo configurado, baseado
no conceito de "Convenção sobre configuração". No terminal, navege até o diretório
que deseja criar o projeto e digite o seguinte comando:

Criar o projeto Rails:

```bash
  rails new crud-rspec -T
```

> O comando rails new com o argumento -T, é para sinalizar ao gerador, que não queremos
> o conjunto de testes padrão do Rails, já que vamos usar o Rspec. Para ver mais opções
> de configuração utilize o help do comando: `rails new -h`

Aguarde a finalização da construção...

Saida da linha de comando

```bash
  rails new crud-rspec -T
  create
  create README.rdoc
  create Rakefile
  create config.ru
  create .gitignore
  create Gemfile
  create app
  create app/assets/javascripts/application.js
  create app/assets/stylesheets/application.css

        ..................
        ..................

  Your bundle is complete!
  Use `bundle show [gemname]` to see where a bundled gem is installed.
  run bundle exec spring binstub --all

  - bin/rake: spring inserted
  - bin/rails: spring inserted
```

No seu bash ou outra ferramenta de linha de comando (recomendo zsh), navegue para
o diretório que foi criado usando `cd crud-rspec`. Abra o diretório com o seu editor
favorito.

O framework Rails nos impõe algumas convenções e uma estrutura inicial para seguir.
Primeiramente temos o uso de uma arquitetura MVC onde temos nos models, que manipulam
nossos dados, controllers que gerenciam o que deve ser executado e para onde entregar
as informações e nossas views que fazem a interação, recebem as entradas e mostram
as saidas ao usuário. Isso tudo está na pasta `app` do projeto. Esse é o local onde
passaremos a maior parte do nosso desenvolvimento. Abaixo uma descrição básica de
todos os diretórios criados.

`app`: Local do código principal da aplicação. Models, Controllers, Views, Helpers,
CSS e código Javascript da nossa aplicação ficam nessa pastas.

`bin`: Armazena os scripts dos geradores do `rails`.

`config`: Configuração do banco de dados, intenacionalizações, especificidades de
algumas gems (Devise,SimpleForm, etc), rotas da aplicação e muitas outras definições.

`db`: Mantém o esquema e as migrações da base de dados.

`lib`: Aqui ficam todos os códigos que não são diretamente ligados a aplicação.
São armazenadas as tarefas `rake`, bem como tasks que são executadas fora do ambiente
web.

`log`: Arquivos de log, simples assim :).

`public`: Páginas de erro (404, 500) e arquivos estáticos como por exemplo o favicon.ico
e o robots.txt.

`tmp`: Guarda o cache e PID da aplicação.

`vendor`: Local onde são colocadas bibliotecas que não sejam gems.

A construtor de projetos do Rails poupa a nós um bom tempo, pois não precisamos nos preocupar
com a arquitetura inicial do projeto, pois ele que cria as pastas e arquivos, de acordo
com a nossas escolhas.

# Configurar nossos testes

Vamos iniciar a configuração dos nossos testes. Primeiro passo é instalar a gem
de testes [Rspec](http://rspec.info/) e a gem para testes de aceitação
[Capybara](http://jnicklas.github.io/capybara/), pra isso devemos adicionar elas ao nosso
arquivo Gemfile, e devemos definir a qual grupo de gems elas pertencem (desenvolvimento, testes ou produção). Como
nossos testes apenas são usados em teste e produção, ficará de seguinte maneira:

Adicione as gems ao Gemfile

```ruby
group :development, :test do
  gem 'rspec-rails'
  gem 'capybara'
end
```

em seguida execute o camando de instalação de gems no terminal:

Execute o comando
{% highlight bash %}
bundle install
{% endhighlight %}

Após instalada as gems, vamos gerar os arquivos de configuração do rspec e
também os diretórios que servem de base para criar nossos testes:

Gerar configs
{% highlight bash %}
rails generate rspec:install
{% endhighlight %}

Se observarmos a saída do nosso console

Saída do console
{% highlight bash %}
rails generate rspec:install
create .rspec
create spec
create spec/spec_helper.rb
create spec/rails_helper.rb
{% endhighlight %}

veremos que o diretório rspec foi gerado, e os arquivos de configuração também.

# Are you ready?

Criamos nosso projeto e também configuramos o rspec, poderiamos implementar nossa
primeira funcionalidade, mas antes, vamos configurar uma biblioteca de front-end, no caso
twitter bootstrap. Para isso adicione a gem do twitter bootstrap em nosso Gemfile:

Adicione ao Gemfile
{% highlight ruby %}
gem 'bootstrap-sass'
gem 'autoprefixer-rails'
{% endhighlight %}

Rode novamente o `bundle install` para instalar a gem e na seqüência configure, da seguinte
maneira:

- Renomeie o arquivo app/assets/stylesheets/application.css para app/assets/stylesheets/application.css.scss

- Adicione ao arquivo application.css.scss:

{% highlight css %}
@import "bootstrap-sprockets";
@import "bootstrap";
{% endhighlight %}

- Adicione ao app/assets/javascript/application.js `//= require bootstrap-sprockets`

- Crie um controller com uma view para servir como o Index de nossa Web Application:

Crie o controlle e a view
{% highlight bash %}
rails g controller home index
{% endhighlight %}

Isso criará uma série de arquivos, que são: o nosso controller, nossa view, arquivos
de css e javascript, testes e também é feita uma alteração em nosso arquivo de rotas:

Arquivos gerados
{% highlight bash %}
rails g controller home index
create app/controllers/home_controller.rb
route get 'home/index'
invoke erb
create app/views/home
create app/views/home/index.html.erb
invoke rspec
create spec/controllers/home_controller_spec.rb
create spec/views/home
create spec/views/home/index.html.erb_spec.rb
invoke helper
create app/helpers/home_helper.rb
invoke rspec
create spec/helpers/home_helper_spec.rb
invoke assets
invoke coffee
create app/assets/javascripts/home.js.coffee
invoke scss
create app/assets/stylesheets/home.css.scss

{% endhighlight %}

# Nosso primeiro teste

Agora que temos nossa estrutura gerada, nossos testes configurados e nosso front-end
pronto, vamos definir a nossa página principal. Nosso primeiro passo é definir que
a nossa rota raiz da aplicação seja o index do controlador home. Com isso feito vamos
colocar um menu superior na página e um link para o cadastro de produtos que será implementado
no próximo post. Crie um diretório denominado `features` dentro do diretório `spec`,
e dentro da pasta criada crie um arquivo chamado `reading_products_spec.rb`.
Nesse arquivo implemente o teste da seguinte maneira:

Escrevendo nosso teste spec/features/reading_products_spec.rb
{% highlight ruby %}
require "rails_helper"

feature "Listando Produtos" do
it "consigo acessar o link da página" do
visit root_path
end
end
{% endhighlight %}

Antes de rodar o teste, desabilite o lançamento de warnings do rspec. Faça isso
removendo a linha que contém o conteúdo `--warnings` do aquivo .rspec que está na raiz do projeto.
Agora rode o teste com o comando `rspec spec/features/reading_products_spec.rb`. Um erro
ocorrerá:

Erro ocorrido
{% highlight bash %}

1. Listando Produtos consigo acessar o link da página
   Failure/Error: visit root_path
   NameError:
   undefined local variable or method `root_path' for #<RSpec::ExampleGroups::ListandoProdutos:0xa824e84>
   {% endhighlight %}

Esse erro acontece pois estamos tentando visitar a raiz de nosso site e ela ainda não
está configurada nas rotas. Abra o arquivo `config/routes.rb`, e deixe a sua implementação
na seguinte forma:

Defina o root_path config/routes.rb
{% highlight ruby %}
Rails.application.routes.draw do
root 'home#index'
end
{% endhighlight %}

Salve e rode o teste novamente. Nosso teste passou, vamos a mais uma parte da implementação,
que é onde vamos colocar o cabeçalho e o link para o recurso de listagem de cadastros.
Ao teste adicione:

Encontrando o link de listagem spec/features/reading_products_spec.rb
{% highlight ruby %}
feature "Listando Produtos" do
it "consigo acessar o link da página" do
visit root_path
expect(page).to have_link('Produtos')
end
end
{% endhighlight %}

Rodando nosso teste, teremos uma quebra, pois esse link não existe em nossa página
inicial. Vamos colocar um menu, com o conjunto de estilos do Twitter Bootstrap a
nossa página de layout `app/views/layouts/application.html.erb`:

Implemente a página de layout app/views/layouts/application.html.erb
{% highlight erb %}

  <body>

    <div class="navbar navbar-inverse navbar-fixed-top" role="navigation">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="#">Crud com RSPEC - Uhuu!</a>
        </div>
        <div class="collapse navbar-collapse">
          <ul class="nav navbar-nav">
            <li class="active"><a href="#">Início</a></li>
            <li><a href="#about">Produtos</a></li>
          </ul>
        </div>
      </div>
    </div>

    <div class="container">
      <%= yield %>
    </div>

  </body>
{% endhighlight %}

Rode os testes novamente, agora tudo vai passar. Temos nossa página inicial definida,
com o uso de testes de integração. O que fizemos foi, adicionar uma rota raiz para
nossa aplicação, colocar um menu e um link para o cadastro de produtos (que está sem caminho),
e tudo isso guiado por testes, usando Rspec e Capybara.

Finalizamos nossa primeira parte, onde configuramos nossa aplicação com algumas
ferramentas que o Rails e o Ruby nos proporcionam para desenvolvermos com Agilidade
sem perder qualidade. Nos próximos posts vamos implementar um Crud usando Rspec, Capybara e Rails.

O código pode ser baixado no [Github](https://github.com/marceloboth/crud-rspec), somente observe que cada post está
dividido por branch.
