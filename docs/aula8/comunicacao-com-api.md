# Aula 8: Comunicação com API de produtos

## 1. Objetivo da aula

Nesta aula, substituímos a lista local de produtos pela comunicação com uma API.

Até a Aula 7, o `produtoService` guardava comidas e bebidas em arrays dentro do próprio frontend. Na Aula 8, a lógica foi alterada para:

- consultar todos os produtos com `GET`;
- filtrar comidas e bebidas pela categoria;
- consultar um produto pelo `id`;
- cadastrar um produto com `POST`;
- alterar um produto com `PUT`;
- aguardar a consulta da API antes de atualizar a tela.

A interface visual não foi refeita nesta aula. O trabalho ficou concentrado na lógica.

## 2. Arquivos trabalhados

Os arquivos cuja lógica foi alterada foram:

```text
src/services/produtoService.ts
src/pages/TelaComes/TelaComes.tsx
src/pages/TelaProdutoForm/TelaProdutoForm.tsx
```

O tipo `Produto` usado pela API também possui o campo `categoria`:

```ts
export type Produto = {
  id: string;
  codBarras: string;
  ingredientes: string[] | null;
  ativo: boolean;
  nome: string;
  preco: number;
  imagem?: string;
  categoria: string;
};
```

!!! note
    A `TelaBebes.tsx` não foi alterada durante esta aula. A função `buscarBebidas` foi criada no service, mas a tela de bebidas continuou com o conteúdo que já possuía.

## 3. Endereço da API

No arquivo:

```text
src/services/produtoService.ts
```

foram criadas as constantes:

```ts
const API_BASE_URL = "https://6a2897974e1e783349a5acce.mockapi.io"
const PRODUTO_URL = `${API_BASE_URL}/produto`
```

Assim, todas as operações usam como endereço:

```text
https://6a2897974e1e783349a5acce.mockapi.io/produto
```

## 4. Buscar todos os produtos

Os arrays locais de comidas e bebidas deixaram de ser usados. No lugar deles, foi criada uma função assíncrona:

```ts
export const buscarTodosOsProdutos = async (): Promise<Produto[] | undefined> => {
  const response = await fetch(PRODUTO_URL);

  if (response.ok == false) {
    throw new Error("nao foi possivel consultar produtos")
  }

  const resposta = await response.json();
  return resposta.map((produto: Produto) => {
    return {
      id: produto?.id ?? 0,
      codBarras: produto?.codBarras ?? "",
      ingredientes: produto?.ingredientes ?? [],
      ativo: produto?.ativo ?? false,
      nome: produto?.nome ?? "",
      preco: produto?.preco ?? 0,
      imagem: produto.imagem ?? "",
      categoria: produto.categoria
    }
  })
};
```

O fluxo dessa função é:

```text
fetch na API
  -> verifica response.ok
  -> converte a resposta para JSON
  -> percorre o array com map
  -> devolve os produtos
```

O operador `??` define um valor padrão quando uma propriedade vier como `null` ou `undefined`.

Exemplo:

```ts
nome: produto?.nome ?? ""
```

Se a API não devolver um nome, será usada uma string vazia.

## 5. Filtrar comidas e bebidas

Depois de buscar todos os produtos, o service filtra o resultado usando a propriedade `categoria`.

### 5.1 Buscar comidas

```ts
export const buscarComidas = async (): Promise<Produto[] | undefined> => {
  const todosOsProdutos = await buscarTodosOsProdutos()
  console.log(todosOsProdutos, 'ola mundo cruel')
  return todosOsProdutos?.filter((produto) => produto.categoria === "comida");
}
```

### 5.2 Buscar bebidas

```ts
export const buscarBebidas = async (): Promise<Produto[] | undefined> => {
  const todosOsProdutos = await buscarTodosOsProdutos()
  return todosOsProdutos?.filter((produto) => produto.categoria === "bebidas");
}
```

As duas funções aguardam `buscarTodosOsProdutos` porque a consulta com `fetch` é assíncrona.

As categorias usadas no projeto desenvolvido em aula foram:

```text
comida
bebidas
```

## 6. Carregar as comidas na tela

No arquivo:

```text
src/pages/TelaComes/TelaComes.tsx
```

a chamada anterior era síncrona. Depois da comunicação com a API, `buscarComidas()` passou a devolver uma `Promise`.

Por isso, a lógica do `useIonViewWillEnter` ficou:

```tsx
useIonViewWillEnter(() => {
  //TODO COLOCAR UM LOADER
  const carregarProdutos = async () => {
    const comidasEncontradas = await buscarComidas();
    if (comidasEncontradas) {
      setProdutos(comidasEncontradas);
    }
  }
  carregarProdutos();
});
```

O restante da tela não foi alterado pela comunicação com a API.

O fluxo agora é:

```text
entrar na tela
  -> chamar carregarProdutos
  -> aguardar buscarComidas
  -> receber o array
  -> atualizar o estado produtos
  -> React renderizar a lista
```

## 7. Buscar um produto pelo id

Para editar um produto, o formulário precisa consultar a API usando o `id` recebido pela rota.

No `produtoService.ts`, a função ficou:

```ts
export const buscarProdutoPorId = async (id: string): Promise<Produto | null> => {
  const response = await fetch(`${PRODUTO_URL}/${id}`);
  //const response = await fetch(PRODUTO_URL+"/"+id);

  if (response.ok == false) {
    throw new Error("nao foi possivel consultar produto")
  }

  const resposta = await response.json();

  if (resposta.data == null) {
    throw new Error("nao foi possivel consultar produto")
  }

  const produto = resposta.data;
  return {
    id: produto?.id ?? 0,
    codBarras: produto?.codBarras ?? "",
    ingredientes: produto?.ingredientes ?? [],
    ativo: produto?.ativo ?? false,
    nome: produto?.nome ?? "",
    preco: produto?.preco ?? 0,
    imagem: produto.imagem ?? "",
    categoria: produto.categoria
  }
}
```

A URL consultada segue este formato:

```text
https://6a2897974e1e783349a5acce.mockapi.io/produto/ID
```

Exemplo:

```text
https://6a2897974e1e783349a5acce.mockapi.io/produto/1
```

No código desenvolvido em aula, a resposta da consulta individual é lida por:

```ts
const produto = resposta.data;
```

## 8. Carregar o produto no formulário

No arquivo:

```text
src/pages/TelaProdutoForm/TelaProdutoForm.tsx
```

o `useEffect` passou a criar uma função assíncrona e aguardar a consulta:

```tsx
useEffect(() => {
  if (!id) {
    return;
  }

  const carregarProdutoPorId = async () => {
    const produtoPorId = await buscarProdutoPorId(id)
    if (!produtoPorId) {
      return
    }

    setCodBarras(produtoPorId?.codBarras)
    setPreco(produtoPorId?.preco.toString())
    setNome(produtoPorId?.nome)
    setIngredientes(produtoPorId?.ingredientes?.join(', ') ?? "")
    setCategoria(produtoPorId?.categoria ?? "comida")
  }

  carregarProdutoPorId()
}, [id])
```

A principal mudança em relação à Aula 6 está nesta linha:

```ts
const produtoPorId = await buscarProdutoPorId(id)
```

Antes, o produto era encontrado imediatamente em um array local. Agora é necessário esperar a resposta da API.

## 9. Cadastrar um produto

A função de cadastro também passou a usar `fetch`.

```ts
export const cadastrarProduto = async (
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
  categoria: string
) => {
  const precoTratado = Number(preco.replace("R$", "").replace(",", "."))
  const novoProduto = {
    nome,
    ingredientes: ingredientes.split(","),
    preco: precoTratado,
    codBarras,
    ativo: true,
    categoria
  }

  console.log(novoProduto, "oi ")
  const response = await fetch(PRODUTO_URL, {
    method: "POST",
    headers: {
      "Content-type": "application-json",
    },
    body: (JSON.stringify(novoProduto))
  })

  if (response.ok == false) {
    throw new Error("nao foi possivel atualizar o produto")
  }
}
```

Antes de enviar:

- o preço digitado é convertido para número;
- os ingredientes são separados por vírgula;
- o produto recebe `ativo: true`;
- a categoria selecionada é enviada para a API.

O cadastro usa:

```text
POST /produto
```

## 10. Alterar um produto

Na Aula 6, `alterarProduto` modificava um objeto dentro do array local. Agora a função envia os novos dados para a API:

```ts
export const alterarProduto = async (
  id: string,
  nome: string,
  ingredientes: string,
  preco: string,
  codBarras: string,
  categoria: string
) => {
  const precoTratado = Number(preco.replace("R$", "").replace(",", "."))
  const novoProduto = {
    nome,
    ingredientes: ingredientes.split(","),
    preco: precoTratado,
    codBarras,
    ativo: true,
    categoria
  }

  const response = await fetch(`${PRODUTO_URL}/${id}`, {
    method: "PUT",
    headers: {
      "Content-type": "application-json",
    },
    body: (JSON.stringify(novoProduto))
  })

  if (response.ok == false) {
    throw new Error("nao foi possivel atualizar o produto")
  }
}
```

A alteração usa:

```text
PUT /produto/ID
```

## 11. Salvar no formulário

As validações e os campos da tela foram mantidos. A decisão entre cadastrar e alterar continua sendo feita com `estaEditando`:

```tsx
if (estaEditando) {
  alterarProduto(id, nome, ingredientes, preco, codBarras, categoria)
} else {
  cadastrarProduto(nome, ingredientes, preco, codBarras, categoria);
}

setCodBarras("")
setIngredientes("")
setNome("")
setPreco("")
setCategoria("comida")
history.push("/comes")
```

A diferença é que `cadastrarProduto` e `alterarProduto` agora executam requisições HTTP.

No código reproduzido em aula, `salvarProduto` continuou sem `async` e as duas funções foram chamadas sem `await`. Por isso, a navegação para `/comes` acontece imediatamente depois de iniciar a requisição.

## 12. Formato final do service

Ao terminar a aula, o `produtoService.ts` passou a fornecer:

```text
buscarTodosOsProdutos()
  -> GET /produto

buscarComidas()
  -> busca todos e filtra categoria "comida"

buscarBebidas()
  -> busca todos e filtra categoria "bebidas"

buscarProdutoPorId(id)
  -> GET /produto/id

cadastrarProduto(...)
  -> POST /produto

alterarProduto(...)
  -> PUT /produto/id
```

## 13. Pontos de atenção do código reproduzido

Para manter este material de acordo com o que foi desenvolvido em aula, os trechos anteriores reproduzem a lógica encontrada no projeto `react-burguer`.

Alguns pontos devem ser observados:

### 13.1 Formato da resposta por id

A função espera que a API devolva:

```json
{
  "data": {
    "id": "1",
    "nome": "Produto"
  }
}
```

Se a API devolver o produto diretamente, `resposta.data` será `undefined`.

### 13.2 Content-Type

O código reproduzido usa:

```ts
"Content-type": "application-json"
```

O tipo normalmente usado para enviar JSON é `application/json`. O material mantém `application-json` para permanecer fiel ao projeto da aula.

### 13.3 Navegação sem aguardar a requisição

Como cadastro e alteração são chamados sem `await`, a tela volta para `/comes` antes de confirmar se a API terminou a operação.

## 14. Resumo

Nesta aula, alteramos somente a lógica necessária para trocar os produtos locais pela API.

O projeto passou a:

- consultar produtos com `fetch`;
- trabalhar com funções assíncronas;
- usar `await` ao carregar a lista e o produto da edição;
- filtrar produtos por categoria;
- cadastrar com `POST`;
- alterar com `PUT`;
- validar falhas usando `response.ok`.

A estrutura visual das telas permaneceu a mesma.
