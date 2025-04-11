# Documentação da Feature: feature-documentacao-ordem-cronologica

## Ordenação Cronológica de Documentos

## Visão Geral
Esta feature permite exibir os documentos de um requerimento organizados e ordenados cronologicamente por data de envio (campo `ultima_modificacao`). 
A ordenação pode ser controlada pelo usuário por meio de um botão na interface da aplicação.
<<INSERIR IMAGEM EXIBINDO OS DOIS ESTAGIOS DO BOTAO>>
<<Ordenar mais recentes>>
<<Ordenar mais antigos>>
*![Ordenar mais recentes](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%202025-04-11%20163907.png)*
*![Ordenar mais antigos](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%202025-04-11%20163935.png)*
*![Botoes Ordenar](https://github.com/wfrsilva/sinfat/blob/main/feature-documentacao-ordem-cronologica%202025-04-11%20163518.gif)*

---

## Arquivos alterados
- Controller: `administrativo/src/main/java/ciga/sinfat/administrativo/controllers/DocumentosController.java`
- Template: `administrativo/src/main/resources/templates/documentos.pebble`

---

## 1. Alterações no DocumentosController.java

### Novas Variáveis

- `ordemDataParam`:
  - Origem: parâmetro da URL (?ordem=asc ou ?ordem=desc)
  - Finalidade: captura a preferência do usuário quanto à ordenação

- `direcaoSql`:
  - Valor derivado de `ordemDataParam`, convertido para o formato esperado pelo SQL (`ASC` ou `DESC`)
  - Usado para substituir o marcador `:ordem_sql` na query nativa

- `queryOrdenada`:
  - A `QUERY_DOCUMENTOS` com o marcador `:ordem_sql` substituído pela `direcaoSql`

### Trecho relevante do Controller
```java
final String ordemDataParam = Optional.ofNullable(request.getParameter("ordem")).orElse("desc");
String direcaoSql = "asc".equalsIgnoreCase(ordemDataParam)? "ASC" : "DESC";
final String queryOrdenada = QUERY_DOCUMENTOS.replace(":ordem_sql", direcaoSql);
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

### Exemplo de trecho no template:
```html
<a href="?ordem={% if ordemDataAtual  == 'asc' %}desc{% else %}asc{% endif %}">
  {% if ordemDataAtual  == 'asc' %}Mais recentes primeiro{% else %}Mais antigos primeiro{% endif %}
</a>
```

Este botão permite ao usuário alternar entre:
- Mais recentes primeiro (`desc`)
- Mais antigos primeiro (`asc`)

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