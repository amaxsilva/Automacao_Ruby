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
# ELemento Select

```ruby
element :selec_city, 'select[id="id_city"]'
element :country, 'select[id="id_country"]'

country.select 'Manaus'
```