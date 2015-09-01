---
layout: post
title: "Utilizando o Ransack com Datatables e Ajax"
date: 2015-09-01 14:01:33 +0000
comments: true
categories: ruby rails javascript 
---
Em um projeto recente precisei usar o [ransack](https://github.com/activerecord-hackery/ransack). Para quem não sabe, o ransack é uma gem que tem como finalidade facilitar a implementação de um search em sua aplicação. Usando os conceitos do Rails de Convetion Over Configuration ele provê uma série de helpers que facilitam a implementação de formulários de pesquisa. O uso do ransack é bastante simples até, mas como nem tudo são flores precisei integrar na minha aplicação que além de usar ajax, precisava apresentar os resultados em um [datatable](https://www.datatables.net/).

Para usar o Datatable com Ajax na minha aplicação rails, eu acabei me baseando no excelente railscast do Ryan Bates. [link](http://railscasts.com/episodes/340-datatables). Eu sei que parece estar meio desatualizado, mas eu gostei da solução apresentada. então mão na massa:

Crie uma aplicação rails com um nome qualquer, aqui chamaremos de ransack-datatable-ajax-sample:

``` ruby
rails new ransack-datatable-ajax-sample
```

Depois da aplicação criada, precisamos adicionar algumas Gems e remover o turbolinks (nunca utilizo ele em meus projetos, pq sempre que vamos ter muito javascript esse cara acaba conflitando). Adicione as seguintes gem ao seu gemfile:

* ransack
* jquery-datatables-rails
* will_paginate (tem um bom suporte a paginação)

Dentro do seu applications.js adicione:

``` javascript
//= require dataTables/jquery.dataTables
```
 e não esqueça de remover a menção ao turbolinks.

Rode un bundle install para instalar as dependências e estamos prontos para começar.

Vou criar um model Product (sim em inglês, sempre prefiram desenvolver as coisas em inglês e depois traduzir. O seu mundo será muito mais felizi, principalmente no rails) com os atributos name, category (para fins didáticos sera uma String e não um relacionamento), price e quantity. Vamos usar os generators do rails, para criar toda essa estrutura inicial para nós.

``` ruby
rails g scaffold product name:string category:string price:decimal quantity:integer
rake db:create
rake db:migrate
```

Nesse ponto, ao rodar sua aplicação e acessar /products você deve ser capaz de manipular esse resource sem problemas. Agora vamos dar uma alterada no layout para incluir o form de cadastro na mesma página que o index, adicionar o suporte a todas as actions via ajax.

Dentro do arquivo app/views/products/index.html.erb adicione a seguinte linha logo no começo do arquivo:

``` ruby 
<div id="form">
  <%= render 'form', :locals => {product: @product} %>
</div>
```

Isso vai renderizar a partial do form dentro do index. Para que tudo funcione corretamente, precisamos fazer mais 2 alterações. Primeiro adicione, dentro do app/views/products/_form.html.erb na tag form_for o remote: true. Isso faz com que nosso form seja submetido utilizando AJAX. Vai ficar mais ou menos assim:

``` ruby
<%= form_for @product, remote: true, :html => { :class => "form-horizontal product" } do |f| %>
``` 

Não podemos esquecer de alterar nosso controller ProductsController em app/controllers/products_controller.rb. Dentro do método index adicione o seguinte:

``` ruby
 def index
    @products = Product.all
    @product = Product.new #isso ja deixa nosso objeto product instanciado, e passa essa referência para nosso partial form
 end
```

Nesse ponto nosso Create, ja deve estar funcionando. Porém ao cadasrar um novo produto, a view não é atualizada, ja que nosso cadastro é feito via ajax vamos precisar de um pouco de javascript para que tudo fique ok. Vamos criar 4 arquivos dentro da pasta view dos produtos: create.js.erb, edit.js.erb, destroy.js.erb, update.js.erb. Abaixo você pode ver o conteúdo dos arquivos:

create
edit
destroy
update

Agora precisamos fazer algumas alterações no nosso controller para que tudo funcione. Seu controller deve ficar parecido com isso:


controller

Atente para as tratativas nos métodos create, update e destroy com a inclusão das respostas no formato js

Vamos precisar criar um partial com o conteúdo da tabela, para facilitar o append de dados na mesma. Crie um arquivo com o nome _product.html.erb dentro da pasta app/views/products com o seguinte conteúdo:

``` html
<tr id="product_<%= product.id %>">
  <td><%= product.id %></td>
  <td><%= product.name %></td>
  <td><%= product.category %></td>
  <td><%= product.price %></td>
  <td><%= product.quantity %></td>
  <td><%=l product.created_at %></td>
  <td>
    <%= link_to t('.edit', :default => t("helpers.links.edit")),
                edit_product_path(product), remote: true, :class => 'btn btn-default btn-xs' %>
    <%= link_to t('.destroy', :default => t("helpers.links.destroy")),
                product_path(product),
                :method => :delete,
                remote: true,
                :data => { :confirm => t('.confirm', :default => t("helpers.links.confirm", :default => 'Are you sure?')) },
                :class => 'btn btn-xs btn-danger' %>
  </td>
</tr>
```
Note que foi adicionado a tag remote: true aos métodos edit e detroy.

Agora altere o tbody do seu index, para que carregue este partial no loop responsável por montar a tabela. Vai ficar assim:

``` ruby
 <tbody>
    <% @products.each do |product| %>
       <%= render product %>
    <% end %>
  </tbody>
```

Primeira parte esta pronta. Uffa, agora todo nosso crud esta funcionando com AJAX.







