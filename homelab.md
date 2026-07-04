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

## Docker
Seguindo o tutorial funcionou tranquilamente => https://docs.docker.com/engine/install/ubuntu/

lspci | grep -i network // TODO
