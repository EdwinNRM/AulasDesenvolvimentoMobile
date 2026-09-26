# Aula 2: ListaTarefas

Projeto prático em React Native, Expo SDK 57, TypeScript e Cloud Firestore. Siga as etapas em ordem. Em cada etapa, substitua o conteúdo do arquivo indicado pelo código completo do bloco. O projeto na raiz já contém a versão final executável; as cópias de cada etapa estão em `etapas/`.

**Versão usada nesta aula:** Expo SDK 57 estável, com React Native 0.86 e React 19.2.3.

## Preparação no Windows

1. Baixe e instale o **Node.js LTS** em https://nodejs.org/en/download. Reinicie o terminal após a instalação. O npm vem junto.
2. Instale o **Visual Studio Code** em https://code.visualstudio.com/download.
3. Instale o **Expo Go** no celular pela loja de aplicativos (https://expo.dev/go). No iPhone, entre na mesma conta Expo usada pelo terminal.
4. Abra o terminal do VS Code e confira:

```powershell
node --version
npm --version
```

5. Em uma pasta para os projetos, crie o aplicativo com o template TypeScript simples:

```powershell
npx create-expo-app@latest ListaTarefas --template blank-typescript@57
cd ListaTarefas
code .
```

6. Pare qualquer servidor Expo antigo com `Ctrl+C` e execute `npx expo start --clear`. O computador e o celular devem estar na mesma rede Wi-Fi. No Android, abra o Expo Go e leia o QR code. No iPhone, use a câmera do sistema. Se aparecer **Failed to download remote update**, atualize o Expo Go e reinicie o servidor com `npx expo start --clear`. Depois, abra no navegador do **celular** o endereço `http://IP_DO_COMPUTADOR:8081` exibido pelo Expo; se ele não carregar, experimente `npx expo start --tunnel`.

## Firebase em poucos passos

1. No https://console.firebase.google.com, crie um projeto ou abra o projeto fornecido `aulamobile-1ed38`.
2. Em **Configurações do projeto**, registre uma aplicação **Web** e copie a configuração exibida. A configuração desta aula já está preenchida no código abaixo.
3. Em **Firestore Database**, crie o banco `(default)` e escolha a região. Para esta demonstração sem login, o modo de teste permite as operações do aplicativo por tempo limitado. Todos que tiverem acesso ao projeto poderão usar a mesma coleção; use apenas tarefas de exemplo e ajuste as regras após a aula. Se aparecer `permission-denied`, confira as regras e a validade do modo de teste.
4. No projeto Expo, instale o SDK com `npx expo install firebase`.
5. Crie `src/services/firebase.ts`. `initializeApp(firebaseConfig)` inicializa a aplicação Firebase e `getFirestore(app)` obtém a conexão com o banco.

A coleção `tarefas` é como uma pasta. Cada tarefa é um **documento** com um ID. `titulo`, `concluida` e `criadaEm` são **campos** do documento. O CRUD cria, lê, atualiza e exclui esses documentos.

O arquivo de configuração identifica a aplicação cliente. O acesso aos dados é controlado pelas **regras do Firestore**.

## Construção incremental

### Etapa 1: Primeira tela

**Altere:** `App.tsx`.

**Somente o que é novo:** `View` organiza a tela, `Text` exibe conteúdo, `StyleSheet` guarda estilos. `StatusBar` define a aparência da barra do sistema.

**No celular:** Aparecem “Minhas Tarefas” e a frase de apoio sobre um fundo claro.

**Arquivo: `App.tsx`**

```tsx
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  // Retorna os componentes que formam a primeira tela.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
});
```

### Etapa 2: Campo para digitar e estado

**Altere:** `App.tsx`.

**Somente o que é novo:** `useState('')` guarda o texto; `value` recebe o estado; `onChangeText` chama `setTitulo`. `placeholder` é uma prop.

**No celular:** O texto digitado aparece no campo e na linha “Você digitou”.

**Arquivo: `App.tsx`**

```tsx
import { useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, TextInput, View } from 'react-native';

export default function App() {
  // Guarda o texto atual do campo e redesenha a tela quando ele muda.
  const [titulo, setTitulo] = useState('');

  // value exibe o estado; onChangeText atualiza esse estado.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <TextInput
        style={styles.campo}
        placeholder="Digite uma tarefa"
        value={titulo}
        onChangeText={setTitulo}
      />
      <Text style={styles.dica}>Você digitou: {titulo}</Text>
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  campo: { backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16, marginTop: 28 },
  dica: { color: '#64748B', marginTop: 16 },
});
```

### Etapa 3: Botão e evento

**Altere:** `App.tsx`.

**Somente o que é novo:** `Pressable` reage ao toque; `onPress` chama `adicionarTarefa`. A função ignora texto vazio, mostra uma mensagem e limpa o campo.

**No celular:** Ao tocar em +, aparece “Botão pressionado” com o texto digitado. Ainda não há persistência.

**Arquivo: `App.tsx`**

```tsx
import { useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { Pressable, StyleSheet, Text, TextInput, View } from 'react-native';

export default function App() {
  // Estes estados guardam o texto do campo e a mensagem mostrada na tela.
  const [titulo, setTitulo] = useState('');
  const [mensagem, setMensagem] = useState('');

  // Trata o evento do botão, valida o texto e limpa o campo.
  function adicionarTarefa() {
    if (!titulo.trim()) return;
    setMensagem('Botão pressionado: ' + titulo.trim());
    setTitulo('');
  }

  // Pressable executa adicionarTarefa quando o usuário toca no botão.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <View style={styles.formulario}>
        <TextInput style={styles.campo} placeholder="Digite uma tarefa"
          value={titulo} onChangeText={setTitulo} />
        <Pressable style={styles.botao} onPress={adicionarTarefa}>
          <Text style={styles.botaoTexto}>+</Text>
        </Pressable>
      </View>
      <Text style={styles.mensagem}>{mensagem}</Text>
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16 },
  botao: { width: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoTexto: { color: '#FFFFFF', fontSize: 30 },
  mensagem: { color: '#2563EB', marginTop: 20 },
});
```

### Etapa 4: Conexão com Firebase

**Altere:** `src/services/firebase.ts`.

**Somente o que é novo:** `firebaseConfig` contém os dados do projeto; `initializeApp` inicia o Firebase; `getFirestore` fornece `db`. O `App.tsx` da etapa 3 permanece igual.

**No celular:** A tela ainda é a da etapa 3. Esta etapa prepara a conexão; a primeira gravação acontece na próxima.

**Arquivo: `src/services/firebase.ts`**

```ts
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';

// Identifica no Firebase qual projeto será usado pelo aplicativo.
const firebaseConfig = {
  apiKey: 'AIzaSyBJvYOFOppmjgziU7bFGi92BW3DoihGRsU',
  authDomain: 'aulamobile-1ed38.firebaseapp.com',
  projectId: 'aulamobile-1ed38',
  storageBucket: 'aulamobile-1ed38.firebasestorage.app',
  messagingSenderId: '19702875368',
  appId: '1:19702875368:web:765201fdf67353b18d355d',
};

// Inicializa o Firebase e exporta a conexão com o Firestore.
const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
```

### Etapa 5: CREATE com addDoc

**Altere:** `App.tsx`.

**Somente o que é novo:** `async` permite usar `await`; `collection(db, 'tarefas')` aponta para a coleção; `addDoc` cria um documento; `serverTimestamp` registra a criação.

**No celular:** Ao tocar em +, aparece “Tarefa salva no Firestore!”. No Firebase Console, a coleção `tarefas` mostra o novo documento.

**Arquivo: `App.tsx`**

```tsx
import { useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, serverTimestamp } from 'firebase/firestore';
import { db } from './src/services/firebase';

export default function App() {
  // Estes estados controlam o texto digitado e o retorno exibido ao usuário.
  const [titulo, setTitulo] = useState('');
  const [mensagem, setMensagem] = useState('');

  // CREATE: cria um documento na coleção tarefas e espera a gravação terminar.
  async function adicionarTarefa() {
    if (!titulo.trim()) return;
    await addDoc(collection(db, 'tarefas'), {
      titulo: titulo.trim(),
      concluida: false,
      criadaEm: serverTimestamp(),
    });
    setMensagem('Tarefa salva no Firestore!');
    setTitulo('');
  }

  // Monta o formulário usado para enviar a nova tarefa.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <View style={styles.formulario}>
        <TextInput style={styles.campo} placeholder="Digite uma tarefa"
          value={titulo} onChangeText={setTitulo} />
        <Pressable style={styles.botao} onPress={adicionarTarefa}>
          <Text style={styles.botaoTexto}>+</Text>
        </Pressable>
      </View>
      <Text style={styles.mensagem}>{mensagem}</Text>
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16 },
  botao: { width: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoTexto: { color: '#FFFFFF', fontSize: 30 },
  mensagem: { color: '#2563EB', marginTop: 20 },
});
```

### Etapa 6: READ e atualização da lista

**Altere:** `src/types/Tarefa.ts e App.tsx`.

**Somente o que é novo:** `Tarefa` define a forma dos dados; `useEffect` inicia o leitor; `onSnapshot` lê e acompanha mudanças; `setTarefas` atualiza o estado. `return cancelar` encerra o leitor ao sair.

**No celular:** A tela mostra a quantidade de tarefas armazenadas, inclusive após fechar e abrir o aplicativo.

**Arquivo: `src/types/Tarefa.ts`**

```ts
// Define o formato de uma tarefa dentro do aplicativo.
export type Tarefa = {
  id: string;
  titulo: string;
  concluida: boolean;
};
```

**Arquivo: `App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, onSnapshot, serverTimestamp } from 'firebase/firestore';
import { db } from './src/services/firebase';
import type { Tarefa } from './src/types/Tarefa';

// Referência reutilizada para ler e gravar na coleção tarefas.
const tarefasRef = collection(db, 'tarefas');

export default function App() {
  // Guarda o texto digitado e os documentos carregados do Firestore.
  const [titulo, setTitulo] = useState('');
  const [tarefas, setTarefas] = useState<Tarefa[]>([]);

  // READ: inicia a escuta da coleção quando a tela é aberta.
  useEffect(() => {
    const cancelar = onSnapshot(tarefasRef, (snapshot) => {
      setTarefas(snapshot.docs.map((documento) => ({
        id: documento.id,
        titulo: documento.data().titulo as string,
        concluida: documento.data().concluida as boolean,
      })));
    });
    // Cancela a escuta quando o componente sai da tela.
    return cancelar;
  }, []);

  // CREATE: adiciona uma tarefa e limpa o campo após salvar.
  async function adicionarTarefa() {
    if (!titulo.trim()) return;
    await addDoc(tarefasRef, {
      titulo: titulo.trim(),
      concluida: false,
      criadaEm: serverTimestamp(),
    });
    setTitulo('');
  }

  // Mostra o formulário e a quantidade de tarefas carregadas.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <View style={styles.formulario}>
        <TextInput style={styles.campo} placeholder="Digite uma tarefa"
          value={titulo} onChangeText={setTitulo} />
        <Pressable style={styles.botao} onPress={adicionarTarefa}>
          <Text style={styles.botaoTexto}>+</Text>
        </Pressable>
      </View>
      <Text style={styles.secao}>Tarefas carregadas: {tarefas.length}</Text>
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16 },
  botao: { width: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoTexto: { color: '#FFFFFF', fontSize: 30 },
  secao: { color: '#0F172A', fontSize: 18, fontWeight: '700', marginTop: 32 },
});
```

### Etapa 7: Renderização com FlatList

**Altere:** `App.tsx`.

**Somente o que é novo:** `data` é o array, `keyExtractor` usa o ID do documento e `renderItem` desenha cada linha. `ListEmptyComponent` aparece quando não há tarefas.

**No celular:** Cada tarefa do Firestore aparece em uma linha; uma lista vazia mostra uma mensagem.

**Arquivo: `App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { FlatList, Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, onSnapshot, serverTimestamp } from 'firebase/firestore';
import { db } from './src/services/firebase';
import type { Tarefa } from './src/types/Tarefa';

// Referência reutilizada para ler e gravar na coleção tarefas.
const tarefasRef = collection(db, 'tarefas');

export default function App() {
  // Guarda o texto digitado e os documentos carregados do Firestore.
  const [titulo, setTitulo] = useState('');
  const [tarefas, setTarefas] = useState<Tarefa[]>([]);

  // READ: mantém o estado sincronizado com os documentos da coleção.
  useEffect(() => {
    const cancelar = onSnapshot(tarefasRef, (snapshot) => {
      setTarefas(snapshot.docs.map((documento) => ({
        id: documento.id,
        titulo: documento.data().titulo as string,
        concluida: documento.data().concluida as boolean,
      })));
    });
    // Cancela a escuta quando o componente sai da tela.
    return cancelar;
  }, []);

  // CREATE: adiciona uma tarefa e limpa o campo após salvar.
  async function adicionarTarefa() {
    if (!titulo.trim()) return;
    await addDoc(tarefasRef, {
      titulo: titulo.trim(),
      concluida: false,
      criadaEm: serverTimestamp(),
    });
    setTitulo('');
  }

  // FlatList transforma o array tarefas em itens visuais.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <View style={styles.formulario}>
        <TextInput style={styles.campo} placeholder="Digite uma tarefa"
          value={titulo} onChangeText={setTitulo} />
        <Pressable style={styles.botao} onPress={adicionarTarefa}>
          <Text style={styles.botaoTexto}>+</Text>
        </Pressable>
      </View>
      <Text style={styles.secao}>Suas tarefas ({tarefas.length})</Text>
      <FlatList
        data={tarefas}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text style={styles.textoItem}>{item.titulo}</Text>
          </View>
        )}
        ListEmptyComponent={<Text style={styles.vazio}>Nenhuma tarefa ainda.</Text>}
      />
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16 },
  botao: { width: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoTexto: { color: '#FFFFFF', fontSize: 30 },
  secao: { color: '#0F172A', fontSize: 18, fontWeight: '700', marginTop: 32 },
  item: { backgroundColor: '#FFFFFF', borderRadius: 14, padding: 18, marginTop: 10 },
  textoItem: { color: '#1E293B', fontSize: 16 },
  vazio: { color: '#64748B', marginTop: 32, textAlign: 'center' },
});
```

### Etapa 8: DELETE com deleteDoc

**Altere:** `App.tsx`.

**Somente o que é novo:** `doc(db, 'tarefas', id)` seleciona o documento; `deleteDoc` o remove. O leitor da etapa 6 atualiza a tela.

**No celular:** Cada linha tem “Excluir”. Ao tocar, a tarefa desaparece da tela e do Firebase Console.

**Arquivo: `App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { FlatList, Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, deleteDoc, doc, onSnapshot, serverTimestamp } from 'firebase/firestore';
import { db } from './src/services/firebase';
import type { Tarefa } from './src/types/Tarefa';

// Referência reutilizada em todas as operações da coleção tarefas.
const tarefasRef = collection(db, 'tarefas');

export default function App() {
  // Guarda o texto digitado e os documentos carregados do Firestore.
  const [titulo, setTitulo] = useState('');
  const [tarefas, setTarefas] = useState<Tarefa[]>([]);

  // READ: mantém o estado sincronizado com os documentos da coleção.
  useEffect(() => {
    const cancelar = onSnapshot(tarefasRef, (snapshot) => {
      setTarefas(snapshot.docs.map((documento) => ({
        id: documento.id,
        titulo: documento.data().titulo as string,
        concluida: documento.data().concluida as boolean,
      })));
    });
    // Cancela a escuta quando o componente sai da tela.
    return cancelar;
  }, []);

  // CREATE: adiciona uma tarefa e limpa o campo após salvar.
  async function adicionarTarefa() {
    if (!titulo.trim()) return;
    await addDoc(tarefasRef, {
      titulo: titulo.trim(),
      concluida: false,
      criadaEm: serverTimestamp(),
    });
    setTitulo('');
  }

  // DELETE: remove o documento que possui o ID recebido.
  async function excluirTarefa(id: string) {
    await deleteDoc(doc(db, 'tarefas', id));
  }

  // FlatList mostra as tarefas e oferece a ação de exclusão.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <View style={styles.formulario}>
        <TextInput style={styles.campo} placeholder="Digite uma tarefa"
          value={titulo} onChangeText={setTitulo} />
        <Pressable style={styles.botao} onPress={adicionarTarefa}>
          <Text style={styles.botaoTexto}>+</Text>
        </Pressable>
      </View>
      <Text style={styles.secao}>Suas tarefas ({tarefas.length})</Text>
      <FlatList
        data={tarefas}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text style={styles.textoItem}>{item.titulo}</Text>
            <Pressable onPress={() => excluirTarefa(item.id)}>
              <Text style={styles.excluir}>Excluir</Text>
            </Pressable>
          </View>
        )}
        ListEmptyComponent={<Text style={styles.vazio}>Nenhuma tarefa ainda.</Text>}
      />
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16 },
  botao: { width: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoTexto: { color: '#FFFFFF', fontSize: 30 },
  secao: { color: '#0F172A', fontSize: 18, fontWeight: '700', marginTop: 32 },
  item: { backgroundColor: '#FFFFFF', borderRadius: 14, padding: 18, marginTop: 10,
    flexDirection: 'row', alignItems: 'center' },
  textoItem: { flex: 1, color: '#1E293B', fontSize: 16 },
  excluir: { color: '#DC2626', fontWeight: '600' },
  vazio: { color: '#64748B', marginTop: 32, textAlign: 'center' },
});
```

### Etapa 9: UPDATE com updateDoc e componente

**Altere:** `src/components/TaskItem.tsx e App.tsx`.

**Somente o que é novo:** `TaskItem` recebe `tarefa`, `onAlternar` e `onExcluir` por props. `updateDoc` altera `concluida`; `!` inverte o valor atual.

**No celular:** Ao tocar no círculo ou no título, aparece ou desaparece a marca de conclusão. A mudança persiste após reiniciar.

**Arquivo: `src/components/TaskItem.tsx`**

```tsx
import { Pressable, StyleSheet, Text, View } from 'react-native';
import type { Tarefa } from '../types/Tarefa';

// Define os dados e eventos que o componente recebe do App.
type Props = {
  tarefa: Tarefa;
  onAlternar: () => void;
  onExcluir: () => void;
};

// Exibe uma tarefa e chama as funções recebidas quando o usuário interage.
export function TaskItem({ tarefa, onAlternar, onExcluir }: Props) {
  return (
    <View style={styles.item}>
      <Pressable style={styles.conteudo} onPress={onAlternar}
        accessibilityRole="checkbox"
        accessibilityState={{ checked: tarefa.concluida }}
        accessibilityLabel={tarefa.titulo}>
        <View style={[styles.circulo, tarefa.concluida && styles.circuloMarcado]}>
          {tarefa.concluida ? <Text style={styles.marca}>✓</Text> : null}
        </View>
        <Text style={styles.titulo}>{tarefa.titulo}</Text>
      </Pressable>
      <Pressable onPress={onExcluir} accessibilityRole="button"
        accessibilityLabel={`Excluir ${tarefa.titulo}`}>
        <Text style={styles.excluir}>Excluir</Text>
      </Pressable>
    </View>
  );
}

// Define a aparência do cartão, do marcador e do botão de exclusão.
const styles = StyleSheet.create({
  item: { flexDirection: 'row', alignItems: 'center', backgroundColor: '#FFFFFF',
    borderRadius: 14, paddingHorizontal: 16, paddingVertical: 16, marginBottom: 10,
    borderWidth: 1, borderColor: '#E5EAF2' },
  conteudo: { flex: 1, flexDirection: 'row', alignItems: 'center', gap: 12 },
  circulo: { width: 24, height: 24, borderRadius: 12, borderWidth: 2,
    borderColor: '#2563EB', alignItems: 'center', justifyContent: 'center' },
  circuloMarcado: { backgroundColor: '#2563EB' },
  marca: { color: '#FFFFFF', fontSize: 14, fontWeight: '700' },
  titulo: { flex: 1, color: '#1E293B', fontSize: 16 },
  excluir: { color: '#DC2626', fontSize: 13, fontWeight: '600', paddingLeft: 10 },
});
```

**Arquivo: `App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { FlatList, Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, deleteDoc, doc, onSnapshot, serverTimestamp, updateDoc } from 'firebase/firestore';
import { db } from './src/services/firebase';
import { TaskItem } from './src/components/TaskItem';
import type { Tarefa } from './src/types/Tarefa';

// Referência reutilizada em todas as operações da coleção tarefas.
const tarefasRef = collection(db, 'tarefas');

export default function App() {
  // Guarda o texto digitado e os documentos carregados do Firestore.
  const [titulo, setTitulo] = useState('');
  const [tarefas, setTarefas] = useState<Tarefa[]>([]);

  // READ: mantém o estado sincronizado com os documentos da coleção.
  useEffect(() => {
    const cancelar = onSnapshot(tarefasRef, (snapshot) => {
      setTarefas(snapshot.docs.map((documento) => ({
        id: documento.id,
        titulo: documento.data().titulo as string,
        concluida: documento.data().concluida as boolean,
      })));
    });
    // Cancela a escuta quando o componente sai da tela.
    return cancelar;
  }, []);

  // CREATE: adiciona uma tarefa e limpa o campo após salvar.
  async function adicionarTarefa() {
    if (!titulo.trim()) return;
    await addDoc(tarefasRef, {
      titulo: titulo.trim(),
      concluida: false,
      criadaEm: serverTimestamp(),
    });
    setTitulo('');
  }

  // DELETE: remove o documento que possui o ID recebido.
  async function excluirTarefa(id: string) {
    await deleteDoc(doc(db, 'tarefas', id));
  }

  // UPDATE: inverte o valor do campo concluida no documento.
  async function alternarConclusao(tarefa: Tarefa) {
    await updateDoc(doc(db, 'tarefas', tarefa.id), {
      concluida: !tarefa.concluida,
    });
  }

  // TaskItem recebe a tarefa e as funções que tratam seus eventos.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <Text style={styles.titulo}>Minhas Tarefas</Text>
      <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
      <View style={styles.formulario}>
        <TextInput style={styles.campo} placeholder="Digite uma tarefa"
          value={titulo} onChangeText={setTitulo} />
        <Pressable style={styles.botao} onPress={adicionarTarefa}>
          <Text style={styles.botaoTexto}>+</Text>
        </Pressable>
      </View>
      <Text style={styles.secao}>Suas tarefas ({tarefas.length})</Text>
      <FlatList
        data={tarefas}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TaskItem tarefa={item}
            onAlternar={() => alternarConclusao(item)}
            onExcluir={() => excluirTarefa(item.id)} />
        )}
        ListEmptyComponent={<Text style={styles.vazio}>Nenhuma tarefa ainda.</Text>}
      />
    </View>
  );
}

// Reúne os estilos usados pelos componentes da tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', padding: 24, paddingTop: 58 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800' },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0',
    borderWidth: 1, borderRadius: 14, padding: 16, fontSize: 16 },
  botao: { width: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoTexto: { color: '#FFFFFF', fontSize: 30 },
  secao: { color: '#0F172A', fontSize: 18, fontWeight: '700', marginTop: 32 },
  item: { backgroundColor: '#FFFFFF', borderRadius: 14, padding: 18, marginTop: 10,
    flexDirection: 'row', alignItems: 'center' },
  textoItem: { flex: 1, color: '#1E293B', fontSize: 16 },
  excluir: { color: '#DC2626', fontWeight: '600' },
  vazio: { color: '#64748B', marginTop: 32, textAlign: 'center' },
});
```

### Etapa 10: Erros e visual final

**Altere:** `App.tsx`.

**Somente o que é novo:** `try/catch` captura falhas assíncronas; `erro` mostra retorno na tela; `salvando` evita toques repetidos; `carregando` informa a leitura inicial. Os estilos refinam a interface.

**No celular:** A tela final mostra cabeçalho, entrada, contador e cartões. Se uma operação falhar, aparece uma mensagem legível.

**Arquivo: `App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { ActivityIndicator, FlatList, Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, deleteDoc, doc, onSnapshot, serverTimestamp, updateDoc } from 'firebase/firestore';
import { TaskItem } from './src/components/TaskItem';
import { db } from './src/services/firebase';
import type { Tarefa } from './src/types/Tarefa';

// Mantém uma referência para a coleção usada em todas as operações do CRUD.
const tarefasRef = collection(db, 'tarefas');

export default function App() {
  // Estados que controlam o formulário, a lista e os retornos visuais da tela.
  const [titulo, setTitulo] = useState('');
  const [tarefas, setTarefas] = useState<Tarefa[]>([]);
  const [carregando, setCarregando] = useState(true);
  const [salvando, setSalvando] = useState(false);
  const [erro, setErro] = useState('');

  // READ: inicia a escuta em tempo real ao abrir a tela.
  useEffect(() => {
    const cancelar = onSnapshot(tarefasRef, (snapshot) => {
      // Converte cada documento do Firestore para o formato Tarefa do aplicativo.
      const lista = snapshot.docs.map((documento) => ({
        id: documento.id,
        titulo: documento.data().titulo as string,
        concluida: documento.data().concluida as boolean,
      }));
      setTarefas(lista);
      setCarregando(false);
      setErro('');
    }, () => {
      setCarregando(false);
      setErro('Não foi possível carregar as tarefas. Confira a internet e as regras do Firestore.');
    });
    // Encerra a escuta quando o componente sai da tela.
    return cancelar;
  }, []);

  // CREATE: valida o texto e cria um documento na coleção tarefas.
  async function adicionarTarefa() {
    const texto = titulo.trim();
    if (!texto || salvando) return;
    setSalvando(true);
    setErro('');
    try {
      await addDoc(tarefasRef, {
        titulo: texto,
        concluida: false,
        criadaEm: serverTimestamp(),
      });
      setTitulo('');
    } catch {
      setErro('Não foi possível adicionar a tarefa. Tente novamente.');
    } finally {
      setSalvando(false);
    }
  }

  // DELETE: localiza o documento pelo ID e o remove do Firestore.
  async function excluirTarefa(id: string) {
    setErro('');
    try {
      await deleteDoc(doc(db, 'tarefas', id));
    } catch {
      setErro('Não foi possível excluir a tarefa. Tente novamente.');
    }
  }

  // UPDATE: alterna o campo concluida entre verdadeiro e falso.
  async function alternarConclusao(tarefa: Tarefa) {
    setErro('');
    try {
      await updateDoc(doc(db, 'tarefas', tarefa.id), { concluida: !tarefa.concluida });
    } catch {
      setErro('Não foi possível atualizar a tarefa. Tente novamente.');
    }
  }

  // Monta a interface com formulário, mensagens e lista de tarefas.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <View style={styles.conteudo}>
        <Text style={styles.etiqueta}>LISTA DE TAREFAS</Text>
        <Text style={styles.titulo}>Minhas Tarefas</Text>
        <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
        <View style={styles.formulario}>
          <TextInput
            style={styles.campo}
            placeholder="Digite uma tarefa"
            placeholderTextColor="#94A3B8"
            value={titulo}
            onChangeText={setTitulo}
            onSubmitEditing={adicionarTarefa}
            returnKeyType="done"
            accessibilityLabel="Nova tarefa"
          />
          <Pressable style={[styles.botao, salvando && styles.botaoDesativado]}
            onPress={adicionarTarefa} disabled={salvando}
            accessibilityRole="button" accessibilityLabel="Adicionar tarefa">
            <Text style={styles.botaoTexto}>{salvando ? '...' : '+'}</Text>
          </Pressable>
        </View>
        {erro ? <Text style={styles.erro}>{erro}</Text> : null}
        <View style={styles.cabecalhoLista}>
          <Text style={styles.secao}>Suas tarefas</Text>
          <Text style={styles.contador}>{tarefas.length}</Text>
        </View>
        {carregando ? <ActivityIndicator color="#2563EB" style={styles.carregando} /> : (
          <FlatList
            data={tarefas}
            keyExtractor={(item) => item.id}
            renderItem={({ item }) => (
              <TaskItem tarefa={item}
                onAlternar={() => alternarConclusao(item)}
                onExcluir={() => excluirTarefa(item.id)} />
            )}
            ListEmptyComponent={<Text style={styles.vazio}>Nenhuma tarefa ainda. Adicione a primeira!</Text>}
            contentContainerStyle={styles.lista}
            keyboardShouldPersistTaps="handled"
          />
        )}
      </View>
    </View>
  );
}

// Centraliza os estilos visuais usados pelos componentes desta tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', paddingTop: 58 },
  conteudo: { flex: 1, paddingHorizontal: 24 },
  etiqueta: { color: '#2563EB', fontWeight: '700', fontSize: 12, letterSpacing: 2 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800', marginTop: 10 },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0', borderWidth: 1,
    borderRadius: 14, paddingHorizontal: 16, fontSize: 16, color: '#0F172A', minHeight: 54 },
  botao: { width: 54, height: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoDesativado: { opacity: 0.6 },
  botaoTexto: { color: '#FFFFFF', fontSize: 30, fontWeight: '500', marginTop: -3 },
  erro: { color: '#B42318', backgroundColor: '#FEF3F2', padding: 12,
    borderRadius: 10, marginTop: 14 },
  cabecalhoLista: { flexDirection: 'row', alignItems: 'center', gap: 8, marginTop: 32, marginBottom: 14 },
  secao: { color: '#0F172A', fontSize: 18, fontWeight: '700' },
  contador: { color: '#2563EB', backgroundColor: '#DBEAFE', overflow: 'hidden',
    borderRadius: 12, paddingHorizontal: 8, paddingVertical: 2, fontWeight: '700' },
  lista: { paddingBottom: 24, flexGrow: 1 },
  vazio: { color: '#64748B', textAlign: 'center', marginTop: 52, fontSize: 15 },
  carregando: { marginTop: 48 },
});
```

## Projeto final

```text
ListaTarefas/
├── App.tsx
├── index.ts
├── app.json
├── .gitignore
├── package.json
├── package-lock.json
├── tsconfig.json
├── eslint.config.js
├── assets/
└── src/
    ├── components/TaskItem.tsx
    ├── services/firebase.ts
    └── types/Tarefa.ts
```

O Expo criou `index.ts`, `tsconfig.json` e `assets/`; eles não precisam ser alterados. `eslint.config.js` foi criado pela checagem `npx expo lint`. `package-lock.json` é gerado pelo npm. O guia e as cópias em `etapas/` são material de apoio e não fazem parte do aplicativo em execução.

### Código final completo

**Arquivo: `App.tsx`**

```tsx
import { useEffect, useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { ActivityIndicator, FlatList, Pressable, StyleSheet, Text, TextInput, View } from 'react-native';
import { addDoc, collection, deleteDoc, doc, onSnapshot, serverTimestamp, updateDoc } from 'firebase/firestore';
import { TaskItem } from './src/components/TaskItem';
import { db } from './src/services/firebase';
import type { Tarefa } from './src/types/Tarefa';

// Mantém uma referência para a coleção usada em todas as operações do CRUD.
const tarefasRef = collection(db, 'tarefas');

export default function App() {
  // Estados que controlam o formulário, a lista e os retornos visuais da tela.
  const [titulo, setTitulo] = useState('');
  const [tarefas, setTarefas] = useState<Tarefa[]>([]);
  const [carregando, setCarregando] = useState(true);
  const [salvando, setSalvando] = useState(false);
  const [erro, setErro] = useState('');

  // READ: inicia a escuta em tempo real ao abrir a tela.
  useEffect(() => {
    const cancelar = onSnapshot(tarefasRef, (snapshot) => {
      // Converte cada documento do Firestore para o formato Tarefa do aplicativo.
      const lista = snapshot.docs.map((documento) => ({
        id: documento.id,
        titulo: documento.data().titulo as string,
        concluida: documento.data().concluida as boolean,
      }));
      setTarefas(lista);
      setCarregando(false);
      setErro('');
    }, () => {
      setCarregando(false);
      setErro('Não foi possível carregar as tarefas. Confira a internet e as regras do Firestore.');
    });

    // Encerra a escuta quando o componente sai da tela.
    return cancelar;
  }, []);

  // CREATE: valida o texto e cria um documento na coleção tarefas.
  async function adicionarTarefa() {
    const texto = titulo.trim();
    if (!texto || salvando) return;
    setSalvando(true);
    setErro('');
    try {
      await addDoc(tarefasRef, {
        titulo: texto,
        concluida: false,
        criadaEm: serverTimestamp(),
      });
      setTitulo('');
    } catch {
      setErro('Não foi possível adicionar a tarefa. Tente novamente.');
    } finally {
      setSalvando(false);
    }
  }

  // DELETE: localiza o documento pelo ID e o remove do Firestore.
  async function excluirTarefa(id: string) {
    setErro('');
    try {
      await deleteDoc(doc(db, 'tarefas', id));
    } catch {
      setErro('Não foi possível excluir a tarefa. Tente novamente.');
    }
  }

  // UPDATE: alterna o campo concluida entre verdadeiro e falso.
  async function alternarConclusao(tarefa: Tarefa) {
    setErro('');
    try {
      await updateDoc(doc(db, 'tarefas', tarefa.id), { concluida: !tarefa.concluida });
    } catch {
      setErro('Não foi possível atualizar a tarefa. Tente novamente.');
    }
  }

  // Monta a interface com formulário, mensagens e lista de tarefas.
  return (
    <View style={styles.tela}>
      <StatusBar style="dark" />
      <View style={styles.conteudo}>
        <Text style={styles.etiqueta}>LISTA DE TAREFAS</Text>
        <Text style={styles.titulo}>Minhas Tarefas</Text>
        <Text style={styles.subtitulo}>Organize o que você precisa fazer.</Text>
        <View style={styles.formulario}>
          <TextInput
            style={styles.campo}
            placeholder="Digite uma tarefa"
            placeholderTextColor="#94A3B8"
            value={titulo}
            onChangeText={setTitulo}
            onSubmitEditing={adicionarTarefa}
            returnKeyType="done"
            accessibilityLabel="Nova tarefa"
          />
          <Pressable style={[styles.botao, salvando && styles.botaoDesativado]}
            onPress={adicionarTarefa} disabled={salvando}
            accessibilityRole="button" accessibilityLabel="Adicionar tarefa">
            <Text style={styles.botaoTexto}>{salvando ? '...' : '+'}</Text>
          </Pressable>
        </View>
        {erro ? <Text style={styles.erro}>{erro}</Text> : null}
        <View style={styles.cabecalhoLista}>
          <Text style={styles.secao}>Suas tarefas</Text>
          <Text style={styles.contador}>{tarefas.length}</Text>
        </View>
        {carregando ? <ActivityIndicator color="#2563EB" style={styles.carregando} /> : (
          <FlatList
            data={tarefas}
            keyExtractor={(item) => item.id}
            renderItem={({ item }) => (
              <TaskItem tarefa={item}
                onAlternar={() => alternarConclusao(item)}
                onExcluir={() => excluirTarefa(item.id)} />
            )}
            ListEmptyComponent={<Text style={styles.vazio}>Nenhuma tarefa ainda. Adicione a primeira!</Text>}
            contentContainerStyle={styles.lista}
            keyboardShouldPersistTaps="handled"
          />
        )}
      </View>
    </View>
  );
}

// Centraliza os estilos visuais usados pelos componentes desta tela.
const styles = StyleSheet.create({
  tela: { flex: 1, backgroundColor: '#F7F9FC', paddingTop: 58 },
  conteudo: { flex: 1, paddingHorizontal: 24 },
  etiqueta: { color: '#2563EB', fontWeight: '700', fontSize: 12, letterSpacing: 2 },
  titulo: { color: '#0F172A', fontSize: 34, fontWeight: '800', marginTop: 10 },
  subtitulo: { color: '#64748B', fontSize: 15, marginTop: 6 },
  formulario: { flexDirection: 'row', gap: 10, marginTop: 28 },
  campo: { flex: 1, backgroundColor: '#FFFFFF', borderColor: '#DCE4F0', borderWidth: 1,
    borderRadius: 14, paddingHorizontal: 16, fontSize: 16, color: '#0F172A', minHeight: 54 },
  botao: { width: 54, height: 54, borderRadius: 14, backgroundColor: '#2563EB',
    alignItems: 'center', justifyContent: 'center' },
  botaoDesativado: { opacity: 0.6 },
  botaoTexto: { color: '#FFFFFF', fontSize: 30, fontWeight: '500', marginTop: -3 },
  erro: { color: '#B42318', backgroundColor: '#FEF3F2', padding: 12,
    borderRadius: 10, marginTop: 14 },
  cabecalhoLista: { flexDirection: 'row', alignItems: 'center', gap: 8, marginTop: 32, marginBottom: 14 },
  secao: { color: '#0F172A', fontSize: 18, fontWeight: '700' },
  contador: { color: '#2563EB', backgroundColor: '#DBEAFE', overflow: 'hidden',
    borderRadius: 12, paddingHorizontal: 8, paddingVertical: 2, fontWeight: '700' },
  lista: { paddingBottom: 24, flexGrow: 1 },
  vazio: { color: '#64748B', textAlign: 'center', marginTop: 52, fontSize: 15 },
  carregando: { marginTop: 48 },
});
```

**Arquivo: `src/components/TaskItem.tsx`**

```tsx
import { Pressable, StyleSheet, Text, View } from 'react-native';
import type { Tarefa } from '../types/Tarefa';

// Props são os dados e eventos recebidos do componente App.
type Props = {
  tarefa: Tarefa;
  onAlternar: () => void;
  onExcluir: () => void;
};

// Exibe uma tarefa e delega as ações de concluir e excluir ao componente pai.
export function TaskItem({ tarefa, onAlternar, onExcluir }: Props) {
  return (
    <View style={styles.item}>
      <Pressable style={styles.conteudo} onPress={onAlternar}
        accessibilityRole="checkbox"
        accessibilityState={{ checked: tarefa.concluida }}
        accessibilityLabel={tarefa.titulo}>
        <View style={[styles.circulo, tarefa.concluida && styles.circuloMarcado]}>
          {tarefa.concluida ? <Text style={styles.marca}>✓</Text> : null}
        </View>
        <Text style={styles.titulo}>{tarefa.titulo}</Text>
      </Pressable>
      <Pressable onPress={onExcluir} accessibilityRole="button"
        accessibilityLabel={`Excluir ${tarefa.titulo}`}>
        <Text style={styles.excluir}>Excluir</Text>
      </Pressable>
    </View>
  );
}

// Define a aparência do cartão, do marcador e do botão de exclusão.
const styles = StyleSheet.create({
  item: { flexDirection: 'row', alignItems: 'center', backgroundColor: '#FFFFFF',
    borderRadius: 14, paddingHorizontal: 16, paddingVertical: 16, marginBottom: 10,
    borderWidth: 1, borderColor: '#E5EAF2' },
  conteudo: { flex: 1, flexDirection: 'row', alignItems: 'center', gap: 12 },
  circulo: { width: 24, height: 24, borderRadius: 12, borderWidth: 2,
    borderColor: '#2563EB', alignItems: 'center', justifyContent: 'center' },
  circuloMarcado: { backgroundColor: '#2563EB' },
  marca: { color: '#FFFFFF', fontSize: 14, fontWeight: '700' },
  titulo: { flex: 1, color: '#1E293B', fontSize: 16 },
  excluir: { color: '#DC2626', fontSize: 13, fontWeight: '600', paddingLeft: 10 },
});
```

**Arquivo: `src/services/firebase.ts`**

```ts
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';

// Identifica no Firebase qual projeto será usado pelo aplicativo.
const firebaseConfig = {
  apiKey: 'AIzaSyBJvYOFOppmjgziU7bFGi92BW3DoihGRsU',
  authDomain: 'aulamobile-1ed38.firebaseapp.com',
  projectId: 'aulamobile-1ed38',
  storageBucket: 'aulamobile-1ed38.firebasestorage.app',
  messagingSenderId: '19702875368',
  appId: '1:19702875368:web:765201fdf67353b18d355d',
};

// Inicializa o Firebase uma única vez e exporta a conexão com o Firestore.
const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
```

**Arquivo: `src/types/Tarefa.ts`**

```ts
// Define os campos que toda tarefa precisa ter dentro do aplicativo.
export type Tarefa = {
  id: string;
  titulo: string;
  concluida: boolean;
};
```

**Arquivo: `package.json`**

```json
{
  "name": "listatarefas",
  "version": "1.0.0",
  "main": "index.ts",
  "dependencies": {
    "expo": "^57.0.25",
    "expo-status-bar": "~57.0.1",
    "firebase": "^12.19.0",
    "react": "19.2.3",
    "react-dom": "19.2.3",
    "react-native": "0.86.3",
    "react-native-web": "^0.21.2"
  },
  "devDependencies": {
    "@expo/ngrok": "^4.1.3",
    "@types/react": "~19.2.4",
    "eslint": "^9.0.0",
    "eslint-config-expo": "~57.0.2",
    "typescript": "~6.0.3"
  },
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "lint": "expo lint"
  },
  "private": true
}
```

**Arquivo: `app.json`**

```json
{
  "expo": {
    "name": "ListaTarefas",
    "slug": "ListaTarefas",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "ios": {
      "supportsTablet": true
    },
    "android": {
      "adaptiveIcon": {
        "backgroundColor": "#E6F4FE",
        "foregroundImage": "./assets/android-icon-foreground.png",
        "backgroundImage": "./assets/android-icon-background.png",
        "monochromeImage": "./assets/android-icon-monochrome.png"
      },
      "predictiveBackGestureEnabled": false
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

**Arquivo: `.gitignore`**

```gitignore
# Learn more https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files

# Instruções locais de agentes de código
AGENTS.md
CLAUDE.md
.agents/
.claude/

# dependencies
node_modules/

# Expo
.expo/
dist/
web-build/
expo-env.d.ts

# Native
.kotlin/
*.orig.*
*.jks
*.p8
*.p12
*.key
*.mobileprovision

# Metro
.metro-health-check*

# debug
npm-debug.*
yarn-debug.*
yarn-error.*

# macOS
.DS_Store
*.pem

# local env files
.env*.local

# typescript
*.tsbuildinfo

# generated native folders
/ios
/android

# Local build checks and slide preparation
.expo-export-check/
.codex-finalizer/
.slide-build/
*.validation.json
```

**Arquivo: `eslint.config.js`**

```js
// https://docs.expo.dev/guides/using-eslint/
const { defineConfig } = require('eslint/config');
const expoConfig = require("eslint-config-expo/flat");

module.exports = defineConfig([
  expoConfig,
  {
    ignores: ["dist/*"],
  }
]);
```

## Comandos para instalar e executar

```powershell
npx create-expo-app@latest ListaTarefas --template blank-typescript@57
cd ListaTarefas
npx expo install firebase
code .
npx expo start
```

Se usar a pasta deste material já pronta, execute apenas `cd ListaTarefas`, `npm install` e `npx expo start`.

## Checklist no celular e no Firebase Console

- [ ] A tela “Minhas Tarefas” abre no Expo Go.
- [ ] Digitar no campo altera o texto.
- [ ] Tocar em + cria um documento na coleção `tarefas`.
- [ ] O documento tem `titulo`, `concluida: false` e `criadaEm`.
- [ ] A lista carrega as tarefas ao abrir o aplicativo.
- [ ] Tocar em uma tarefa alterna a marca de conclusão e o campo `concluida`.
- [ ] Tocar em “Excluir” remove o documento.
- [ ] Fechar e abrir o aplicativo mantém as tarefas que não foram excluídas.
- [ ] Ao falhar uma operação, o aplicativo mostra uma mensagem.

## Atividade dos alunos

Em `src/components/TaskItem.tsx`, faça o título de uma tarefa concluída aparecer com outra cor e riscado. Crie um estilo `tituloConcluido` com `textDecorationLine: 'line-through'` e aplique-o condicionalmente com `tarefa.concluida`. Verifique a mudança no Expo Go e confirme que ela permanece ao reabrir o app.

## Prints para os slides

1. **VS Code:** `App.tsx` nas linhas do `TextInput`, `Pressable` e `useState`, com a árvore de arquivos visível.
2. **VS Code:** `src/services/firebase.ts` mostrando `initializeApp` e `getFirestore`.
3. **VS Code:** `App.tsx` nas funções `addDoc`, `onSnapshot`, `deleteDoc` e `updateDoc`. Faça um print por operação, com zoom legível.
4. **Terminal:** saída de `node --version`, `npm --version` e o QR code de `npx expo start`.
5. **Expo Go:** tela vazia, tela com duas tarefas, uma tarefa marcada e lista após exclusão.
6. **Firebase Console:** coleção `tarefas` e um documento com os campos `titulo`, `concluida` e `criadaEm`. Oculte outros dados do projeto antes de compartilhar.

## Documentação consultada

- Expo: https://docs.expo.dev/more/create-expo/, https://docs.expo.dev/versions/v57.0.0/
- Expo com Firebase: https://docs.expo.dev/guides/using-firebase/
- Firestore: https://firebase.google.com/docs/firestore/quickstart
- Leitura em tempo real: https://firebase.google.com/docs/firestore/query-data/listen
- Escrita, atualização e exclusão: https://firebase.google.com/docs/firestore/manage-data/add-data e https://firebase.google.com/docs/firestore/manage-data/delete-data
