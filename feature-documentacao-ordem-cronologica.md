# Ordenação Cronológica de Documentos
## Branch: feature-documentacao-ordem-cronologica

## Introdução
Esta documentação explica a implementação da ordenação cronológica dos documentos de um requerimento, incluindo os pontos de integração no back-end (controller) e no front-end (template).


## Visão Geral
Esta feature permite exibir os documentos de um requerimento organizados e ordenados cronologicamente por data de envio (campo `ultima_modificacao`). 
A ordenação pode ser controlada pelo usuário por meio de um botão na interface da aplicação.
<<INSERIR IMAGEM EXIBINDO OS DOIS ESTAGIOS DO BOTAO>>
<<Ordenar mais recentes>>
<<Ordenar mais antigos>>

## Comportamento na Interface
O usuário do SINFAT Administrativo pode alternar a ordenação clicando em um botão:
![Ordenar mais recentes](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%202025-04-11%20163907.png) 
  - mostra os documentos mais novos primeiro.
    
![Ordenar mais antigos](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%202025-04-11%20163935.png)
  - mostra os documentos mais antigos primeiro.

    
![Botoes Ordenar](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%202025-04-11%20163518.gif) 

---

## Arquivos alterados
- Back-end:
  - Controller: `administrativo/src/main/java/ciga/sinfat/administrativo/controllers/DocumentosController.java`
- front-end:
  - Template: `administrativo/src/main/resources/templates/documentos.pebble`

---

## 1. Alterações no DocumentosController.java

### Como funciona

O controller captura o parâmetro de ordenação (`?ordem=asc|desc`) e injeta essa escolha na SQL usando substituição:
```java
final String ordemDataParam = Optional.ofNullable(request.getParameter("ordem")).orElse("desc");
String direcaoSql = "asc".equalsIgnoreCase(ordemDataParam)? "ASC" : "DESC";
final String queryOrdenada = QUERY_DOCUMENTOS.replace(":ordem_sql", direcaoSql);
```
Isso substitui dinamicamente:
```sql
ORDER BY ultima_modificacao :ordem_sql
```
Por:
```sql
ORDER BY ultima_modificacao ASC
```
Se `.../documentos/{fecCodigo}?ordem=asc` , então `ordemDataParam = "asc"`, entao `direcaoSql = "ASC"`.


OU

Por:
```sql
ORDER BY ultima_modificacao DESC
```
Se `.../documentos/{fecCodigo}?ordem=desc` , então `ordemDataParam = "desc"`, entao `direcaoSql = "DESC"`.

<< < < INSERIR DIAGRAMA DE ATIVIDADES > > >>

A variável `direcaoSql` é então enviada ao modelo:

```java
model.put("ordemDataAtual", direcaoSql.toLowerCase());
```

## 2. Lógica no Front-end `documentos.pebble`

O template utiliza a variável `ordemDataAtual` para:

- Exibir o botão de alternância de ordenação

- Gerar o próximo link com base no estado atual (asc ou desc)


# PAREI AQUI AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

### Novas Variáveis

- `ordemDataParam`:
  - Origem: parâmetro da URL (?ordem=asc ou ?ordem=desc)
  - Finalidade: captura a preferência do usuário quanto à ordenação

- `direcaoSql`:
  - Valor derivado de `ordemDataParam`, convertido para o formato esperado pelo SQL (`ASC` ou `DESC`)
  - Usado para substituir o marcador `:ordem_sql` na query nativa
 

- `queryOrdenada`:
  - A `QUERY_DOCUMENTOS` com o marcador `:ordem_sql` substituído pela `direcaoSql`
  - `+ "    ORDER BY ultima_modificacao :ordem_sql\n"`


### Trecho relevante do Controller
```java
final String ordemDataParam = Optional.ofNullable(request.getParameter("ordem")).orElse("desc");
String direcaoSql = "asc".equalsIgnoreCase(ordemDataParam)? "ASC" : "DESC";
final String queryOrdenada = QUERY_DOCUMENTOS.replace(":ordem_sql", direcaoSql);
```

#### Troca ordem_sql por ASC ou DESC 
```java
(...)
final String queryOrdenada = QUERY_DOCUMENTOS.replace(":ordem_sql", direcaoSql);
(...)
+ "    ORDER BY ultima_modificacao :ordem_sql\n"
(...)
```

### Resultado no modelo da view
```java
model.put("ordemDataAtual ", direcaoSql.toLowerCase());
```

---

## 2. Efeito no documentos.pebble

O template utiliza a variável `ordemDataAtual ` para:
- Exibir o botão de alternância de ordenação
- Gerar o próximo link com base no estado atual (asc ou desc)

### Exemplo de trecho no template (documentos.pebble):
```html

{% if ordemDataAtual == "asc" %}
    <a href="?ordem=desc" class="btn btn-outline-secondary" ><i class="bi bi-sort-down"></i> Ordenar mais recentes</a>
{% else %}
    <a href="?ordem=asc" class="btn btn-outline-secondary"  ><i class="bi bi-sort-up"></i> Ordenar mais antigos</a>
{% endif %}

```

Este botão permite ao usuário alternar entre:
- Mais recentes primeiro (`desc`)
    - ![Ordenar mais recentes](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%20botao-recentes.png)

- Mais antigos primeiro (`asc`)
     - ![Ordenar mais antigos](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%20botao-antigos.png)

---

## Considerações Técnicas

- O SQL usa duas vezes `ORDER BY ultima_modificacao :ordem_sql`, tanto na CTE principal quanto no JSON_AGG, garantindo que:
  - Os documentos dentro de cada grupo estejam ordenados corretamente
  - A consulta geral já venha pré-ordenada

- A variável `ordemDataAtual` é a única ponte entre controller e view no que diz respeito ao controle de ordenação.


---

## Futuras melhorias sugeridas
- Suporte a ordenação por outros critérios (ex: tipo de documento, status)
- Tornar a ordenação persistente entre acessos (ex: via sessionStorage ou cookie)
