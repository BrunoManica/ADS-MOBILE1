# Aula 7: Tela de login e permissao de usuario

## 1. Objetivo da aula

Nesta aula, vamos colocar login no app.

Ate agora, qualquer pessoa que abrisse o aplicativo conseguia ver o cardapio, adicionar itens ao pedido, cadastrar produto e editar produto.

Agora o app vai ter dois tipos de usuario:

- `admin`: pode cadastrar e editar produtos.
- `atendente`: pode montar pedidos, mas nao pode alterar o cardapio.

O login ainda sera local. Os usuarios ficarao escritos no codigo, porque a comunicacao com API vai ficar para a proxima aula.

## 2. O que vamos fazer

Nesta aula, vamos criar:

```text
src/types/Usuario.ts
src/services/authService.ts
src/pages/TelaLogin/TelaLogin.tsx
```

E vamos alterar:

```text
src/App.tsx
src/pages/TabsPrincipal/TabsPrincipal.tsx
src/pages/TelaComes/TelaComes.tsx
```

No final, o app tera:

- tela de login;
- usuario salvo no `localStorage`;
- botao para sair;
- bloqueio das tabs para quem nao estiver logado;
- botoes de cadastro e edicao aparecendo apenas para admin.

## 3. Criar o tipo de usuario

Crie o arquivo:

```text
src/types/Usuario.ts
```

Coloque este codigo:

```ts
export type PerfilUsuario = 'admin' | 'atendente';

export type Usuario = {
  id: string;
  nome: string;
  email: string;
  senha: string;
  perfil: PerfilUsuario;
};

export type UsuarioLogado = {
  id: string;
  nome: string;
  email: string;
  perfil: PerfilUsuario;
};
```

O tipo `Usuario` tem a senha porque representa o usuario cadastrado no sistema.

O tipo `UsuarioLogado` nao tem senha. Ele sera usado para guardar quem entrou no app.

## 4. Criar o service de login

Crie o arquivo:

```text
src/services/authService.ts
```

Coloque este codigo:

```ts
import { Usuario, UsuarioLogado } from '../types/Usuario';

const CHAVE_USUARIO_LOGADO = 'usuarioLogado';

const usuarios: Usuario[] = [
  {
    id: '1',
    nome: 'Administrador',
    email: 'admin@email.com',
    senha: '123456',
    perfil: 'admin',
  },
  {
    id: '2',
    nome: 'Atendente',
    email: 'atendente@email.com',
    senha: '123456',
    perfil: 'atendente',
  },
];

export const fazerLogin = (
  email: string,
  senha: string,
): UsuarioLogado | null => {
  const usuarioEncontrado = usuarios.find((usuario) => {
    return usuario.email == email && usuario.senha == senha;
  });

  if (!usuarioEncontrado) {
    return null;
  }

  const usuarioLogado: UsuarioLogado = {
    id: usuarioEncontrado.id,
    nome: usuarioEncontrado.nome,
    email: usuarioEncontrado.email,
    perfil: usuarioEncontrado.perfil,
  };

  localStorage.setItem(CHAVE_USUARIO_LOGADO, JSON.stringify(usuarioLogado));

  return usuarioLogado;
};

export const buscarUsuarioLogado = (): UsuarioLogado | null => {
  const usuarioSalvo = localStorage.getItem(CHAVE_USUARIO_LOGADO);

  if (!usuarioSalvo) {
    return null;
  }

  return JSON.parse(usuarioSalvo);
};

export const usuarioEstaLogado = (): boolean => {
  const usuarioLogado = buscarUsuarioLogado();

  return usuarioLogado !== null;
};

export const usuarioEhAdmin = (): boolean => {
  const usuarioLogado = buscarUsuarioLogado();

  return usuarioLogado?.perfil == 'admin';
};

export const sair = () => {
  localStorage.removeItem(CHAVE_USUARIO_LOGADO);
};
```

Temos dois usuarios para testar:

```text
email: admin@email.com
senha: 123456
```

```text
email: atendente@email.com
senha: 123456
```

Quando o login da certo, salvamos no `localStorage` somente os dados necessarios. A senha nao e salva.

## 5. Criar a tela de login

Crie a pasta:

```text
src/pages/TelaLogin
```

Dentro dela, crie o arquivo:

```text
src/pages/TelaLogin/TelaLogin.tsx
```

Coloque este codigo:

```tsx
import {
  IonButton,
  IonContent,
  IonHeader,
  IonInput,
  IonItem,
  IonList,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/react';
import { useState } from 'react';
import { useHistory } from 'react-router';
import { fazerLogin } from '../../services/authService';
import '../Cardapio.css';

export const TelaLogin = () => {
  const history = useHistory();

  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');
  const [erro, setErro] = useState('');

  const entrar = () => {
    setErro('');

    if (email == '' || email == null) {
      setErro('O email esta com valor invalido');
      return;
    }

    if (senha == '' || senha == null) {
      setErro('A senha esta com valor invalido');
      return;
    }

    const usuarioLogado = fazerLogin(email, senha);

    if (!usuarioLogado) {
      setErro('Email ou senha invalidos');
      return;
    }

    history.push('/bebes');
  };

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Login</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent className="cardapio-content">
        <div className="cardapio-content">
          {erro !== '' && <p className="erro">{erro}</p>}

          <IonList>
            <IonItem>
              <IonInput
                label="Email"
                labelPlacement="stacked"
                value={email}
                placeholder="admin@email.com"
                onIonInput={(event) => setEmail(event.detail.value ?? '')}
              >
              </IonInput>
            </IonItem>

            <IonItem>
              <IonInput
                label="Senha"
                labelPlacement="stacked"
                type="password"
                value={senha}
                placeholder="123456"
                onIonInput={(event) => setSenha(event.detail.value ?? '')}
              >
              </IonInput>
            </IonItem>
          </IonList>

          <IonButton expand="block" onClick={entrar}>
            entrar
          </IonButton>
        </div>
      </IonContent>
    </IonPage>
  );
};
```

Rode o app e acesse:

```text
/login
```

Digite:

```text
admin@email.com
123456
```

Ao tocar em `entrar`, o app deve abrir a tela de bebidas.

## 6. Registrar a tela de login

Abra:

```text
src/App.tsx
```

Deixe o arquivo assim:

```tsx
import { Redirect, Route } from 'react-router-dom';
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import TabsPrincipal from './pages/TabsPrincipal/TabsPrincipal';
import { TelaLogin } from './pages/TelaLogin/TelaLogin';

import '@ionic/react/css/core.css';
import '@ionic/react/css/normalize.css';
import '@ionic/react/css/structure.css';
import '@ionic/react/css/typography.css';
import '@ionic/react/css/padding.css';
import '@ionic/react/css/float-elements.css';
import '@ionic/react/css/text-alignment.css';
import '@ionic/react/css/text-transformation.css';
import '@ionic/react/css/flex-utils.css';
import '@ionic/react/css/display.css';
import './theme/variables.css';

setupIonicReact();

const App = () => (
  <IonApp>
    <IonReactRouter>
      <IonRouterOutlet>
        <Route exact path="/login" component={TelaLogin}></Route>
        <Route
          path={[
            '/bebes',
            '/comes',
            '/pedido',
            '/produto-novo',
            '/produtos/editar/:id',
          ]}
          component={TabsPrincipal}
        >
        </Route>
        <Route exact path="/">
          <Redirect to="/bebes"></Redirect>
        </Route>
      </IonRouterOutlet>
    </IonReactRouter>
  </IonApp>
);

export default App;
```

A rota `/login` fica aqui no `App.tsx`.

As telas principais continuam dentro de `TabsPrincipal`.

## 7. Bloquear as tabs sem login

Abra:

```text
src/pages/TabsPrincipal/TabsPrincipal.tsx
```

Adicione estes imports:

```tsx
import { useEffect } from 'react';
import { Redirect, Route, useHistory } from 'react-router-dom';
import { sair, usuarioEstaLogado } from '../../services/authService';
```

Se o arquivo ja tinha `Redirect` e `Route`, apenas acrescente `useHistory` no mesmo import.

Agora, dentro do componente `TabsPrincipal`, antes do `return`, coloque:

```tsx
const history = useHistory();

useEffect(() => {
  if (!usuarioEstaLogado()) {
    history.replace('/login');
  }
}, [history]);

const deslogar = () => {
  sair();
  history.replace('/login');
};
```

O componente completo fica assim:

```tsx
import { useEffect } from 'react';
import { Redirect, Route, useHistory } from 'react-router-dom';
import {
  IonIcon,
  IonLabel,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
} from '@ionic/react';
import {
  cafeOutline,
  cartOutline,
  logOutOutline,
  restaurantOutline,
} from 'ionicons/icons';
import TelaBebes from '../TelaBebes/TelaBebes';
import TelaComes from '../TelaComes/TelaComes';
import TelaPedido from '../TelaPedido/TelaPedido';
import { TelaProdutoForm } from '../TelaProdutoForm/TelaProdutoForm';
import { sair, usuarioEstaLogado } from '../../services/authService';

const TabsPrincipal = () => {
  const history = useHistory();

  useEffect(() => {
    if (!usuarioEstaLogado()) {
      history.replace('/login');
    }
  }, [history]);

  const deslogar = () => {
    sair();
    history.replace('/login');
  };

  return (
    <IonTabs>
      <IonRouterOutlet>
        <Route exact path="/bebes" component={TelaBebes}></Route>
        <Route exact path="/comes" component={TelaComes}></Route>
        <Route exact path="/pedido" component={TelaPedido}></Route>
        <Route exact path="/produto-novo" component={TelaProdutoForm}></Route>
        <Route exact path="/produtos/editar/:id" component={TelaProdutoForm}></Route>
        <Route exact path="/">
          <Redirect to="/bebes"></Redirect>
        </Route>
      </IonRouterOutlet>

      <IonTabBar slot="bottom">
        <IonTabButton tab="bebes" href="/bebes">
          <IonIcon icon={cafeOutline} />
          <IonLabel>Bebes</IonLabel>
        </IonTabButton>

        <IonTabButton tab="comes" href="/comes">
          <IonIcon icon={restaurantOutline} />
          <IonLabel>Comes</IonLabel>
        </IonTabButton>

        <IonTabButton tab="pedido" href="/pedido">
          <IonIcon icon={cartOutline} />
          <IonLabel>Pedido</IonLabel>
        </IonTabButton>

        <IonTabButton tab="sair" onClick={deslogar}>
          <IonIcon icon={logOutOutline} />
          <IonLabel>Sair</IonLabel>
        </IonTabButton>
      </IonTabBar>
    </IonTabs>
  );
};

export default TabsPrincipal;
```

Agora teste direto no navegador:

```text
/comes
```

Se nao tiver usuario logado, o app deve abrir a tela de login.

Depois faca login e toque em `Sair`. O app deve voltar para o login.

## 8. Liberar cadastro e edicao apenas para admin

Abra:

```text
src/pages/TelaComes/TelaComes.tsx
```

Adicione o import:

```tsx
import { usuarioEhAdmin } from '../../services/authService';
```

Procure o botao de novo produto:

```tsx
<IonButton routerLink="/produto-novo">
  Adicionar NOVO produto
</IonButton>
```

Troque por:

```tsx
{usuarioEhAdmin() && (
  <IonButton routerLink="/produto-novo">
    Adicionar NOVO produto
  </IonButton>
)}
```

Agora procure o botao `Editar` dentro da lista:

```tsx
<IonButton routerLink={`/produtos/editar/${comida.id}`}>
  Editar
</IonButton>
```

Troque por:

```tsx
{usuarioEhAdmin() && (
  <IonButton routerLink={`/produtos/editar/${comida.id}`}>
    Editar
  </IonButton>
)}
```

Nao altere o botao `Adicionar`:

```tsx
<IonButton onClick={() => abrirModal(comida)}>Adicionar</IonButton>
```

Esse botao precisa continuar aparecendo para o atendente.

## 9. Teste final

Entre com o admin:

```text
email: admin@email.com
senha: 123456
```

Abra a tela `Comidas`.

O admin deve ver:

- `Adicionar NOVO produto`;
- `Adicionar`;
- `Editar`.

Toque em `Sair`.

Entre com o atendente:

```text
email: atendente@email.com
senha: 123456
```

Abra a tela `Comidas`.

O atendente deve ver:

- `Adicionar`.

O atendente nao deve ver:

- `Adicionar NOVO produto`;
- `Editar`.

## 10. Erros comuns

### 10.1 Login sempre volta para a tela inicial

Confira se o login esta salvando o usuario:

```ts
localStorage.setItem(CHAVE_USUARIO_LOGADO, JSON.stringify(usuarioLogado));
```

### 10.2 Senha aparecendo no `localStorage`

Confira se voce esta salvando `usuarioLogado`, e nao `usuarioEncontrado`.

O correto e:

```ts
localStorage.setItem(CHAVE_USUARIO_LOGADO, JSON.stringify(usuarioLogado));
```

### 10.3 Tela de login com tabs embaixo

Confira o `App.tsx`.

A rota `/login` deve ficar fora do `TabsPrincipal`.

### 10.4 Botao `Sair` nao funciona

Confira se o botao chama `deslogar`:

```tsx
<IonTabButton tab="sair" onClick={deslogar}>
```

E confira se a funcao remove o usuario:

```ts
const deslogar = () => {
  sair();
  history.replace('/login');
};
```

### 10.5 Atendente consegue editar produto

Confira se o botao `Editar` ficou dentro desta regra:

```tsx
{usuarioEhAdmin() && (
  <IonButton routerLink={`/produtos/editar/${comida.id}`}>
    Editar
  </IonButton>
)}
```

## 11. Resumo

Nesta aula, o app ganhou login local e controle simples de permissao.

O admin pode cadastrar e editar produtos.

O atendente pode navegar pelo cardapio e adicionar itens ao pedido.

Na proxima aula, vamos tirar os dados fixos do codigo e buscar as informacoes em uma API.
