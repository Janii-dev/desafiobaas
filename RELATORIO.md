## BUG #01 — Erro no login

### O que estava acontecendo

Quando o usuário tentava fazer login com uma senha incorreta, o sistema não mostrava corretamente uma mensagem informando que as credenciais estavam erradas.

### Por que acontecia

O problema estava no tratamento do erro da autenticação do Firebase. O código não verificava corretamente os erros retornados pelo Firebase Authentication, como `invalid-credential`, `wrong-password` e `user-not-found`.

### Como corrigi

Foi corrigido o bloco `catch` do login para identificar os diferentes tipos de erro e mostrar uma mensagem adequada ao usuário.

**Antes:**

```ts
} catch (err) {
  // tratamento incorreto do erro
}
```

**Depois:**

```ts
} catch (err) {
  const msg = err instanceof Error ? err.message : "Erro desconhecido";

  if (msg.includes("invalid-credential") || msg.includes("wrong-password")) {
    setErro("E-mail ou senha incorretos.");
  } else if (msg.includes("user-not-found")) {
    setErro("Nenhuma conta encontrada com este e-mail.");
  } else {
    setErro("Erro ao entrar. Tente novamente.");
  }
}
```

Depois da correção, o sistema passou a tratar os erros do Firebase Authentication corretamente.

Quando o usuário informa e-mail ou senha incorretos, o sistema mostra:

**"E-mail ou senha incorretos."**

Quando não existe uma conta com o e-mail informado, o sistema mostra:

**"Nenhuma conta encontrada com este e-mail."**



## BUG #02 — Middleware com condição invertida

### O que estava acontecendo

O middleware estava com a condição de autenticação invertida. Isso fazia com que usuários autenticados pudessem ser redirecionados para a tela de login, enquanto a proteção da rota não funcionava corretamente para usuários que não estavam logados.

### Por que acontecia

O problema estava na condição `if` do arquivo `middleware.ts`.

O código verificava se o `token` existia:

**Antes:**

```ts
if (token) {
  return NextResponse.redirect(new URL("/login", request.url));
}
```

Dessa forma, quando o usuário estava autenticado e possuía um token, ele era redirecionado para a página de login.

### Como corrigi

A condição foi alterada para verificar quando o usuário **não possui** um token.

**Depois:**

```ts
if (!token) {
  return NextResponse.redirect(new URL("/login", request.url));
}
```

O operador `!` significa negação. Portanto, a condição agora verifica corretamente se o usuário não está autenticado.

### Screenshot ou resultado

Não foram registrados screenshots durante a execução do bug.

Após a correção, o middleware passou a proteger corretamente as rotas. Usuários que não estão autenticados são direcionados para `/login`, enquanto usuários autenticados podem acessar as páginas protegidas.



## BUG #03 — Confirmação de senha compara com nome

### O que estava acontecendo

Ao realizar o cadastro, a validação da confirmação de senha estava sendo feita de forma incorreta. O sistema comparava a senha digitada com o nome do usuário em vez de comparar a senha com o campo de confirmação de senha.

### Por que acontecia

O problema estava na condição de validação do formulário de cadastro. A variável `nome` foi utilizada por engano no lugar da variável `confirmarSenha`.

**Código com o erro:**

```ts
if (senha !== nome) {
```

### Como corrigi

A variável utilizada na comparação foi corrigida para `confirmarSenha`.

**Código corrigido:**

```ts
if (senha !== confirmarSenha) {
```

Dessa forma, o sistema passa a verificar corretamente se a senha e a confirmação de senha são iguais.

Após a correção, o formulário passou a comparar corretamente os campos de senha e confirmação de senha. Caso os valores sejam diferentes, o cadastro é impedido e o usuário recebe a validação correspondente.



## BUG #04 — Query sem filtro de userId
O que estava acontecendo

A busca dos personagens no Firestore estava retornando os documentos da coleção personagens sem verificar a qual usuário cada personagem pertencia.

Por que acontecia

A consulta não possuía um filtro pelo campo userId.

Código com o erro:

const q = query(collection(db, "personagens"));

Dessa forma, a consulta buscava todos os personagens da coleção.

Como corrigi

Foi adicionado o filtro where() para buscar somente os personagens pertencentes ao usuário autenticado.

Código corrigido:

const q = query(
  collection(db, "personagens"),
  where("userId", "==", _uid)
);

Também foi adicionado o where ao import do Firebase:

import {
  collection,
  query,
  getDocs,
  addDoc,
  deleteDoc,
  doc,
  getDoc,
  setDoc,
  updateDoc,
  where,
  serverTimestamp,
} from "firebase/firestore";
Screenshot ou resultado

Não foram registrados screenshots durante a execução do bug.

Após a correção, a consulta passou a filtrar os personagens pelo userId, fazendo com que cada usuário visualize somente os seus próprios personagens.



## BUG #05 — Nome de coleção errado no Create
O que estava acontecendo

Ao criar um novo personagem, o sistema estava tentando salvar os dados em uma coleção chamada personagem, no singular.

Porém, a coleção correta utilizada pelo projeto é personagens, no plural.

Por que acontecia

O problema estava no nome da coleção utilizado no addDoc().

Código com o erro:

const ref = await addDoc(collection(db, "personagem"), { ... });

O nome "personagem" estava diferente do nome correto da coleção.

Como corrigi

O nome da coleção foi alterado de personagem para personagens.

Código corrigido:

const ref = await addDoc(collection(db, "personagens"), { ... });
Screenshot ou resultado

Não foram registrados screenshots durante a execução do bug.

Após a correção, a criação de personagens passou a utilizar a coleção correta personagens, mantendo a consistência com as demais operações do Firestore.



## BUG #06 — setDoc apaga o documento inteiro

### O que estava acontecendo

Ao equipar um item no personagem, o sistema utilizava `setDoc()`. Essa função poderia substituir os dados existentes do documento em vez de atualizar somente o campo que precisava ser alterado.

### Por que acontecia

O problema estava na utilização de `setDoc()` para uma atualização parcial do documento.

Código com o erro:

```ts
await setDoc(doc(db, "personagens", personagemId), { [slot]: itemId });

Como corrigi

O setDoc() foi substituído por updateDoc(), que permite atualizar somente o campo necessário sem substituir os outros dados do personagem.

Código corrigido:

await updateDoc(doc(db, "personagens", personagemId), { [slot]: itemId });
Screenshot ou resultado

Não foram registrados screenshots durante a execução do bug.

Após a correção, o sistema passou a atualizar somente o campo do item equipado, mantendo os demais dados do personagem.




## BUG #07 — Deletar usa índice como ID

### O que estava acontecendo

Ao excluir um personagem, o sistema utilizava o índice do personagem na lista como se fosse o ID do documento no Firestore.

### Por que acontecia

O problema estava na utilização da variável `indice` para identificar o documento.

**Código com o erro:**

``ts
await deleteDoc(doc(db, "personagens", String(indice)));

O índice representa apenas a posição do personagem na lista, e não o ID real do documento no Firestore.

Como corrigi

Foi utilizado o ID real do documento do personagem.

Código corrigido:

await deleteDoc(doc(db, "personagens", personagem.id));
Screenshot ou resultado

Não foram registrados screenshots durante a execução do bug.

Após a correção, o sistema passou a excluir o personagem utilizando corretamente o ID do documento no Firestore.


## BUG #08 — Security Rules abertas

### O que estava acontecendo

As regras de segurança do Firestore estavam permitindo que qualquer pessoa pudesse ler e escrever os documentos do banco de dados.

### Por que acontecia

O problema estava na regra que permitia leitura e escrita sem verificar se o usuário estava autenticado.

**Código com o erro:**

``text
match /{document=**} {
  allow read, write: if true;
}
Como a condição era true, qualquer pessoa poderia acessar os dados.

Como corrigi

As regras foram alteradas para exigir autenticação e verificar se o personagem pertence ao usuário que está fazendo a operação.

Código corrigido:

match /personagens/{personagemId} {
  allow read: if request.auth != null &&
              request.auth.uid == resource.data.userId;

  allow create: if request.auth != null &&
                request.auth.uid == request.resource.data.userId;

  allow update, delete: if request.auth != null &&
                        request.auth.uid == resource.data.userId;
}

Após a correção, as regras do Firestore passaram a proteger a coleção personagens, permitindo que cada usuário acesse e altere somente os seus próprios dados.