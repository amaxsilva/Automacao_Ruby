# Setando urls

Uma página geralmente tem um URL. Se você quiser navegar até uma página, precisará definir seu URL. Veja como:

```ruby
class Home < SitePrism::Page
  set_url 'http://www.google.com'
end
```

Se você tiver definido o app_host de Capybara, poderá definir o URL da seguinte maneira:

```ruby
class Home < SitePrism::Page
  set_url '/home.htm'
end
```

Tenha em atenção que a definição de um URL é opcional. Basta definir um URL para poder navegar diretamente para essa página. Faz sentido definir o URL para um modelo de página de uma página inicial ou uma página de login, mas provavelmente não é uma página de resultados de pesquisa.

# Verificar se uma página específica é exibida

```ruby
class Account < SitePrism::Page
  set_url 'www.google.com.br'
end
```

```ruby
@account_page = Account.new #(Você tbm pode usar o metodo criada no pagehelper.rb )
@account_page.load

expect(@account_page.current_url).to end_with('https://www.google.com.br')
expect(@account_page).to be_displayed

expect(@some_other_page).not_to be_displayed #(Valida se a outra página esta visivel)
```

A chamada #displayed?retornará verdadeiro se o URL atual do navegador corresponder ao modelo da página e falso se não corresponder. Ele vai esperar por Capybara.default_max_wait_timesegundos ou você pode passar um tempo de espera explícito em segundos como o primeiro argumento, como este:

```ruby 
@account_page.displayed?(10) # aguarde até 10 segundos pela exibição
```

# Título da página: 

Obter o título de uma página não é difícil:

```ruby
class Account < SitePrism::Page
end

@account = Account.new
@account.title #=> "Welcome to Your Account"
```

# HTTP x HTTPS

Você pode saber facilmente se a página é segura ou não, verificando se o URL atual começa com 'https' ou não. SitePrism fornece o secure?método que retornará verdadeiro se o url atual começar com 'https' e falso se não começar. Por exemplo:

```ruby
class Account < SitePrism::Page
end

@account = Account.new
@account.secure? #=> true/false
expect(@account).to be_secure
```

# Obtendo o URL da página atual

SitePrism permite que você obtenha o URL da página atual. Veja como é feito:

```ruby
class Account < SitePrism::Page
end

@account = Account.new
@account.current_url #=> "http://www.example.com/account/123"
expect(@account.current_url).to include('example.com/account/')
```
