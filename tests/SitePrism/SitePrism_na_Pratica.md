# O que são page objects?

Page Objects são abstrações da interface do usuário que podem ser usadas nos testes automatizados. A maneira mais comum é modelar cada página como uma classe e usar instância dessa classe em seus testes. SitePrism é baseado em torno deste conceito, como você verá abaixo.

# O que é o SitePrism?

O SitePrism é uma DSL (Domain Specific Language) criada para facilitar a criação de page objects para testes automatizados em Ruby, utilizando o Capybara.

# Estrutura do projeto exemplo

Como exemplo, utilizarei a estrutura de projeto mostrada neste post do Ship it!, que é baseada em Cucumber e Capybara para criação dos testes automatizados. Como padrão deste projeto, todos os arquivos referentes a page objects ficam no diretório features/pages.

# Adicionando SitePrism ao seu projeto

Para adicionar o SitePrism ao projeto é muito fácil, basta adicionar a gema site_prism ao seu Gemfile. Como o SitePrism é uma DSL criada para ser utilizada com o Capybara, a gema dele também deve estar no seu projeto.

```ruby
  gem 'site_prism'
```

Como nosso projeto utiliza o Cucumber, devemos adicionar um require do SitePrism ```require 'site_prism' no arquivo features/support/env.rb,``` que é o arquivo que carrega todo o contexto antes de executar os testes do cucumber.

# Criando um Page Object

Para criarmos um page object com o SitePrism é simples. Utilizarei como exemplo a página de Conta do RD Station, conforme imagem abaixo:

Página de configurações de conta

Como nosso page object é referente a conta, criaremos o arquivo account.rb dentro da pasta features/pages/account, e nesse arquivo implementaremos uma classe chamada AccountPage que herda os métodos de SitePrism::Page.

```ruby
#features/pages/account/account_page.rb

class AccountPage < SitePrism::Page
  <elementos>
end
```

E no arquivo de steps da feature basta instanciar a page da seguinte forma:

```ruby
#features/step_definitions/account/account_steps.rb
age = AccountPage.new
Set URL
O método set_url nos permite definir a URL correspondente ao nosso page. Por exemplo:

class GooglePage < SitePrism::Page
  set_url "http://www.google.com"
end
```

Caso você tenha configurado a url padrão do capybara Capybara.app_host para a url da sua aplicação, basta adicionar o path da sua page no set_url:

```ruby
# features/pages/account/account_page.rb
class AccountPage < SitePrism::Page
  set_url "/accounts"
end
```

Também é possível configurar uma url com parâmetros ou queries:

```ruby
# features/pages/account/account_page.rb
class AccountPage < SitePrism::Page
  set_url "/accounts"
  set_url "/account{/account_id}"
  set_url "/accounts{?account_query*}"
end
```

```ruby
# features/step_definitions/account/account_steps.rb
account_page = AccountPage.new

When(/^I visit the account page$/) do
  account_page.load #=> http://app_host/accounts
  # Load com parâmetros e query
  account_page.load(account_id: 123) #=> http://app_host/account/123
  account_page.load(account_query: {'city': 'florianopolis', 'status': 'active'}) #=> http://app_host/accounts?city=florianopolis&status=active
end
```

# Elementos individuais

Para adicionarmos elementos da página em nosso page object, basta utilizarmos o método element passando por parâmetro o nome do nosso elemento e o seu respectivo seletor. Dica: Dê preferência sempre para os ids dos elementos.

```ruby
# features/pages/account/account_page.rb
class AccountPage < SitePrism::Page
  set_url "/accounts"

  element :account_name, '#account-name'
  element :save_button, "input[name='save-btn']"
  element :account_language, '.account-language'
end
```

Todo elemento possui uma série de métodos que podem ser utilizados. Os métodos variam de acordo com o tipo do elemento, e os mais utilizados são:

```ruby
Set: É utilizado para inserir texto no elemento.
Click: Efetua a ação de clique no elemento.
Text: Retorna o texto do elemento. Se o elemento for uma div com vários campos preenchidos, retornará o texto de todos juntos. Então, é recomendável sempre utilizar o elemento mais específico possível.
Select: Este método é específico para campos do tipo dropdown. O valor passado por parâmetro será selecionado no dropdown.
```

```ruby
# features/step_definitions/account/account_steps.rb
account_page = AccountPage.new

When(/^I fill the account name with "([^"]*)"$/) do |name|
  account_page.account_name.set name
end

When(/^I click to save the account settings"$/) do
  account_page.save_button.click
end

When(/^I select the account language with "([^"]*)"$/) do |language|
  account_page.account_language.select language
end

Then(/^the account name is filled with "([^"]*)"$/) do |name|
  expect(account_page.account_name.text).to eq(name)
end
```

# Coleção de Elementos

As vezes não queremos utilizar um elemento individual, mas uma coleção de elementos semelhantes, por exemplo, uma lista de nomes. Para isto o SitePrism fornece o método elements, e para ilustrar utilizarei a listagem de contas externas que fica na página de contas do RD Station, conforme imagem abaxo:

Página de configurações de contas externas

```ruby
# features/pages/account/account_page.rb
class AccountPage < SitePrism::Page
  set_url "/accounts"

  element :account_name, '#account-name'
  element :save_button, "input[name='save-btn']"
  element :account_language, '.account-language'

  elements :external_accounts, 'li.list-group-item .accounts-title'
end
```

```ruby
# features/step_definitions/account/account_steps.rb
account_page = AccountPage.new

Then(/^I see the external accounts name$/) do
  account_page.external_accounts.first.text # => Google Analytics
  account_page.external_accounts.last.text # => Twitter
  account_page.external_accounts.each { |name| puts name.text } # => 'Google Analytics', 'Twitter', 'Facebook', 'LinkedIn'

  accounts_names = account_page.external_accounts.map { |name| name.text }
  expect(accounts_names).to eq ['Google Analytics', 'Twitter', 'Facebook', 'LinkedIn']
end
```

Matchers e waits de elementos
Para verificar se um elemento está presente na página, o SitePrism fornece o matcher have_<element_name>. Para o contrário, você pode utilizar o matcher have_no_<element_name>, conforme exemplo:

```ruby
# features/step_definitions/account/account_steps.rb
account_page = AccountPage.new

Then(/^the save button is visible$/) do
  expect(account_page).to have_save_button
end

Then(/^the save button is invisible$/) do
  expect(account_page).to have_no_save_button
end
```
Várias vezes nos deparamos com testes que falham na transição de páginas, pois o teste automatizado executa tão rápido que a página ainda não carregou completamente. Para resolver estes casos, o SitePrism fornece alguns métodos:

```ruby
wait_for_<element_name>: Método que aguarda um elemento estar presente na página;
wait_until_<element_name>_visible: Método que aguarda até o elemento ficar visível na página;
wait_until_<element_name>_invisible: Método que aguarda até o elemento ficar invisível na página;
```

```ruby
# features/step_definitions/account/account_steps.rb
account_page = AccountPage.new

When(/^I fill the account name with "([^"]*)"$/) do |name|
  account_page.wait_for_account_name
  account_page.account_name.set name
end

When(/^I fill the account name with "([^"]*)"$/) do |name|
  account_page.wait_until_account_name_visible
  account_page.account_name.set name
end

When(/^I click to save the account settings"$/) do
  account_page.save_button.click
  account_page.wait_until_save_button_invisible(10) # => é possível passar o timeout por parâmetro. Ex: 10s
end
```

Veja que, para os três métodos, é possível passar por parâmetro o tempo máximo para a espera do elemento.

# Sections

Quando nos deparamos com páginas complexas e que possuem vários elementos (seções, modais, listas, campos de texto, tabelas, botões, entre outros), a manutenção dos page objects se torna complexa, e compreender o que cada step está fazendo e em qual local da página aquele determinado elemento está não é tarefa fácil.

Para ajudar na organização do page object, uma boa prática é agrupar os elementos que estejam em um mesmo contexto em sections sempre que possível, pois isso deixa muito mais fácil a compreensão do que o código do teste está fazendo.

Para ilustrar esse cenário, criei um page object baseado na página de configurações de usuários da conta no RD Station, conforme imagem abaixo:

Página de configurações de usuários

Na imagem vemos que a página possui uma modal de edição de perfil do usuário além da listagem de usuários da conta. Primeiro vou mostrar como fazer o page mapeando os elementos direto no page object (inclusive os elementos da modal), e em seguida mostrarei como agrupar os elementos utilizando section.

```ruby
# features/pages/user/user_page.rb
class UserPage < SitePrism::Page
  set_url "configuracoes/usuarios"
  elements :edit_user_buttons, '.js-update-user-modal-link'
  element :marketing_option, '#role_name_marketing'
  element :sales_option, '#role_name_sales'
  element :financial_option, '#role_name_financial'
  element :admin_option, '#role_name_admin'
  element :update_button, "input[value='Atualizar']"
end
```

```ruby
# features/step_definitions/user/user_steps.rb
user_page = UserPage.new

When(/^I edit the first user to marketing profile"$/) do
  user_page.edit_user_buttons.first.click # => clica no botão Editar do primeiro usuário
  user_page.wait_until_marketing_option_visible # => Aguarda a opção Marketing aparecer
  user_page.marketing_option.click # => clica na opção marketing na modal
  user_page.update_button.click # => clica no botão Atualizar
  user_page.wait_until_update_button_invisible # => Aguarda o botão Atualizar desaparecer
end
```
Para agrupar elementos de um mesmo contexto em uma section, basta criar uma classe que herda os métodos de SitePrism::Section. Esta classe pode estar dentro da própria classe do page ou em um outro arquivo, desde que seja feito o require do arquivo contendo a classe dentro do page. Como a section do exemplo é um modal com poucos campos, a section será criada dentro da própria classe do page.

```ruby
# features/pages/user/user_page.rb
class UserPage < SitePrism::Page
  set_url "configuracoes/usuarios"
  elements :edit_user_buttons, '.js-update-user-modal-link'

  class EditUserModal < SitePrism::Section
    element :marketing_option, 'input#role_name_marketing'
    element :sales_option, 'input#role_name_sales'
    element :financial_option, 'input#role_name_financial'
    element :admin_option, 'input#role_name_admin'
    element :update_button, "input[value='Atualizar']"
  end

  section :edit_user_modal, EditUserModal, 'div#edit-role-modal'
end
```
```ruby
# features/step_definitions/user/user_steps.rb
user_page = UserPage.new

When(/^I edit the first user to marketing profile"$/) do
  user_page.edit_user_buttons.first.click # => clica no botão Editar do primeiro usuário (Elemento da Page)
  user_page.wait_until_edit_user_modal_visible # => Aguarda a modal de edição aparecer
  user_page.edit_user_modal.marketing_option.click # => clica na opção marketing na modal (Elemento da Section)
  user_page.edit_user_modal.update_button.click # => clica no botão Atualizar (Elemento da Section)
  user_page.wait_until_edit_user_modal_invisible # => Aguarda a modal de edição desaparecer
end
```

Já que neste exemplo o seletor da section edit_user_modal é a própria div do modal, as chamadas dos métodos de wait para aguardar o modal aparecer e desaparecer foram feitas utilizando a própria section (wait_until_edit_user_modal_visible e wait_until_edit_user_modal_invisible).

Criação de métodos dentro do page objects
No exemplo anterior, o nosso step não foi implementado de uma forma dinâmica que pode ser utilizada em outros steps. Se for preciso implementar o step que edita o perfil do usuário para vendedor, as cinco linhas vão ser duplicadas mudando apenas para selecionar a opção vendedor. Isso deixa nosso arquivo de steps inchado e com código duplicado.

Para resolver esse tipo de caso, podemos criar métodos dinâmicos dentro do arquivo de pages e utilizá-los nos steps, conforme exemplo abaixo:

```ruby
# features/pages/user/user_page.rb
class UserPage < SitePrism::Page
  set_url "configuracoes/usuarios"
  elements :edit_user_buttons, '.js-update-user-modal-link'

  class EditUserModal < SitePrism::Section
    element :marketing_option, 'input#role_name_marketing'
    element :sales_option, 'input#role_name_sales'
    element :financial_option, 'input#role_name_financial'
    element :admin_option, 'input#role_name_admin'
    element :update_button, "input[value='Atualizar']"
  end

  section :edit_user_modal, EditUserModal, 'div#edit-role-modal'

  def edit_user_profile(option)
    edit_user_buttons.first.click # => clica no botão Editar do primeiro usuário
    wait_until_edit_user_modal_visible # => Aguarda a modal de edição aparecer
    edit_user_modal.send(option).click # => clica na opção marketing na modal
    edit_user_modal.update_button.click # => clica no botão Atualizar
    wait_until_edit_user_modal_invisible # => Aguarda a modal de edição desaparecer
  end
end
```

As mesmas cinco linhas agora estão dentro do método edit_user_profile no page object. Note que nesse caso não é mais preciso utilizar a página instanciada user_page antes de cada ação, pois como o método está dentro do próprio page todos os elementos já estão disponíveis.

Obs: O método send() é nativo do Ruby e é utilizado para executar um método (passado por parâmetro) da classe que o chama. No exemplo, é passado como parâmetro o nome do elemento que queremos utilizar, conforme steps abaixo:

```ruby
#features/step_definitions/user/user_steps.rb
user_page = UserPage.new

When(/^I edit the first user to marketing profile"$/) do
  user_page.edit_user_profile 'marketing_option' # => O parâmetro pode ser o nome do elemento no page como string
end

When(/^I edit the first user to sales profile"$/) do
  user_page.edit_user_profile :sales_option # => O parâmetro pode ser o nome do elemento no page como símbolo
end
Note que o arquivo de steps fica bem mais enxuto e sem duplicação de código. Até nos dá a possibilidade de fazer um step único, passando a opção desejada por parâmetro.
```

# iFrames

Para manipular elementos dentro de um iframe na página, os passos são semelhantes a criação de sections. Basta criar uma classe que herda os métodos de SitePrism::Page (essa classe também pode ficar dentro da classe do page, ou em um arquivo externo). Depois de criada a classe, devemos criar um elemento do tipo iframe dentro do page, relacionando a classe que criamos com o seu respectivo seletor. Como exemplo, suponhamos que ao invés de uma modal, os elementos da edição de perfil do usuário estejam agora dentro de um iframe na página:

```ruby
# features/pages/user/user_page.rb
class UserPage < SitePrism::Page
  set_url "configuracoes/usuarios"
  elements :edit_user_buttons, '.js-update-user-modal-link'

  class EditUserIFrame < SitePrism::Page
    element :marketing_option, 'input#role_name_marketing'
    element :sales_option, 'input#role_name_sales'
    element :financial_option, 'input#role_name_financial'
    element :admin_option, 'input#role_name_admin'
    element :update_button, "input[value='Atualizar']"
  end

  iframe :edit_user_iframe, EditUserIFrame, 'iframe#edit-role-modal'

  def edit_user_profile(option)
    edit_user_buttons.first.click # => clica no botão Editar do primeiro usuário

    edit_user_iframe do |frame|
      frame.send(option).click # => clica na opção marketing no iframe
      frame.update_button.click # => clica no botão Atualizar no iframe
    end
  end
end
```

Note que para acessar os elementos do iframe devemos declará-lo em um bloco de código. Isso acontece porque o SitePrism muda o contexto do teste (foco) para o iframe. Quando o bloco do iframe acaba, o contexto do teste volta para a página.

#  Conclusão

Quando você tem um page object bem estruturado, criar os testes fica muito mais simples e fácil. Depois que conheci o SitePrism e toda a praticidade que ele proporciona, nunca mais utilizei Capybara “puro” nos meus testes automatizados e meu código ficou muito mais simples e claro.

Aqui na Resultados Digitais utilizamos o SitePrism para estruturar os page objects e isso ajuda muito na nossa produtividade no dia-a-dia.

E você, já automatiza criando page objects?