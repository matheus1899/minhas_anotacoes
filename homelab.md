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

## Servidor de arquivo/web/banco

Usando o comando `lspci`, irá listar tudo que está conectado à maquina:

Output:

```
00:00.0 Host bridge: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor DRAM Controller (rev 09)
00:02.0 VGA compatible controller: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor Graphics Controller (rev 09)
00:14.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB xHCI Host Controller (rev 04)
00:16.0 Communication controller: Intel Corporation 7 Series/C216 Chipset Family MEI Controller #1 (rev 04)
00:16.3 Serial controller: Intel Corporation 7 Series/C210 Series Chipset Family KT Controller (rev 04)
00:19.0 Ethernet controller: Intel Corporation 82579LM Gigabit Network Connection (Lewisville) (rev 04)
00:1a.0 USB controller: Intel Corporation 7 Series/C216 Chipset Family USB Enhanced Host Controller #2 (rev 04)
00:1b.0 Audio device: Intel Corporation 7 Series/C216 Chipset Family High Definition Audio Controller (rev 04)
00:1d.0 USB controller: Intel Corporation 7 Series/C216 Chipset Family USB Enhanced Host Controller #1 (rev 04)
00:1e.0 PCI bridge: Intel Corporation 82801 PCI Bridge (rev a4)
00:1f.0 ISA bridge: Intel Corporation Q77 Express Chipset LPC Controller (rev 04)
00:1f.2 SATA controller: Intel Corporation 7 Series/C210 Series Chipset Family 6-port SATA Controller [AHCI mode] (rev 04)
00:1f.3 SMBus: Intel Corporation 7 Series/C216 Chipset Family SMBus Controller (rev 04)
```

Aqui observamos que temos somente uma interface de conexão com a rede

Usando o comando `lspci -s 00:19.0 -v`, o `-s` filtra pelo dominio e o `-v` é o verbose que traz mais informações que podem ser pertinentes.

Output:
```
00:19.0 Ethernet controller: Intel Corporation ... Gigabit Network Connection (Lewisville) (rev 04)
        DeviceName:  Onboard LAN
        Subsystem: Lenovo
        Flags: bus master, fast devsel, latency 0, IRQ 26
        Memory at f7c00000 (32-bit, non-prefetchable) [size=128K]
        Memory at f7c39000 (32-bit, non-prefetchable) [size=4K]
        I/O ports at f080 [size=32]
        Capabilities: <access denied>
        Kernel driver in use: e1000e
        Kernel modules: e1000e
```

Usando `ip address` ou `ip addr` ou ainda `ip a` nos informa sobre os ip da maquina:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fc:4d:d4:4d:48:e5 brd ff:ff:ff:ff:ff:ff
    altname enp0s25
    altname enxfc4dd44d48e5
    inet 192.168.1.60/24 brd 192.168.1.255 scope global eno1
       valid_lft forever preferred_lft forever
    inet6 2804:214:800d:645f:fe4d:d4ff:fe4d:48e5/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 195505sec preferred_lft 109105sec
    inet6 fe80::fe4d:d4ff:fe4d:48e5/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
```
O lo, ou loopback é permite que um cliente no mesmo host consiga se comunicar, usando protocolo TCP/IP, internamente. Ele utiliza a ip 127.0.0.1 com máscara de rede 8. Essa máscará equivala à 255.0.0.0.

Já o eno1 corresponde à interface fisica do computador.

`link/ether` significa a endereço fisico da placa de rede. Essa informação é mais conhecida como MAC Address.
`inet` é o ip da maquina na sua rede local. No meu caso a máscara é 24 que equivale à 255.255.255.0.



Executando o comando `ifdown nome_interface` você irá reiniciar as configurações de conexão, sem necessariamente precisar de reiniciar a máquina.

Já executando o comando `ifup nome_interface`, ele irá buscar um novo IP no servidor DHCP do seu modem. No meu caso, o ip é estático.

# Linux Básico + Certificação Linux
'usuario@maquina:~$'

usuario => indica o nome do usuario atual. meio óbvio mas...

maquina => Nome da maquina que está utilizando

~ => Indica o caminho no qual usuario se encontra é o padrão do usuário, no caso ~ significa /home/usuario. No caso do root seria /root.

$ => Indica que o usuário não tem privilégio de super usuário e '#' indica que o usuário tem privilégio de super usuário

`mkdir caminho/nome_da_pasta` => Criando diretorio no caminho especificado, com o nome especificado

`rm -r caminho/pasta` => Deleta arquivos e pastas, o -r significa que vai trabalhar recursivamente, deletando o conteudo das pastas.

`ls` => Lista arquivos e diretorios

`ls -l` => Lista arquivo e diretorios com mais detalhes. Dica, se começa com d é diretorio, se começa com - é arquivo e se começa com l é link simbolico.

drwxr-xr-x => Diretorio

-rwxr-xr-x => Arquivo

lrwxr-xr-x => Link simbólico

`ls -a` => Mostra tudo que está na pasta, inclusive arquivos e pastas escondidas (nome começa com .)

`mv` => Mover e renomear arquivo

`mv nome_atual nome_novo` => Renomeando arquivo

`cp` => Copia um arquivo

`cp arquivo arquivo.bkp` => Criando uma cópia do arquivo

## Alterando mensagem inicial do servidor

`sudo apt install linuxlogo`

Com esse comando, vai ser criado na pasta /etc o issue.linuxlogo

Vamos fazer o backup do issue original

`sudo cp /etc/issue /etc/issue.bkp`

Depois vamos usar o comando:

`cat /etc/issue.linuxlogo > /etc/issue`

Ele copiou o conteudo do issue.linuxlogo para o issue. Vamos validar:

`cat /etc/issue`

Depois, vamos reiniciar
`sudo reboot`

## systemd

`systemd-analyze` => Nos dá a informação de inicialização da maquina, em tempo e separado por processo
Output: 
```
Startup finished in 26.723s (firmware) + 4.172s (loader) + 654ms (kernel) + 5.937s (initrd) + 2min 13.701s (userspace) = 2min 51.188s`
```

Para uma consulta ainda mais detalhada use:
`systemd-analyze blame`

Output:
```
2min 43ms systemd-networkd-wait-online.service
  11.538s dev-disk-by\x2ddiskseq-9\x2dpart3.device
  11.538s dev-disk-by\x2dpath-pci\x2d0000:00:1f.2\x2data\x2d1.0\x2dpart-by\x2dpartnum-3.device
  11.538s dev-disk-by\x2dpartuuid-d2da897f\x2d63ea\x2d41c6\x2d8036\x2d8e31824b0298.device
  11.538s dev-disk-by\x2did-scsi\x2d35000c500601f0294\x2dpart3.device
  11.538s dev-disk-by\x2did-wwn\x2d0x5000c500601f0294\x2dpart3.device
  11.538s dev-disk-by\x2did-ata\x2dST500DM002\x2d1BD142_W2AQKHM6\x2dpart3.device
  11.538s dev-disk-by\x2dpath-pci\x2d0000:00:1f.2\x2data\x2d1\x2dpart3.device
  11.538s dev-sda3.device
  11.538s dev-disk-by\x2dpath-pci\x2d0000:00:1f.2\x2data\x2d1.0\x2dpart-by\x2dpartuuid-d2da897f\x2d63ea\x2d41c6\x2d>
 ...
```

Usando o proximo comando, teremos informações da maquina de forma enxuta:
`hostnamectl`

Output:
```
 Static hostname: servidor-casa
       Icon name: computer-desktop
         Chassis: desktop 🖥
      Machine ID: ...
         Boot ID: ...
Operating System: Ubuntu 26.04 LTS                
          Kernel: Linux 7.0.0-28-generic
    Architecture: x86-64
 Hardware Vendor: Lenovo
  Hardware Model: ThinkCentre
    Hardware SKU: LENOVO
Firmware Version: ...
   Firmware Date: ...
    Firmware Age: ...
```
Usando o comando `systemctl`, obteremos o status de todos os serviços executando na máquina:

Output:
```
UNIT                                                                                                   LOAD     sys-subsystem-net-devices-br\x2dadc6eff47ec6.device                                                    loaded active plugged   /sys/subsystem/net/devices/br-adc6eff47ec6
  sys-subsystem-net-devices-docker0.device                                                               loaded active plugged   /sys/subsystem/net/devices/docker0
  sys-subsystem-net-devices-eno1.device                                                                  loaded active plugged   ... Gigabit Network Connection (Lewisville)
  sys-subsystem-net-devices-vethfdfb7da.device                                                           loaded active plugged   /sys/subsystem/net/devices/vethfdfb7da                                                           
  -.mount                                                                                                loaded active mounted   Root Mount
  boot-efi.mount                                                                                         loaded active mounted   /boot/efi
  boot.mount                                                                                             loaded active mounted   /boot
  dev-hugepages.mount                                                                                    loaded active mounted   Huge Pages File System
  dev-mqueue.mount                                                                                       loaded active mounted   POSIX Message Queue File System
  proc-sys-fs-binfmt_misc.mount                                                                          loaded active mounted   Arbitrary Executable File Formats File System
  run-docker-netns-5b1e2bb83462.mount                                                                    loaded active mounted   /run/docker/netns/5b1e2bb83462
  run-user-1000.mount                                                                                    loaded active mounted   /run/user/1000
  sys-fs-fuse-connections.mount                                                                          loaded active mounted   FUSE Control File System
  sys-kernel-config.mount                                                                                loaded active mounted   Kernel Configuration File System
  sys-kernel-debug.mount                                                                                 loaded active mounted   Kernel Debug File System
  sys-kernel-tracing.mount                                                                               loaded active mounted   Kernel Trace File System
  tmp.mount                                                                                              loaded active mounted   Temporary Directory /tmp
  var-lib-docker-rootfs-overlayfs-0a894d1f8713bb3af3f23e4a660191cd4896087717d494560af837c8c8d76665.mount loaded active mounted   /var/lib/docker/rootfs/overlayfs/0a894d1f8713bb3af3f23e4a660191cd4896087717d494560af837c8c8d76665
...
```

`sudo systemctl status ssh.service` => Pegando status de um serviço

`sudo systemctl stop ssh.service` => Parando um serviço

`sudo systemctl start ssh.service` => Iniciando um serviço

`sudo systemctl restart ssh.service` => Reiniciando um serviço

`sudo systemctl enable ssh.service` => Habilita o serviço para iniciar junto com o sistema

`sudo systemctl disable ssh.service` => Desabilita o serviço para iniciar junto com o sistema

`sudo systemctl is-enable ssh.service` => Valida se o serviço está habilitado iniciar junto com o sistema


## Usando o VI

`i` => Permite a edição de um arquivo. Insere em cima do cursor.

`a` => Também permite a edição. Insere à esquerda do cursor.

`Esc + :` => Ativa o modo de comando

`wq` => "Write and quit", salvar e sair. Necessario estar no modo de comando.

`cat arquivo` => Mostra o conteudo do arquivo de cima para baixo.

`tac arquivo` => Mostra o conteudo do arquivo de baixo para cima. Bom para logs

`Esc + u` => Desfazer

`q!` => Sai do editor sem salvar

`set number` => Mostra o número das linhas






---------------------------
