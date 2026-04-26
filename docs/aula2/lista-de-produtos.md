# Aula 2: Carregando listas de comidas e bebidas

## Índice

- [1. Objetivo da aula](#1-objetivo-da-aula)
- [2. Resultado final](#2-resultado-final)
- [3. Contexto](#3-contexto)
- [4. Explicação conceitual](#4-explicacao-conceitual)
- [5. Setup inicial](#5-setup-inicial)
- [6. Passo a passo](#6-passo-a-passo)
- [7. Código completo](#7-codigo-completo)
- [8. Erros comuns](#8-erros-comuns)
- [9. Resumo](#9-resumo)
- [10. Próximo passo](#10-proximo-passo)

## 1. Objetivo da aula

Nesta aula, você vai transformar as telas `Comes` e `Bebes` em telas que exibem listas de produtos.

Ao final, o app terá:

- Um tipo `Produto` para representar os dados de comidas e bebidas.
- Um arquivo separado para guardar esse tipo.
- Funções de service para listar comidas e bebidas.
- Uma renderização dinâmica usando `.map()`.
- Nome, descrição e preço sendo exibidos para cada produto.

As telas deixam de mostrar apenas `Comida 1` e `Bebida 1` e passam a trabalhar com dados estruturados.

## 2. Resultado final

Ao acessar a rota de comidas:

```text
/comes
```

A tela vai exibir uma lista parecida com esta:

```text
Tela de Comes

Torrada
Torrada tradicional da casa
R$ 20,00

Xis Salada
Pão, hambúrguer, queijo, alface, tomate e maionese
R$ 28,00

Batata Frita
Porção média de batata frita
R$ 18,00
```

Ao acessar a rota de bebidas:

```text
/bebes
```

A tela vai exibir uma lista parecida com esta:

```text
Tela de Bebes

Refrigerante
Lata 350ml
R$ 6,00

Suco Natural
Suco natural da fruta
R$ 9,00

Água Mineral
Garrafa 500ml
R$ 4,00
```

O importante aqui não é a aparência final. O foco é entender como uma lista de dados vira conteúdo na tela.

## 3. Contexto

Um app de comanda precisa mostrar produtos disponíveis para venda.

No mundo real, esses produtos normalmente vêm de uma API ou de um banco de dados. Mas antes de buscar dados externos, é melhor começar com uma lista fixa no próprio código.

Isso ajuda a entender três coisas importantes:

- Como representar um produto.
- Como guardar comidas e bebidas em listas.
- Como transformar essas listas em elementos visuais na tela.

Depois que isso estiver claro, buscar produtos de uma API fica muito mais simples.

## 4. Explicação conceitual

Antes de mexer nas telas, precisamos organizar os dados do cardápio.

Uma comida e uma bebida têm praticamente o mesmo formato para o app: nome, descrição e preço. Por isso, vamos criar um tipo chamado `Produto`. Ele serve como o “formato combinado” que todo item do cardápio deve seguir.

Depois disso, vamos colocar as listas em um arquivo de service. A tela não precisa saber se os dados vieram de um array local, de um arquivo ou de uma API. Ela só precisa chamar uma função e exibir.

Com os dados chegando na tela, usamos `.map()` para transformar cada produto em um item visual do Ionic.

## 5. Setup inicial

Esta aula continua a partir da Aula 1.

Os arquivos usados nesta aula são:

```text
src/types/produto.ts
src/services/produtoService.ts
src/pages/TelaComes/TelaComes.tsx
src/pages/TelaBebes/TelaBebes.tsx
```

Para rodar o projeto:

No Linux, entre na pasta do projeto com um caminho parecido com:

```bash
cd /caminho/para/a/pasta-do-projeto
```

No Windows, entre na pasta do projeto com um caminho parecido com:

```bash
cd C:\caminho\para\a\pasta-do-projeto
```

Depois execute:

```bash
ionic serve
```

Depois, acesse a tela de comidas:

```text
http://localhost:8100/comes
```

E também a tela de bebidas:

```text
http://localhost:8100/bebes
```

## 6. Passo a passo

### 6.1 Começar pelo caminho dos dados

Vamos começar pelo que vem antes da tela: o formato dos dados.

Primeiro criamos o `Produto`, porque comidas e bebidas precisam seguir o mesmo padrão. Em seguida criamos funções de service para entregar esses dados para o app. Só depois disso entramos em `TelaComes` e `TelaBebes`.

Assim fica mais fácil entender o papel de cada arquivo: tipo define o formato, service fornece os dados, tela exibe o resultado.

### 6.2 Criar a pasta de tipos

Dentro de `src`, crie a pasta:

```text
src/types
```

Essa pasta guarda tipos que podem ser usados em várias partes do projeto.

Por que fazer isso?

- A tela fica responsável por renderizar interface.
- O tipo fica responsável por descrever o formato dos dados.
- Outros arquivos podem reutilizar `Produto` depois, sem copiar e colar código.

### 6.3 Criar o tipo `Produto`

Dentro de `src/types`, crie o arquivo:

```text
src/types/produto.ts
```

Antes de escrever o código, veja os tipos usados:

- `number`: número, usado em `id` e `preco`.
- `string`: texto, usado em `nome` e `descricao`.
- `type`: cria um formato próprio para um objeto.

Depois escreva:

```ts
export type Produto = {
  id: number;
  nome: string;
  descricao: string;
  preco: number;
};
```

O que esse código faz:

- Cria um formato chamado `Produto`.
- Define quais campos todo produto precisa ter.
- Ajuda o editor a sugerir propriedades como `produto.nome` e `produto.preco`.
- Evita erros simples, como escrever `produto.valor` quando o campo correto é `produto.preco`.
- Usa `export` para permitir que esse tipo seja importado em outros arquivos.

### 6.4 Criar a pasta de services

Dentro de `src`, crie a pasta:

```text
src/services
```

Essa pasta guarda arquivos responsáveis por fornecer dados para o app.

Nesta aula, o service ainda vai usar arrays locais. Mesmo assim, essa separação já deixa o projeto preparado para trocar os arrays por uma chamada de API depois.

### 6.5 Criar o arquivo de service

Dentro de `src/services`, crie o arquivo:

```text
src/services/produtoService.ts
```

No topo do arquivo, importe o tipo `Produto`:

```ts
import type { Produto } from '../types/produto';
```

O `import type` deixa claro que esse import existe apenas para tipagem. Ele não representa um componente visual nem uma função que roda no navegador.

### 6.6 Criar as listas dentro do service

No mesmo arquivo `produtoService.ts`, crie a lista de comidas:

```ts
const comidas: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    descricao: 'Torrada tradicional da casa',
    preco: 20,
  },
  {
    id: 2,
    nome: 'Xis Salada',
    descricao: 'Pão, hambúrguer, queijo, alface, tomate e maionese',
    preco: 28,
  },
  {
    id: 3,
    nome: 'Batata Frita',
    descricao: 'Porção média de batata frita',
    preco: 18,
  },
];
```

Depois crie a lista de bebidas:

```ts
const bebidas: Produto[] = [
  {
    id: 1,
    nome: 'Refrigerante',
    descricao: 'Lata 350ml',
    preco: 6,
  },
  {
    id: 2,
    nome: 'Suco Natural',
    descricao: 'Suco natural da fruta',
    preco: 9,
  },
  {
    id: 3,
    nome: 'Água Mineral',
    descricao: 'Garrafa 500ml',
    preco: 4,
  },
];
```

O trecho `Produto[]` significa:

```text
uma lista de Produto
```

Então, cada item dentro do array precisa respeitar o formato definido no tipo `Produto`.

### 6.7 Exportar as funções do service

Ainda em `produtoService.ts`, exporte uma função para cada lista:

```ts
export const listarComidas = () => {
  return comidas;
};

export const listarBebidas = () => {
  return bebidas;
};
```

O que esse código faz:

- `listarComidas` entrega a lista de comidas.
- `listarBebidas` entrega a lista de bebidas.
- As telas não precisam saber onde os dados estão guardados.
- Mais tarde, essas funções podem trocar o array por uma chamada `fetch`.

### 6.8 Usar o service na tela `TelaComes`

Volte para:

```text
src/pages/TelaComes/TelaComes.tsx
```

Abaixo dos imports do Ionic, importe o service:

```tsx
import { listarComidas } from '../../services/produtoService';
```

Depois busque as comidas:

```tsx
const comidas = listarComidas();
```

Neste momento, a chamada ainda é local. Mesmo assim, a tela já passa a depender de uma função do service, não de um array escrito dentro dela.

### 6.9 Criar uma função para formatar o preço

O preço está salvo como número:

```tsx
preco: 20
```

Mas na tela queremos exibir como moeda:

```text
R$ 20,00
```

Para isso, crie a função:

```tsx
const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};
```

O que acontece aqui:

- `preco` entra como número.
- `toLocaleString` formata esse número de acordo com o padrão brasileiro.
- `style: 'currency'` indica que queremos formato de moeda.
- `currency: 'BRL'` indica Real brasileiro.

Essa função evita repetir a mesma lógica em vários lugares.

### 6.10 Renderizar a lista de comidas com `.map()`

Dentro do `IonList`, troque o item fixo por:

```tsx
{comidas.map((produto) => (
  <IonItem key={produto.id}>
    <IonLabel>
      <h2>{produto.nome}</h2>
      <p>{produto.descricao}</p>
      <p>{formatarPreco(produto.preco)}</p>
    </IonLabel>
  </IonItem>
))}
```

O que cada parte faz:

- `comidas.map(...)`: percorre todos os produtos da lista de comidas.
- `(produto) => (...)`: recebe um produto por vez.
- `IonItem`: cria uma linha visual para o produto.
- `key={produto.id}`: identifica cada item para o React.
- `{produto.nome}`: mostra o nome.
- `{produto.descricao}`: mostra a descrição.
- `{formatarPreco(produto.preco)}`: mostra o preço formatado.

### 6.11 Usar o service na tela `TelaBebes`

Agora abra:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Importe o mesmo service:

```tsx
import { listarBebidas } from '../../services/produtoService';
```

Depois busque as bebidas:

```tsx
const bebidas = listarBebidas();
```

Repare que `Produto` serve para comida e bebida. O tipo fica no arquivo `produto.ts`, os dados ficam no service e as telas apenas renderizam.

### 6.12 Renderizar a lista de bebidas com `.map()`

Na `TelaBebes`, troque o item fixo:

```tsx
<IonItem>
  <IonLabel>Bebida 1</IonLabel>
</IonItem>
```

Por:

```tsx
{bebidas.map((produto) => (
  <IonItem key={produto.id}>
    <IonLabel>
      <h2>{produto.nome}</h2>
      <p>{produto.descricao}</p>
      <p>{formatarPreco(produto.preco)}</p>
    </IonLabel>
  </IonItem>
))}
```

A lógica é a mesma da tela `Comes`:

- `bebidas.map(...)`: percorre a lista de bebidas.
- `produto`: representa uma bebida por vez.
- `IonItem`: cria uma linha para cada bebida.
- `formatarPreco(produto.preco)`: exibe o preço em Real.

### 6.13 Entender o fluxo completo

O fluxo da tela fica assim:

```text
src/types/produto.ts
    define o formato dos dados

produtoService.ts
    guarda as listas e expõe funções para buscar dados

TelaComes.tsx
    chama listarComidas()

comidas.map()
    transforma cada comida em um IonItem

TelaBebes.tsx
    chama listarBebidas()

bebidas.map()
    transforma cada bebida em um IonItem
```

Essa é uma base muito comum em React: criar uma lista de dados e transformar essa lista em interface.

## 7. Código completo

Arquivo:

```text
src/types/produto.ts
```

Código completo:

```ts
export type Produto = {
  id: number;
  nome: string;
  descricao: string;
  preco: number;
};
```

Arquivo:

```text
src/services/produtoService.ts
```

Código completo:

```ts
import type { Produto } from '../types/produto';

const comidas: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    descricao: 'Torrada tradicional da casa',
    preco: 20,
  },
  {
    id: 2,
    nome: 'Xis Salada',
    descricao: 'Pão, hambúrguer, queijo, alface, tomate e maionese',
    preco: 28,
  },
  {
    id: 3,
    nome: 'Batata Frita',
    descricao: 'Porção média de batata frita',
    preco: 18,
  },
];

const bebidas: Produto[] = [
  {
    id: 1,
    nome: 'Refrigerante',
    descricao: 'Lata 350ml',
    preco: 6,
  },
  {
    id: 2,
    nome: 'Suco Natural',
    descricao: 'Suco natural da fruta',
    preco: 9,
  },
  {
    id: 3,
    nome: 'Água Mineral',
    descricao: 'Garrafa 500ml',
    preco: 4,
  },
];

export const listarComidas = () => {
  return comidas;
};

export const listarBebidas = () => {
  return bebidas;
};
```

Arquivo:

```text
src/pages/TelaComes/TelaComes.tsx
```

Código completo:

```tsx
import {
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';

import { listarComidas } from '../../services/produtoService';

const comidas = listarComidas();

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaComes = () => {
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Tela de Comes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          {comidas.map((produto) => (
            <IonItem key={produto.id}>
              <IonLabel>
                <h2>{produto.nome}</h2>
                <p>{produto.descricao}</p>
                <p>{formatarPreco(produto.preco)}</p>
              </IonLabel>
            </IonItem>
          ))}
        </IonList>
      </IonContent>
    </IonPage>
  );
};

export default TelaComes;
```

Arquivo:

```text
src/pages/TelaBebes/TelaBebes.tsx
```

Código completo:

```tsx
import {
  IonContent,
  IonHeader,
  IonItem,
  IonLabel,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';

import { listarBebidas } from '../../services/produtoService';

const bebidas = listarBebidas();

const formatarPreco = (preco: number) => {
  return preco.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  });
};

const TelaBebes = () => {
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Tela de Bebes</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonList>
          {bebidas.map((produto) => (
            <IonItem key={produto.id}>
              <IonLabel>
                <h2>{produto.nome}</h2>
                <p>{produto.descricao}</p>
                <p>{formatarPreco(produto.preco)}</p>
              </IonLabel>
            </IonItem>
          ))}
        </IonList>
      </IonContent>
    </IonPage>
  );
};

export default TelaBebes;
```

## 8. Erros comuns

### 8.1 Esquecer a `key`

Errado:

```tsx
{produtos.map((produto) => (
  <IonItem>
    <IonLabel>{produto.nome}</IonLabel>
  </IonItem>
))}
```

Correto:

```tsx
{produtos.map((produto) => (
  <IonItem key={produto.id}>
    <IonLabel>{produto.nome}</IonLabel>
  </IonItem>
))}
```

Sem `key`, o React costuma mostrar um aviso no console.

### 8.2 Criar produto fora do formato definido

Errado:

```tsx
const produtos: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    valor: 20,
  },
];
```

O campo `valor` não existe no tipo `Produto`. O campo correto é `preco`.

Correto:

```tsx
const produtos: Produto[] = [
  {
    id: 1,
    nome: 'Torrada',
    descricao: 'Torrada tradicional da casa',
    preco: 20,
  },
];
```

### 8.3 Salvar preço como texto

Evite:

```tsx
preco: 'R$ 20,00'
```

Prefira:

```tsx
preco: 20
```

O preço deve ficar como número para permitir cálculos depois. A formatação para `R$ 20,00` deve acontecer apenas na hora de exibir.

### 8.4 Esquecer as chaves ao escrever JavaScript dentro do JSX

Errado:

```tsx
<h2>produto.nome</h2>
```

Isso mostra o texto literal `produto.nome`.

Correto:

```tsx
<h2>{produto.nome}</h2>
```

As chaves `{}` permitem escrever JavaScript dentro do JSX.

### 8.5 Esquecer de exportar ou importar o tipo

Errado em `src/types/produto.ts`:

```ts
type Produto = {
  id: number;
  nome: string;
  descricao: string;
  preco: number;
};
```

Sem `export`, outro arquivo não consegue usar esse tipo.

Correto:

```ts
export type Produto = {
  id: number;
  nome: string;
  descricao: string;
  preco: number;
};
```

Errado em `produtoService.ts`:

```ts
import { Produto } from '../types/produto';
```

Para tipos, prefira:

```ts
import type { Produto } from '../types/produto';
```

O `import type` deixa o código mais explícito: esse import existe só para o TypeScript entender o formato dos dados.

## 9. Resumo

Nesta aula, as telas `Comes` e `Bebes` passaram a trabalhar com listas de produtos.

O ponto principal é entender que:

- `src/types/produto.ts` guarda o tipo `Produto`.
- `export type Produto` permite reutilizar o tipo em outros arquivos.
- `src/services/produtoService.ts` concentra as listas de comidas e bebidas.
- `import type` importa um tipo sem misturar com código visual.
- `Produto[]` representa uma lista de produtos.
- `.map()` transforma cada produto em um item visual.
- `key` ajuda o React a controlar listas.
- Preço deve ser armazenado como número e formatado apenas para exibição.

Com isso, o app começa a sair de telas fixas e passa a representar dados reais de uma comanda.

## 10. Próximo passo

Na próxima aula, o projeto pode evoluir melhorando o visual do cardápio com CSS.

As listas já estão funcionando. Agora faz sentido cuidar da aparência: espaçamento, cards, título, fundo da tela e destaque para o preço.
