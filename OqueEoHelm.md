Nós, humanos, tendemos a lutar com a complexidade. E os aplicativos que implantamos em nosso cluster Kubernetes podem se tornar muito complicados. Um aplicativo típico geralmente é composto por uma coleção de objetos que precisam se interconectar para que tudo funcione. Por exemplo, mesmo um site WordPress relativamente simples pode precisar do seguinte:

Um volume persistente para armazenar o banco de dados
Uma reivindicação de volume persistente
Um serviço para expor o servidor da Web em execução em um pod para a Internet
Um segredo para armazenar a senha de administrador
Uma implantação para declarar os pods que você deseja executar: servidores de banco de dados MySQL, servidores da Web etc.
E talvez ainda mais se você quiser coisas extras como backups periódicos e assim por diante.

Para cada objeto, podemos precisar de um arquivo .yaml separado. Então precisamos “kubectl apply -f” em cada arquivo .yaml. Isso pode ser tedioso, mas não é o fim do mundo. Mas imagine que baixamos esses arquivos yaml da Internet. Não estamos satisfeitos com os padrões, então começamos a mudar as coisas. Os volumes persistentes são de 20 GB, mas sabemos que nosso site precisará de muito mais armazenamento. Vamos para os arquivos .yaml onde os volumes/declarações persistentes são declarados, mudamos de 20 para 100. Mais coisas que queremos mudar? Teremos que abrir cada arquivo yaml e editar cada um de acordo com nossas necessidades. Ainda não está ruim o suficiente? Agora imagine 2 meses se passando. Agora temos que atualizar alguns componentes em nosso aplicativo. Voltando à edição de várias declarações yaml, com muito cuidado, para não alterar a coisa errada no lugar errado. Precisa excluir o aplicativo? Precisamos lembrar cada objeto que pertence ao nosso aplicativo e excluí-los todos, um por um. Isso se traduziria em muitos comandos como:

```bash
kubectl delete -f wordpress-service.yaml
kubectl delete -f wordpress-deployment.yaml
kubectl delete -f wordpress-pvc.yaml
```

Mas algumas pessoas podem estar pensando: “Ei, não é grande coisa, podemos simplesmente escrever todas as declarações de objetos em um único arquivo yaml”. Isso é verdade, mas pode tornar ainda mais difícil encontrar o material que estamos procurando. Teríamos que procurar continuamente por coisas que precisamos editar em algo que poderia ser 25 páginas de texto. Pelo menos em vários arquivos, eles seriam um pouco organizados e saberíamos que encontraríamos coisas relacionadas à implantação em “mysql-deployment.yaml”.

Helm muda o paradigma. O Kubernetes realmente não se importa com nosso aplicativo como um todo. Tudo o que ele sabe é que declaramos vários objetos e ele faz cada um deles existir em nosso cluster. Ele realmente não sabe que esse volume persistente e essa implantação e esse segredo e esse serviço fazem parte de um grande aplicativo chamado WordPress. Ele olha todos os pedacinhos que o administrador queria ter no cluster e cuida de cada um, individualmente.

Helm, no entanto, é construído desde o início para saber sobre essas coisas. É por isso que às vezes é chamado de gerenciador de pacotes para Kubernetes. Ele olha para esses objetos como parte de um grande pacote, como um grupo. Sempre que precisamos realizar uma ação, não dizemos ao Helm os objetos que ele deve tocar. Apenas informamos em qual pacote queremos que ele atue, como nosso aplicativo/pacote WordPress. Com base no nome do pacote, ele sabe quais objetos devem ser alterados e como, mesmo se houver 100 objetos pertencentes a esse pacote.

Para tornar isso mais fácil de entender, pense sobre isso. Um jogo de computador está contido em centenas ou milhares de arquivos. Existem alguns arquivos com o código executável do programa, outros arquivos com áudio, sons de jogos e músicas, outros arquivos com gráficos, texturas, imagens, arquivos com dados de configuração e assim por diante. Agora imagine que teríamos que baixar cada um deles separadamente. Uau, isso seria tedioso. Felizmente, não temos que passar por tais horrores, pois temos um instalador de jogos. Nós rodamos, apertamos “next, next, next”, escolhemos o diretório onde queremos instalar, configurações do jogo, e o instalador faz o resto, colocando milhares de arquivos em seus devidos locais. O Helm faz algo semelhante (e mais) para os arquivos yaml e objetos Kubernetes que compõem nosso aplicativo.

Assim, temos vantagens como estas:

Usamos um único comando para instalar todo o nosso aplicativo, mesmo que precise de 100 objetos. Em seguida, o Helm adiciona automaticamente todos os objetos necessários ao Kubernetes sem nos incomodar com os detalhes.
Usamos um único comando para desinstalar nosso aplicativo. Ele mantém o controle de todos os objetos usados ​​por cada aplicativo para saber o que remover. Não precisamos mais lembrar de cada objeto que pertence a um de nossos aplicativos ou usar 10 comandos separados para remover tudo. O leme faz todo o trabalho.
Podemos personalizar as configurações que desejamos para nosso aplicativo/pacote, especificando os valores desejados no momento da instalação. Mas, em vez de editar vários valores em vários arquivos .yaml, temos um único local onde podemos declarar todas as configurações personalizadas. Em um arquivo como “values.yaml”, podemos alterar o tamanho de nossos volumes persistentes, escolher o nome do nosso site WordPress, a senha do administrador, as configurações do mecanismo de banco de dados e assim por diante.
Podemos atualizar nosso aplicativo com um único comando. O Helm saberá quais objetos individuais precisam mudar para que isso aconteça.
Podemos reverter para a chamada "revisão" anterior. Por exemplo, se atualizarmos um pacote/aplicativo Helm, isso pode nos levar da revisão número 4 para a revisão número 5. Se a atualização não funcionar como esperamos, talvez tenhamos novos bugs que não gostamos, podemos reverter (reverter) para a revisão anterior e voltar de 5 para 4. Embora semelhante, isso não deve ser confundido com a restauração de um backup. Ele não recupera dados antigos. Por exemplo, se excluirmos acidentalmente um banco de dados, isso não o recuperará. As reversões do Helm são apenas uma maneira de fazer com que os objetos Kubernetes do nosso aplicativo voltem ao estado exato em que estavam antes, em uma revisão anterior.
Também podemos usar o Helm para automatizar instalações. Por exemplo, imagine o seguinte cenário. Um usuário faz uma solicitação em nosso site, ele escolhe algumas configurações. Essas configurações são passadas para um comando do Helm que inicia um aplicativo de acordo com as preferências do usuário.
Assim, o Helm funciona como uma espécie de assistente de instalação (ou gerenciador de pacotes), assistente de desinstalação, mas também como um gerenciador de atualização/reversão (gerenciador de lançamento). O principal é que ele nos permite tratar nossos aplicativos Kubernetes como aplicativos, em vez de uma coleção de objetos. Isso tira um fardo enorme de nossos ombros, pois não precisamos mais microgerenciar cada objeto do Kubernetes, o Helm pode fazer isso por nós