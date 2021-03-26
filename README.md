## Core Java NIO

# Introdução ao Seletor Java NIO

# 1. Visão Geral
Neste artigo, exploraremos as partes introdutórias do componente Selector do Java NIO.

Um seletor fornece um mecanismo para monitorar um ou mais canais NIO e reconhecer quando um ou mais se tornam disponíveis para transferência de dados.

Dessa forma, um único thread pode ser usado para gerenciar vários canais e, portanto, várias conexões de rede.

# 2. Por que usar um seletor?
Com um seletor, podemos usar um thread em vez de vários para gerenciar vários canais. A troca de contexto entre threads é cara para o sistema operacional e, além disso, cada thread ocupa memória.

Portanto, quanto menos threads usarmos, melhor. No entanto, é importante lembrar que os sistemas operacionais e CPU modernos estão cada vez melhores em multitarefa, de modo que os overheads de multitarefa diminuem com o tempo.

Aqui, estaremos lidando com como podemos lidar com vários canais com um único thread usando um seletor.

Observe também que os seletores não ajudam apenas a ler os dados; eles também podem escutar conexões de rede de entrada e gravar dados em canais lentos.

# 3. Configuração
Para usar o seletor, não precisamos de nenhuma configuração especial. Todas as classes de que precisamos estão no pacote principal java.nio e só temos que importar o que precisamos.

Depois disso, podemos registrar vários canais com um objeto seletor. Quando a atividade de I / O acontece em qualquer um dos canais, o seletor nos notifica. É assim que podemos ler um grande número de fontes de dados em um único thread.

Qualquer canal que registramos com um seletor deve ser uma subclasse de SelectableChannel. Esses são um tipo especial de canais que podem ser colocados no modo sem bloqueio.

# 4. Criação de um seletor
Um seletor pode ser criado invocando o método estático aberto da classe Selector, que usará o provedor de seletor padrão do sistema para criar um novo seletor:

```
Selector selector = Selector.open();
```

# 5. Registro de canais selecionáveis
Para que um seletor monitore quaisquer canais, devemos registrar esses canais com o seletor. Fazemos isso invocando o método de registro do canal selecionável.

Mas antes que um canal seja registrado com um seletor, ele deve estar no modo sem bloqueio:

```
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

Isso significa que não podemos usar FileChannels com um seletor, pois eles não podem ser alternados para o modo sem bloqueio da maneira que fazemos com canais de soquete.

O primeiro parâmetro é o objeto Selector que criamos anteriormente, o segundo parâmetro define um conjunto de interesse, ou seja, quais eventos estamos interessados ​​em escutar no canal monitorado, por meio do seletor.

Existem quatro eventos diferentes que podemos ouvir, cada um é representado por uma constante na classe SelectionKey:

- Conectar - quando um cliente tenta se conectar ao servidor. Representado por SelectionKey.OP_CONNECT;
- Aceitar - quando o servidor aceita uma conexão de um cliente. Representado por SelectionKey.OP_ACCEPT;
- Ler - quando o servidor está pronto para ler do canal. Representado por SelectionKey.OP_READ;
- Gravação - quando o servidor está pronto para gravar no canal. Representado por SelectionKey.OP_WRITE.

O objeto retornado SelectionKey representa o registro do canal selecionável com o seletor. Veremos isso mais detalhadamente na seção a seguir.

# 6. O objeto SelectionKey
Como vimos na seção anterior, quando registramos um canal com um seletor, obtemos um objeto SelectionKey. Este objeto contém dados que representam o registro do canal.

Ele contém algumas propriedades importantes que devemos entender bem para poder usar o seletor no canal. Veremos essas propriedades nas subseções a seguir.

### 6.1. O Conjunto de Interesses
Um conjunto de interesse define o conjunto de eventos que queremos que o seletor observe neste canal. É um valor inteiro; podemos obter essas informações da seguinte maneira.

Primeiro, temos o conjunto de juros retornado pelo método interestOps de SelectionKey. Então temos a constante de evento em SelectionKey que vimos anteriormente.

Quando fazemos o AND desses dois valores, obtemos um valor booleano que nos diz se o evento está sendo observado ou não:

```
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```

### 6.2. O Conjunto Pronto
O conjunto pronto define o conjunto de eventos para os quais o canal está pronto. É um valor inteiro também; podemos obter essas informações da seguinte maneira.

Temos o conjunto pronto retornado pelo método readyOps de SelectionKey. Quando fazemos o AND desse valor com as constantes de eventos como fizemos no caso do conjunto de interesse, obtemos um booleano que representa se o canal está pronto para um determinado valor ou não.

Outra alternativa e maneira mais rápida de fazer isso é usar os métodos de conveniência do SelectionKey para o mesmo propósito:

```
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWriteable();
```

### 6.3. O canal
Acessar o canal que está sendo assistido a partir do objeto SelectionKey é muito simples. Nós apenas chamamos o método do canal:

```
Channel channel = key.channel();
```

### 6.4. O seletor
Assim como obter um canal, é muito fácil obter o objeto Selector do objeto SelectionKey:

```
Selector selector = key.selector();
```

### 6.5. Anexando objetos

Podemos anexar um objeto a um SelectionKey. Às vezes, podemos querer dar a um canal um ID personalizado ou anexar qualquer tipo de objeto Java que desejamos acompanhar.

Anexar objetos é uma maneira prática de fazer isso. Aqui está como você anexa e obtém objetos de um SelectionKey:

```
key.attach(Object);

Object object = key.attachment();
```

Como alternativa, podemos escolher anexar um objeto durante o registro do canal. Nós o adicionamos como um terceiro parâmetro ao método de registro do canal, assim:

```
SelectionKey key = channel.register(
  selector, SelectionKey.OP_ACCEPT, object);
```

# 7. Seleção de chave de canal
Até agora, vimos como criar um seletor, registrar canais para ele e inspecionar as propriedades do objeto SelectionKey que representa o registro de um canal para um seletor.

Esta é apenas a metade do processo, agora temos que realizar um processo contínuo de seleção do conjunto pronto que vimos anteriormente. Fazemos a seleção usando o método de seleção do seletor, assim:

```
int channels = selector.select();
```

Este método bloqueia até que pelo menos um canal esteja pronto para uma operação. O inteiro retornado representa o número de chaves cujos canais estão prontos para uma operação.

Em seguida, geralmente recuperamos o conjunto de chaves selecionadas para processamento:

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

O conjunto que obtivemos é de objetos SelectionKey, cada chave representa um canal registrado que está pronto para uma operação.

Depois disso, geralmente iteramos sobre este conjunto e para cada chave, obtemos o canal e executamos qualquer uma das operações que aparecem em nosso conjunto de interesse nele.


Durante a vida útil de um canal, ele pode ser selecionado várias vezes conforme sua chave aparece no conjunto pronto para diferentes eventos. É por isso que devemos ter um loop contínuo para capturar e processar eventos de canal como e quando eles ocorrem.

# 8. Exemplo Completo
Para cimentar o conhecimento que adquirimos nas seções anteriores, vamos construir um exemplo cliente-servidor completo.

Para facilitar o teste de nosso código, construiremos um servidor de eco e um cliente de eco. Nesse tipo de configuração, o cliente se conecta ao servidor e começa a enviar mensagens para ele. O servidor ecoa mensagens enviadas por cada cliente.

Quando o servidor encontra uma mensagem específica, como fim, ele a interpreta como o fim da comunicação e fecha a conexão com o cliente.

### 8.1. O servidor
Aqui está nosso código para EchoServer.java:

```
public class EchoServer {

    private static final String POISON_PILL = "POISON_PILL";

    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.bind(new InetSocketAddress("localhost", 5454));
        serverSocket.configureBlocking(false);
        serverSocket.register(selector, SelectionKey.OP_ACCEPT);
        ByteBuffer buffer = ByteBuffer.allocate(256);

        while (true) {
            selector.select();
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iter = selectedKeys.iterator();
            while (iter.hasNext()) {

                SelectionKey key = iter.next();

                if (key.isAcceptable()) {
                    register(selector, serverSocket);
                }

                if (key.isReadable()) {
                    answerWithEcho(buffer, key);
                }
                iter.remove();
            }
        }
    }

    private static void answerWithEcho(ByteBuffer buffer, SelectionKey key)
      throws IOException {
 
        SocketChannel client = (SocketChannel) key.channel();
        client.read(buffer);
        if (new String(buffer.array()).trim().equals(POISON_PILL)) {
            client.close();
            System.out.println("Not accepting client messages anymore");
        }
        else {
            buffer.flip();
            client.write(buffer);
            buffer.clear();
        }
    }

    private static void register(Selector selector, ServerSocketChannel serverSocket)
      throws IOException {
 
        SocketChannel client = serverSocket.accept();
        client.configureBlocking(false);
        client.register(selector, SelectionKey.OP_READ);
    }

    public static Process start() throws IOException, InterruptedException {
        String javaHome = System.getProperty("java.home");
        String javaBin = javaHome + File.separator + "bin" + File.separator + "java";
        String classpath = System.getProperty("java.class.path");
        String className = EchoServer.class.getCanonicalName();

        ProcessBuilder builder = new ProcessBuilder(javaBin, "-cp", classpath, className);

        return builder.start();
    }
}
```

Java NIO usa um modelo orientado a buffer diferente de um modelo orientado a fluxo. Portanto, a comunicação de soquete geralmente ocorre escrevendo e lendo de um buffer.

Portanto, criamos um novo ByteBuffer no qual o servidor irá gravar e ler. Inicializamos para 256 bytes, é apenas um valor arbitrário, dependendo da quantidade de dados que planejamos transferir para lá e para cá.

Por fim, realizamos o processo de seleção. Selecionamos os canais prontos, recuperamos suas chaves de seleção, iteramos sobre as chaves e executamos as operações para as quais cada canal está pronto.

Fazemos isso em um loop infinito, pois os servidores geralmente precisam continuar em execução, haja uma atividade ou não.

A única operação que um ServerSocketChannel pode manipular é uma operação ACEITAR. Quando aceitamos a conexão de um cliente, obtemos um objeto SocketChannel no qual podemos ler e escrever. Nós o configuramos para o modo sem bloqueio e o registramos para uma operação READ no seletor.

Durante uma das seleções subsequentes, este novo canal ficará pronto para leitura. Nós o recuperamos e lemos seu conteúdo no buffer. Como é um servidor de eco, devemos escrever esse conteúdo de volta para o cliente.

Quando desejamos escrever em um buffer do qual estivemos lendo, devemos chamar o método flip ().

Finalmente configuramos o buffer para o modo de gravação chamando o método flip e simplesmente gravando nele.

O método start() é definido para que o servidor de eco possa ser iniciado como um processo separado durante o teste de unidade.

### 8.2. O cliente

Aqui está nosso código para EchoClient.java:

```
public class EchoClient {
    private static SocketChannel client;
    private static ByteBuffer buffer;
    private static EchoClient instance;

    public static EchoClient start() {
        if (instance == null)
            instance = new EchoClient();

        return instance;
    }

    public static void stop() throws IOException {
        client.close();
        buffer = null;
    }

    private EchoClient() {
        try {
            client = SocketChannel.open(new InetSocketAddress("localhost", 5454));
            buffer = ByteBuffer.allocate(256);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public String sendMessage(String msg) {
        buffer = ByteBuffer.wrap(msg.getBytes());
        String response = null;
        try {
            client.write(buffer);
            buffer.clear();
            client.read(buffer);
            response = new String(buffer.array()).trim();
            System.out.println("response=" + response);
            buffer.clear();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return response;

    }
}
```

O cliente é mais simples que o servidor.

Usamos um padrão singleton para instanciá-lo dentro do método estático de início. Chamamos o construtor privado deste método.

No construtor privado, abrimos uma conexão na mesma porta em que o canal do servidor foi vinculado e ainda no mesmo host.

Em seguida, criamos um buffer no qual podemos escrever e a partir do qual podemos ler.

Finalmente, temos um método sendMessage que lê quebra qualquer string que passamos a ele em um buffer de bytes que é transmitido pelo canal para o servidor.

Em seguida, lemos no canal do cliente para obter a mensagem enviada pelo servidor. Nós retornamos isso como o eco de nossa mensagem.

### 8.3. Testando
Dentro de uma classe chamada EchoTest.java, vamos criar um caso de teste que inicia o servidor, envia mensagens para o servidor e só passa quando as mesmas mensagens são recebidas de volta do servidor. Como uma etapa final, o caso de teste para o servidor antes da conclusão.

Agora podemos executar o teste:

```
public class EchoTest {

    Process server;
    EchoClient client;

    @Before
    public void setup() throws IOException, InterruptedException {
        server = EchoServer.start();
        client = EchoClient.start();
    }

    @Test
    public void givenServerClient_whenServerEchosMessage_thenCorrect() {
        String resp1 = client.sendMessage("hello");
        String resp2 = client.sendMessage("world");
        assertEquals("hello", resp1);
        assertEquals("world", resp2);
    }

    @After
    public void teardown() throws IOException {
        server.destroy();
        EchoClient.stop();
    }
}
```

# 9. Selector.wakeup()

Como vimos antes, chamar selector.select() bloqueia o thread atual até que um dos canais assistidos esteja pronto para operação. Podemos substituir isso chamando selector.wakeup() de outro thread.

O resultado é que o thread de bloqueio retorna imediatamente, em vez de continuar a esperar, quer um canal esteja pronto ou não.

Podemos demonstrar isso usando um CountDownLatch e rastreando as etapas de execução do código:

```
@Test
public void whenWakeUpCalledOnSelector_thenBlockedThreadReturns() {
    Pipe pipe = Pipe.open();
    Selector selector = Selector.open();
    SelectableChannel channel = pipe.source();
    channel.configureBlocking(false);
    channel.register(selector, OP_READ);

    List<String> invocationStepsTracker = Collections.synchronizedList(new ArrayList<>());

    CountDownLatch latch = new CountDownLatch(1);

    new Thread(() -> {
        invocationStepsTracker.add(">> Count down");
        latch.countDown();
        try {
            invocationStepsTracker.add(">> Start select");
            selector.select();
            invocationStepsTracker.add(">> End select");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }).start();

    invocationStepsTracker.add(">> Start await");
    latch.await();
    invocationStepsTracker.add(">> End await");

    invocationStepsTracker.add(">> Wakeup thread");
    selector.wakeup();
    //clean up
    channel.close();

    assertThat(invocationStepsTracker)
      .containsExactly(
        ">> Start await",
        ">> Count down",
        ">> Start select",
        ">> End await",
        ">> Wakeup thread",
        ">> End select"
    );
}
```

Neste exemplo, usamos a classe Pipe do Java NIO para abrir um canal para fins de teste. Nós rastreamos as etapas de execução do código em uma lista de thread-safe. Ao analisar essas etapas, podemos ver como selector.wakeup() libera a thread bloqueada por selector.select().

# 10. Conclusão
Neste artigo, cobrimos o uso básico do componente Seletor do Java NIO.