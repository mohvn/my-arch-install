# my-arch-install
📔 A little guide to install the Arch Linux for me

## Keyboard 
First, check if my keyboard is in the correct layout, since by default the Arch installation comes with the keyboard in en-US layout.

To change the layout, use the following command:

```bash
loadkeys br-abnt2
```

## Internet Connection
I usually use my computer directly wired, but there are some cases where I will need to use Wi-Fi. In that case, I use the [iwd](https://wiki.archlinux.org/title/iwd) package.

To use it, start with the command:

```bash
iwctl

```
Find out the name of the Wi-Fi network device on your computer with the command:

```
[iwd]# device list
```

<div align="center">
  <img src="https://i.imgur.com/GGxULsZ.png">
</div>

Now that we know our device is called wlan0, we use the following command to scan for nearby networks:

```
[iwd]# station wlan0 get-networks
```

Now to connect to a network, use:

```
[iwd]# station wlan0 connect REDE-2.4G
```

Enter the password and the Wi-Fi will be connected.

To check, just ping any website.

## Particionamento de disco
First, run the **fdisk** command to list the partitions:

```bash
fdisk -l
```

It will list all the storage devices on the computer, the disk that is normally used is **/dev/sda**

<div align="center">
  <img src="https://i.imgur.com/62SPwt3.png">
</div>

Agora para particionar o disco utilizamos a ferramenta de **cfdisk** por ser de fácil utilização.

No meu caso, eu sempre crio 3 partições:
- /dev/sda1 (500MB para o /boot/efi para sistemas UEFI)
- /dev/sda2 (2GB para swap)
- /dev/sda3 (todo o resto para o /)

<div align="center">
  <img src="https://i.imgur.com/JK2LpVA.png">
</div>

Sempre marcando o Type da partição para suas respectivas funções.

Em seguida, faça o cfdisk escrever as partições clicando na indo na opção “Write” e saia indo na opção "Quit"

## Formatando as partições
Após criar todas as partições, vamos formatá-las para os seus respectivos formatos.

Para formatar a partição de boot (/boot/efi)
```bash
mkfs.fat -F 32 /dev/sda1
```
Para criar a partição de Swap
```bash
mkswap /dev/sda2
```
Para formatar a partição raiz do sistema
```bash
mkfs.ext4 /dev/sda3
```

## Montagem do sistema
Após formatar, o próximo passo é montar as partições, para isso inicialmente monto a partição raiz do sistema com:
```bash
mount /dev/sda3 /mnt
```
Logo em seguida, crio o diretório de **/boot/efi** do sistema usando o comando:
```bash
mkdir -p /mnt/boot/efi
```
Monto a partição de boot neste diretório criado
```bash
mount /dev/sda1 /mnt/boot/efi
```
Ativo a partição de Swap
```bash
swapon /dev/sda2
```

# Instalando pacotes de instalação do Arch Linux
Agora iremos instalar os pacotes base e o kernel Linux do Arch além de algumas ferramentas de editores de texto (*vim*) e ferramenta de gerenciamento de rede (*dhcpcd*)
```bash
pacstrap -K /mnt base base-devel linux linux-firmware vim dhcpcd
```
Este é o processo mais demorado da instalação pois depende da velocidade de Download.

Após a instalação, geramos nossa tabela **fstab** que sinaliza para o boot onde estão montadas cada uma das partições com o comando
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## Dentro do sistema
Para entrarmos no ambiente do sistema para fazermos as configurações iremos utilizar o comando
```bash
arch-chroot /mnt
```

Esta parte para mim é opcional porém podemos configurar a data e hora do sistema criando um link simbólico do arquivo de nossa região em nosso localtime.

No meu caso, usando o horário de Brasília seria:
```bash
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime/
```

Próximo passo seria configurar o idioma do sistema, neste caso gosto de deixar o sistema em Inglês, portanto vamos editar o arquivo **locale.gen**
```bash
vim /etc/locale.gen
```
Dentro dele haverá diversas linguagens comentadas com um **#**, logo iremos para o idioma que queremos e *descomentamos* o idioma que queremos removendo o **#** do começo da linha.

No meu caso o **en_US.UTF-8 UTF-8**

<div align="center">
  <img src="https://i.imgur.com/7XZFpq3.png">
</div>

Após isso salve o arquivo e gere a configuração de idioma usando o comando
```bash
locale-gen
```
Agora iremos configurar o nome do computador na rede, para isso edite o arquivo **/etc/hostname** e coloque o nome do computador, no meu caso eu sempre coloco como **arch**

## Configurando usuários

Primeiramente, configure a senha do usuário **root** (muito importante)
```bash
passwd
```

Crie o seu usuário, irei usar de exemplo o meu username *mdxv* com o comando
```bash
useradd -m -g users -G wheel -s /bin/bash mdxv
```
Em seguida defina uma senha para esse usuário
```bash
passwd mdxv
```

Agora iremos instalar alguns pacotes úteis para logo em seguida fazer a instalação do GRUB
```bash
pacman -S dosfstools os-prober mtools dialog grub efibootmgr sudo
```

Caso eu tenha feito a instalação com rede Wi-Fi, talvez seja necessário eu instalar esses programas adicionais
```bash
pacman -S wpa_supplicant wireless_tools
```

## Instalação do GRUB

Para instalar o GRUB no meu sistema que utiliza o sistema UEFI, instale o grub com o comando
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck
```
E gere o arquivo de configuração do GRUB
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Caso eu esteja instalando o GRUB em um sistema BIOS-Legacy, irá ser os seguintes comandos
```bash
grub-install --target=i386-pc --recheck /dev/sda
```
E gere o arquivo de configuração
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Por fim, reinicie o sistema e se tudo der certo, ele irá iniciar com GRUB

<div align="center">
  <img src="https://i.imgur.com/5NEF1vW.png">
</div>

## Pós-instalação

Após iniciar o sistema, logamos em nosso usuário e veremos que não possuímos permissão de utilizar o comando sudo, pois temos que adicionar o usuário no arquivo *sudoers* e realizar a configuração de rede com o *dhcpcd*

Para isso precisamos logar como *root* com o comando
```bash
su
```
Digite a senha do usuário root.

Agora iremos editar o arquivo *sudoers*, para isso utilizaremos o comando *visudo* definindo o editor de texto com o **vim**
```bash
EDITOR=vim visudo
```
Procure pela linha `%wheel ALL=(ALL:ALL) ALL` descomente removendo a “#” da frente do texto

agora saia do usuário root e verifique se já é possível utilizar o comando root já utilizando o comando do **dhcpcd** para configurarmos a Internet.
```bash
sudo dhcpcd
```

E pronto, o Arch Linux está instalado e configurado, a partir daí é só configurar o seu sistema do jeito que você quiser.