---
layout: post
title: "CRUD com Rails - Editando um cadastro"
date: 2015-01-11 22:05:34 -0200
comments: true
categories: Rails, Iniciantes, Editando
---

Seguindo nosso CRUD, vamos trabalhar nesse post, a alteração de dados (Update).
Primeiramente crie o arquivo para testar o processo de alteração. Em `spec/features/products`
crie mais um arquivo nomeado `editing_products_spec.rb`. Nesse arquivo adicione
o seguinte código:

spec/features/products/editing_products_spec.rb
{% highlight ruby %}
require 'rails_helper'

feature 'Editando Produtos' do
before do
visit root_path
click_link 'Produtos'
end

    scenario "posso editar um produto" do
      click_link "Editar"
    end

end
{% endhighlight %}

Rode os testes e um erro devido a falta do link "Editar" será apresentado pelo rspec.

rspec spec/features/products/editing_products_spec.rb
{% highlight bash %}
Failure/Error: click_link "Editar"
Capybara::ElementNotFound:
Unable to find link "Editar"
{% endhighlight %}

Precisamos adicionar o link de edição a listagem de produtos. Vamos lá, no arquivo de
template `app/views/products/index.html.erb` adicione na listagem de dados, o link de edição:

app/views/products/index.html.erb
{% highlight erb %}
<% @products.each do |product| %>

<tr>
<td><%= product.name %></td>
<td><%= product.description %></td>
<td><%= link_to "Editar", edit_product_path(product), class: "btn btn-default" %></td>
</tr>
<% end %>
{% endhighlight %}

Mas somente isso não é suficiente, precisamos usar o FactoryGirl novamente, para ao menos
termos um registro e assim a nossa listagem ter o link. Ao teste adicione no início
do bloco before:

rspec spec/features/products/editing_products_spec.rb
{% highlight ruby %}
before do
product = FactoryGirl.create(:product)

    visit root_path
    click_link 'Produtos'

end
{% endhighlight %}

Rode seu teste e ocorrerá um erro devido a falta de uma ação edit em nosso controlador.
Adicione:

app/controllers/products_controller.rb
{% highlight ruby %}
class ProductsController < ApplicationController

... Adicione antes dos metodos privados ...
def edit

end

private
....
end
{% endhighlight %}

Rodando novamente os testes, teremos um erro pela falta do nosso template de edição.
Adicione um arquivo de template em `app/views/products/edit.html.erb`. Agora nosso
teste está passando. Ou seja, nossa aplicação já tem um página de edição. Nosso próximo
passo e exibir um formulário com os dados e alterá-los.

## Carregando os dados no formulário de edição

## Editando os dados

Vamos començar pelo nosso teste, onde vamos preencher o campo nome do produto, na
verdade vamos modificar o valor que deverá estar carregado. Adicione ao teste:

spec/features/products/editing_product_spec.rb
{% highlight ruby %}
scenario "posso editar um produto" do
click_link "Editar"
fill_in 'Nome', with: 'Produto 1'
click_button 'Salvar'

    expect(page).to have_content('Produto foi editado.')

end
{% endhighlight %}

Rode o teste. E teremos o error:

Failure/Error: fill_in 'Nome', with: 'Produto 1'
Capybara::ElementNotFound:
Unable to find field "Nome"

Ok, não temos o nosso form e o campo Nome no formulário de edição. Vamos reaproveitar
o formulário, usando o atual formulário de novo cadastro, tudo isso com o uso de
partials. Crie no diretório `app/views/products/` o arquivo `_form.html.erb`.
No arquivo `edit.html.erb` (crie ele se existir) implemente a chamada do partial:

app/views/products/edit.html.erb
{% highlight html %}

  <p>Editando</p>

<%= render 'form' %>
{% endhighlight %}

Abra o arquivo new.html.erb e transfira o codigo do formulário para o arquivo \_form.html.erb,
não esquecendo de adiocionar a chamada do partial ao formulário.

app/views/products/new.html.erb
{% highlight html %}

  <p>Criando</p>

<%= render 'form' %>
{% endhighlight %}

Já o partial ficará assim:

app/views/products/\_form.html.erb
{% highlight html %}
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
<%= f.submit "Salvar" %>
</p>
<% end %>
{% endhighlight %}

Obs: Mude o click_button do teste de novo registro para "Salvar".

Rode o teste `rspec spec/features/products/editing_products_spec.rb`. Erros, erros novamente...
ok mas isso é devido que temos a váriavel `@product` e nosso controller ela não existe. Vamos
a implementação no nosso controller. Vamos encontrar o registro que queremos editar, usando
o id que é passado a action edit junto com os params da requisição:

app/controllers/products_controller.rb
{% highlight ruby %}
class ProductsController < ApplicationController
... actions ...

    def edit
      @product = Product.find(params[:id])
    end

    ... private methods ...

end
{% endhighlight %}

Rode o teste o erro agora é diferente e bem claro. Pois queremos gravar a alteração
e o método de update não existe. Crie ele e adicione o código para realizar a alteração

app/controllers/products_controller.rb
{% highlight ruby %}
class ProductsController < ApplicationController
... actions ...

    def edit
      @product = Product.find(params[:id])
    end

    def update
      @product = Product.find(params[:id])
      if @product.update(product_params)
        redirect_to @product, notice: 'Produto foi editado.'
      end
    end

    ... private methods ...

end
{% endhighlight %}

Ao rodarmos o teste, ele vai passar. Agora vamos colocar uma cenário de teste negativo.
Deixaremos o campo nome em branco e vamos submeter, o campo nome é obrigatório, logo
uma mensagem de validação vai ser retornada, impedindo que a alteração aconteça:

spec/features/products/editing_product_spec.rb
{% highlight ruby %}
... outro cenario ...

scenario "quando nome em branco não posso editar um produto" do
click_link "Editar"
fill_in 'Nome', with: ''
click_button 'Salvar'

    expect(page).to have_content('Produto não foi alterado, verifique os erros.')
    expect(page).to have_content('Nome é muito curto (mínimo: 5 caracteres)')

end
{% endhighlight %}

E para finalizar adicione uma mensagem de alerta para quando ocorrer uma falha:

app/controllers/products_controller.rb
{% highlight ruby %}
class ProductsController < ApplicationController
... actions ...

    def edit
      @product = Product.find(params[:id])
    end

    def update
      @product = Product.find(params[:id])
      if @product.update(product_params)
        redirect_to @product, notice: 'Produto foi editado.'
      else
        flash[:alert] = 'Produto não foi alterado, verifique os erros.'
        render :edit
      end
    end

    ... private methods ...

end
{% endhighlight %}

Rode seus testes, e tudo estará verde. Finalizamos mais uma etapa, o código fonte como
sempre estará no meu [github](https://github.com/marceloboth/crud-rspec). Até..
