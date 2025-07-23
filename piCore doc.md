## Pré-instalação

### 0. Observações importantes

- É recomendado fazer essa instalação (flashear o cartão SD e configuração do boot) a partir de um sistema UNIX, preferencialmente Linux. Recomendo fazer a partir de outro RasPI.

* A primeira configuração necessita acesso à internet via Ethernet, portanto é recomendável bootar pela primeira vez num RasPI 4 cabeado. Ou então, é possível baixar os drivers manualmente e colocá-los na pasta correta, porém é necessário estar num ambiente UNIX e dá trabalho.
* O sistema bootará diretamente no BoxERP, em primeiro plano, e quando necessário o gerenciador de arquivos e o leitor de PDFs abrirão por cima. Não terá ambiente desktop, apenas terminal. A navegação entre ERP, PDFs e explorer será com mouse.
* É possível basicamente tirar todos os acessos aos usuários, evitando problemas corriqueiros causados pelo RasPI.
* O sistema conterá: GLPI-Agent, BoxERP, File Explorer básico e leitor de PDFs básico.
* **Importante**: O piCore não salva nada na memória flash (SD) a não ser que especificado. Portanto, há a possibilidade de fazer arquivos PDF, logs e afins deletarem-se após o reboot. Estes arquivos ficam salvos na memória RAM e/ou swap, que é resetada no reboot.

### 1. Baixando e flasheando a imagem

A imagem utilizada nessa instalação é a piCore64-16.0.0.img.gz, e o tutorial funciona especificamente para ela. Algumas etapas diferem com a versão do kernel, e baixar outra versão pode causar problemas.

Baixe a imagem do site e então extraia-a. Todos os comandos na documentação presumem estar sendo executados pelo usuário root, e para acedê-lo, deve preceder todo comando com `sudo`. **Não** é possível usar ``su -`` ou ``su root``, pois o usuário root ainda não está configurado.

``````bash
cd Downloads
gunzip -d piCore64-16.0.0.img.gz
``````

Para flashear a imagem, pode-se usar o comando `dd`. Antes limparemos o cartão. 
``````bash
# Listando partições para selecionar a do cartão SD.
fdisk -l

# Supondo que a partição do SD seja /dev/sdx:
fdisk /dev/sdx
> d [enter] # Pressione enter até printar ".......". Repita até não ter mais partições.
> w

# Flasheando o cartão
cd ~LOCALIZAÇÃO_DO_PICORE_IMG~
dd if=piCore64-16.0.0.img of=/dev/sdx bs=1M status=progress conv=fsync
fdisk -l # Certifique-se de que existem duas partições em /dev/sdx
``````

Para possibilitar a instalação do BoxERP e outros programas necessários, devemos aumentar a partição de dados do piCore. Se tiver conectado via Ethernet, já é possível usar SSH, que vem embutido. Basta usar `ifconfig`, e conectar no IP do eth0.

``````bash
fdisk -l
fdisk /dev/sdx # Suponhando que o SD seja /dev/sdx

# Localize a partição com Type = Linux, e delete-a
> d [enter]
> 2 [enter]
> Confirme

# Crie uma partição iniciando-se no exato bloco que iniciava a deletada.
> n [enter]
> p [enter]
> 2 [enter]
> ~bloco que iniciava a partição deletada~ [enter]
> 10000000 [enter] (dez milhões)
> Confirme

# Cheque que está tudo correto
> p [enter]

# Se estiver tudo correto (a partição tipo Linux deve conter por volta de 4GB de dados)
> w [enter]

# Aumentar o filesystem
resize2fs -p /dev/sdx
``````



## Pós-instalação e configuração inicial

Antes de mais nada, montaremos o diretório para os tces (pacotes):

```bash
sudo mkdir -p /mnt/mmcblk0p2/tce
sudo mount /dev/mmcblk0p2 /mnt/mmcblk0p2
export TCE=/mnt/mmcblk0p2/tce
```

Para a primeira conexão, será necessário utilizar um Raspberry Pi 4, com conexão cabeada via Ethernet, e liberada no Firewall. Baixaremos drivers necessários para o Wi-Fi e outras funcionalidades básicas.

``````bash
tce-load -wi wifi                 # Controle de Wi-Fi
tce-load -wi firmware-rpi-wifi    # Firmware do Wi-Fi
tce-load -wi wireless-$(uname -r) # Drivers Wi-Fi
tce-load -wi wireless_tools       # Configurações como wiconfig and ip
tce-load -wi wpa_supplicant       # Necessário para conexões via WPA/WPA2
tce-load -wi libnl                # Dependência de wpa_supplicant
tce-load -wi openssl              # Dependência de wpa_supplicant
tce-load -wi openssh			  # Necessário para algumas funções
tce-load -wi ca-certificates      # Dependência de openssl
tce-load -wi dhcpcd				  # DCHP
tce-load -wi kmap                 # Necessário para trocar teclado para br-abnt2

# Certifique que todos os módulos em optional estejam carregados em onboot.
# Todos os arquivos *.tcz devem estar lá. Não adicionar .tcz.dep ou .tcz.md5.txt.
ls -l /mnt/mmcblk0p2/tce/optional/
sudo vi /mnt/mmcblk0p2/tce/onboot.lst
``````

Ativar o wifi e carregar o layout de teclado correto:

```bash
sudo wifi.sh
> Logar na rede wifi

ifconfig # Conferir se puxou IP


sudo nano /opt/bootlocal.sh
# Adicione ao fim do arquivo:
loadkmap < /usr/share/kmap/qwerty/br-abnt2.kmap (ou br-abnt.kmap)

# E então execute:
filetool.sh -b
```
