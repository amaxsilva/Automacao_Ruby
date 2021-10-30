# Metodos

```ruby
Set: É utilizado para inserir texto no elemento.
Click: Efetua a ação de clique no elemento.
Text: Retorna o texto do elemento. Se o elemento for uma div com vários campos preenchidos, retornará o texto de todos juntos. Então, é recomendável sempre utilizar o elemento mais específico possível.
Select: Este método é específico para campos do tipo dropdown. O valor passado por parâmetro será selecionado no dropdown.
```

#  Elemento ID

```ruby
element :elemento_id, '#inserir_element_ID'
element :login, '#email'

login.set 'amax.kerickson@gmail.com'
```
# Elemento Classe

```ruby
element :elemento_class, '.inserir_element_class'
element :first_name, '.last_name'

first_name.set 'Amax'
```

# Elemento Type Button

```ruby
element :elemento_button, 'button[type="submit"]'
element :search_button, 'button[name="btnK"]'

search_button.click
```
# Elemento por Tag

```ruby
element :elemento_tag, 'input[name="q"]'
element :images, 'a.image-search'

elemento_tag.set 'Git Amax kerickson'
```
# Elemento Select

```ruby
element :selec_city, 'select[id="id_city"]'
element :country, 'select[id="id_country"]'

country.select 'Amazonas'
selec_city.select 'Manaus'
```

# Elemento por Xpath

```ruby
element :elemento_xpath, :xpath, '//*[text()="Inserir o texto desejado"]'
element :create, :xpath, '//a[type()="SubmitCreate"]'
element :first_name, :xpath, '//div[@id="signup"]//input[@name="first-name"]'

elemento_xpath.click
create.click
first_name.set 'Clark Kent'
```

# Elemento por CSS

```ruby
element :first_name, 'div#signup input[name="first-name"]'
```

# Esperando que os elementos fiquem visíveis ou invisíveis

Como um elemento individual, chamar o elementsmétodo criará dois métodos: wait_until_<elements_name>_visiblee wait_until_<elements_name>_invisible. Chamar esses métodos fará com que seu teste espere que os elementos se tornem visíveis ou invisíveis. Usando o exemplo acima:

```ruby
class Friends < SitePrism::Page
  elements :names, 'ul#names li a'
end

@friends_page = Friends.new

@friends_page.wait_until_names_visible
# and...
@friends_page.wait_until_names_invisible
É possível esperar por um determinado período de tempo em vez de usar o tempo de espera padrão da Capivara:

@friends_page.wait_until_names_visible(wait: 5)
# and...
@friends_page.wait_until_names_invisible(wait: 7)
```