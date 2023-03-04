# Proteções_anti-frida



## Proteção contra Hooking com frida

##### **Explicando a comparação de seções de texto**

**O que é seções de texto**
As seções de texto são áreas da memória que contêm o código executável do aplicativo, podendo ser armazenadas tanto na memória do processo quanto no disco, dependendo da fase de execução do aplicativo.
 
**Comparação**
Para comparar a seção de texto na memória com a seção de texto no disco, é necessário obter o endereço de memória da biblioteca carregada e o caminho do arquivo da biblioteca no disco.

Para a biblioteca libc, podemos utilizar os seguintes passos:

1. Obter o endereço base da biblioteca libc na memória, chamando a função `dladdr` com o endereço de qualquer símbolo da biblioteca libc.

2. Abrir o arquivo da biblioteca libc no disco usando `fopen`, e obter o descritor do arquivo usando `fileno`.

3. Mapear a seção de texto do arquivo da biblioteca na memória usando `mmap`, e obter o endereço da seção mapeada.

4. 4.  Comparar a seção de texto na memória com a seção de texto no disco usando `memcmp`.

Para bibliotecas nativas, os passos são semelhantes, exceto que é necessário conhecer o caminho do arquivo da biblioteca no disco. O `dlopen` pode ser usado para carregar a biblioteca na memória e obter o seu endereço base. Em seguida, basta seguir os mesmos passos da biblioteca libc para comparar a seção de texto na memória com a seção de texto no disco.

---

##### Explicando a detecção de pipes do Frida

O Frida utiliza pipes nomeados para permitir a comunicação entre o processo em execução e o código injetado.

Um pipe é criado como um arquivo no sistema de arquivos e fica acessível por vários processos que compartilham o mesmo caminho de arquivo.

No Frida, o processo em execução e o código injetado se comunicam através de um ou mais pipes nomeados, que são criados pelo próprio Frida. O código injetado pode escrever dados em um pipe nomeado que são lidos pelo processo em execução e vice-versa.

**O que é um pipe?**

 Um pipe é um mecanismo de comunicação interprocesso (IPC) em sistemas operacionais. É um canal de comunicação que permite que processos se comuniquem por meio de um fluxo de dados unidirecional.

**Detecção através de pipes**

A detecção de pipes nomeados monitora a existência e o uso de pipes nomeados por processos em execução. Quando o Frida injeta código em um processo, ele cria pipes nomeados exclusivos para esse processo e, se o aplicativo em execução tentar criar um pipe nomeado com o mesmo caminho de arquivo, pode ser um sinal de que o aplicativo está tentando se comunicar com o código injetado pelo Frida.

**proteção anti-Frida detectando pipe nomeado**

A proteção anti-Frida monitora as syscalls do processo alvo e procura por chamadas relacionadas a pipes nomeados com certos critérios, como determinado nome de pipe ou um determinado modo de abertura. Quando as chamadas são detectadas, a proteção identifica o Frida como criador do pipe nomeado e pode encerrar o processo ou bloquear a comunicação entre o Frida e o agente remoto.

---

##### Explicando a detecção através de threads nomeados
A detecção de threads nomeados é utilizada para detectar a presença do Frida em um processo alvo. O Frida utiliza um thread específico nomeado `frida-helper-XXXX` para realizar algumas operações em um processo alvo.

**O que é um thread?**
 Um thread é uma unidade básica de um processo que é executado
 em paralelo com outros threads do mesmo processo. Cada thread tem seu próprio conjunto de registradores, pilhas e contexto de execução.

**Detecção de threads nomeados**
A detecção monitora os threads em execução no processo alvo e procura por threads com o nome `frida-helper-XXXX`. Caso um processo seja injetado com o Frida, o próprio Frida cria um thread nomeado com esse nome para realizar algumas operações.

**Proteção anti-Frida**
A proteção anti-Frida faz o monitoramento dos threads do processo alvo e quando detecta a presença de um thread nomeado com `frida-helper-XXXX`, pode tomar medidas para impedir que o Frida injete códigos ou realize outras operações. Uma das medidas tomadas pelo anti-Frida é interromper a execução do processo, bloquear a comunicação com o agente remoto do Frida ou tomar outras ações que impeçam a ação do Frida no processo alvo.

---
