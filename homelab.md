# HomeLab do Tenorio
## Instalando Ubuntu Server 26.04
Não usei a versão minimalista. Não instalei nada alem do sistema, nem SSH. Faremos tudo manualmente.
Instalar com rede conectada facilita bastante a vida. Na primeira trouxe mais problemas como não obter IPV4 da rede, por isso tive que mexer no arquivo yaml de configuração do netplan (/etc/netplan/00-installer-config.yaml) além de forçar a captura de um IPV4 do Linux

## Tamanho da fonte no terminal
Ok, talvez apos a instalação, você queira aumentar a fonte do terminal. No meu caso, meu monitor é fullhd e a fonte de uma certa distância fica ininteligível.

Então execute o comando: 
* `sudo nano /etc/default/console-setup`

Vai abrir um arquivo de configuração. Lá tem o `FONTSIZE="8x16"`. Confesso que não sei o que significa esse numero, se é pixels ou outra forma de se usar uma fonte.
Vou alterar para `16x32`

  

**IMPORTANTE:** Utiliza `Ctrl+O` e `Enter` para salvar o arquivo. E `Ctrl+X` para sair do editor.

* `sudo update-initramfs -u`
 (`-u` significa update, assim como o `-c` é create)

Execute para aplicar as mudanças
* `sudo reboot`

  
## SSH 

* Para atualizar tudo do seu sistema: `sudo apt upgrade` 
* Para instalar o SSH use: `sudo apt install -y openssh-server`

  `-y`=> Aceita automaticamente todos os avisos
* Para habilitar o serviço, utilize: `sudo systemctl enable ssh`
* Para executar o serviço utilize: `sudo systemctl start ssh`
* Para executar os dois comandos acima de uma vez utilize: `sudo systemctl enable --now ssh`
* Para configurar o firewall para uso do ssh use:<br>
`sudo ufw allow ssh` => O retorno foi: Rules update - Rules update(v6)<br>
`sudo ufw reload` => O retorno foi: Firewall not enabled (skipping reload)<br>

Com o SSH, é possivel fazer a gestão do servidor remotamente.
Com isso, facilita um pouco minha vida ja que o PC e o monitor estão atras de mim.  

No seu PC você irá precisar do ipv4 do servidor, para consulta use o comando:
* `ip address`

Pode-se usar a abreviação que seria: `ip a`

No meu caso o ip é dinâmico, e isso é um problema, pois toda vez teremos que consultar no servidor.
Para deixar o ip estático iremos manipular o arquivo /etc/netplan/00-installer-config.yaml
Para isso execute o comando:
* `sudo nano /etc/netplan/00-installer-config.yaml`

  

Dessa forma o editor irá abrir. Farei as seguintes modificações:

**ANTES**
```
network:  
  ethernets:  
    nome_interface:  
      dhcp4: true  
  dhcp6: true  
  match:  
      macadress: ...  
    setname: nome_interface  
  version: 2
```

**DEPOIS**
```
network:
 ethernets:
  nome_interface:
   dhcp4: false
   dhcp6: false
   addresses:
    - 192.168.1.60/24
   routes:
    - to: default
      via: 192.168.1.1
  match:
   macaddress: ...
 setname: nome_interface
version: 2
```
**IMPORTANTE(1):** Não use tabulação nessa bomba, somente espaços.

**IMPORTANTE(2):** Utiliza `Ctrl+O` e `Enter` para salvar o arquivo. E `Ctrl+X` para sair do editor.

Para validar a formatação do aquivo, use:
* `sudo netplan try`

E se tiver tudo ok, utilize o comando:
* `sudo netplan apply`

No caso ao aplicar, deixei estático o ip: **192.168.1.60**
Para conectar seu PC ao servidor utilize:
* `ssh usuario_do_servidor@ip_que_configuramos`

 Dessa forma ele irá pedir a senha do usuario do servidor, informe e dê Enter.
Provavelmente deu bom. Se der um aviso, deleta o arquivo known_hosts na sua pasta de usuario => /home/seu_usuario_pc/known_hosts
Importante. Tem que ter o SSH instalado no PC também, achei que fosse implicito e não mencionei antes.

 Agora faremos a configuração de chave pública e chave privada,
de maneira que não necessitaremos mais de login e senha para conectar ao servidor.
Como não sei fazer isso, vou aprender e transcrever o processo para cá.
  
Ok, na pasta /home/usuario do servidor e do PC deve existir a pasta .ssh; Caso não exista utilize:
* `mkdir -p /home/usuario/.ssh` => -p ele cria todas as pastas que não existe até caminho informado no caso criaria somente a .ssh

No servidor, caso não exista, crie o arquivo authorized_keys
* `touch authorized_keys`

No PC, execute o seguinte comando:
* `ssh-keygen -t ed25519`

`-t` serve pra especificar o tipo de criptografia, 

`ed25519` [Criptografia de curva elíptica](https://pt.wikipedia.org/wiki/Criptografia_de_curva_el%C3%ADptica) é o padrão de criptografia utilizado

Logo em seguida vai ter a opção de alterar o nome do arquivo gerado, tanto o publico quanto privado.
De deixe o padrão, não escrevi nada, logo ele criou dois arquivos. O **id_ed25519.pub** e o **id_ed25519**.
O primeiro como indica a extensão, é a chave pública e o segundo, a chave privada.

Depois você irá digitar o comando:
* `ssh-copy-id nome_usuario_servidor@ip_do_servidor`

Se der certo a conexão, ele irá pedir a senha do usuario do servidor.
Para validar se funcionou, olhamos o arquivo authorized_keys no servidor.
Ao olha o arquivo, observaremos o tipo de criptografia usada, a chave publica e o usuario_pc@pc
  
Pra finalizar o teste, fechamos todos os terminais no cliente e em seguida testamos a conexão usando:
* `ssh nome_usuario_servidor@servidor`

Conectando sem pedir a senha, nossa configuração deu certo.

Então se eu entendi corretamente, geramos uma chave publica e privada no cliente, cadastramos essa chave publica no servidor.
Pelo o que a IA do Google disse(to me sentindo um imbecil inutil mas to aprendendo pelo menos), a chave privada é utilizada para
validar se a chave publica cadastrada no servidor é mesmo minha. É valido pegar um fluxograma disso ocorrendo.
No futuro é valido se aprofundar em cada comando usado.

## Segurança no SSH
Desabilitando uso de senha no SSH, login apenas por chave publica/privada

`sudo nano /etc/ssh/sshd_config`

Alterei os seguinte parametros para:

```
PubkeyAuthentication yes
PasswordAuthentication no
UsePAM no
```
No Ubuntu o arquivo `/etc/ssh/sshd_config.d/*.conf` tem precedência. 
No meu caso como instalei depois, entendo que não preenchou nada.
Pelo menos o ls -la não trouxe nada alem de `. ..`

Execute o comando para reiniciar o serviço do ssh: `sudo /etc/init.d/ssh reload`

Instalei o app SSH Terminal Client da Tower App Inc no meu celular.
Ao informar a senha e tentar, autenticação falhou, como esperado.

## Copia de arquivos via SSH

Vamos lá, quebrei bastante cabeça com isso e era por bobagem.
Meu objetivo é criar uma pasta no servidor chamada midia, dentro contem as pastas filmes e series.
Minha ideia é copiar o conteudo da minha maquina, no caso um filme, para essa pasta.

Como ja fiz, vou deletar tudo que foi feito e iniciamos aqui do zero.
Então no servidor, vamos executar o comando

`sudo mkdir -p /usr/midia/{filmes,series}`

Com o -p ele garante que toda pasta inexiste nesse caminho seja criada.
No caso criamos a pasta midia, e dentro dela duas pastas que falamos antes.

Agora temos que garantir a permissão para gravar(foi aqui que eu estava "burrando")
Primeiro vamos executar um `ls -la`. 
Esse comando nos mostra as pastas e quem tem permissao de alterar.
Vamos executar: `sudo chown -R usuario:grupo /usr/midia`
No caso o `-R` garante que vai afetar tudo que está dentro da pasta midia

Depois executamos novamente o comando `ls -la` para averiguarmos que deu certo

Agora a hora da verdade. Vamos copiar do nosso PC ao servidor.
Utilizaremos o comando `scp`. Na minha pasta de Videos no PC pessoal, organizei uma pasta
de filmes e outra de series, Infelizmente não tenho nenhuma série então vamos de filmes.

Executaremos o seguinte comando e logo vamos destrincha-lo

`scp -r -v /home/usuario/Vídeos/filmes/* usuario@ip_servidor:/usr/midia/filmes/`

`-r` => implica que ele irá copiar recursivamente todo o contéudo existente na pasta origem que definimos

`-v` => Aqui habilita o verbose.

`/home/usuario/Vídeos/filmes/*` => Pasta origem, estamos copiando todo o conteudo de filmes. O * garante que 
pegaremos todo o conteudo mas não a pasta filmes em si, se não ficaria duplicado no servidor.

`usuario@ipservidor:[porta]` => Aqui é o que vimos anteriormente. No caso podemos informar a porta, mas é a padrão(22) logo não precisa

`/usr/midia/filmes/` => Nesse ponto informamos a pasta destino dentro do servidor.

Aqui funfou, coisa boa. E invertendo a origem e o destino conseguimos copiar do servidor para a nossa maquina local.

## Testando RSYNC

Vamos tentar refazer o mesmo processo porem, com outro comando. Deletei os arquivos do servidor.

`rsync -avP /home/usuario/Vídeos/filmes/* usuario@ip_servidor:/usr/midia/filmes/`

`-a` => Preserva todas as propriedades do arquivo

`-v` => Verbose... n precisa de explicação.

`-P` => Mostra o progresso da execução

Funcionou perfeitamente.

## Docker
Seguindo o tutorial funcionou tranquilamente => https://docs.docker.com/engine/install/ubuntu/

lspci | grep -i network // TODO

# Linux Básico + Certificação Linux
'usuario@maquina:~$'

usuario => indica o nome do usuario atual. meio óbvio mas...

maquina => Nome da maquina que está utilizando

~ => Indica o caminho no qual usuario se encontra, no caso ~ significa /home/usuario. No caso do root seria /root

$ => Indica que o usuário não tem privilégio de super usuário e '#' indica que o usuário tem privilégio de super usuário

---------------------------
