+++
title = "Meu ambiente de trabalho"
+++
Por um tempo eu fiquei usando Linux como meu sistema operacional. Sem dual boot nem nada, só Linux (Ubuntu). Depois de passar um bom tempo usando descobri que, por mais que eu adore o Linux e suas ferramentas, ele só brilha mesmo na linha de comando. Linux para desktop ainda está longe de ser ideal e eu me vi sentindo saudades do Windows, principalmente depois de ter experimentado o Win7, que está excelente.

Tomei a dura decisão de que eu devia voltar para o Windows, mas agora existia um problema. Todo o meu ambiente de trabalho estava focado no Linux e eu precisava arranjar um jeito de conseguir trabalhar com as duas plataformas em conjunto.  Entrei no Forrst e perguntei se era possível criar uma VM e trabalhar nela com as aplicações que eu precisava mas realizar toda a interface através do Windows.  Foi aí que alguém me sugeriu o [Vagrant](http://vagrantup.com/)!

De cara eu me apaixonei pelo Vagrant. Achei a idéia sensacional. Basicamente ela é uma ferramenta de linha de comando que te permite configurar maquinas virtuais com extrema facilidade e usá-las para desenvolver dentro do seu sistema operacional de escolha. Pode ser Windows, OS X, o próprio linux, o que for, bastando ser possivel instalar Ruby e VirtualBox. O legal é que eu poderia reproduzir o servidor virtual que eu realmente vou usar em produção no meu ambiente de desenvolvimento e trabalhar sem me preocupar em adequar os ambientes. O Vagrant me permitia configurar uma pasta local a ser compartilhada na VM e eu poderia fazer todo o desenvolvimento a partir do próprio Windows.

O Vagrant é ótimo! Ainda mais se você aprender a usar [Puppet](http://www.puppetlabs.com/puppet/introduction/) ou [Chef](http://www.opscode.com/chef/) para gerenciar as dependências dos pacotes utilizados na máquina, mas existiam alguns problemas que eu não conseguia resolver. O principal "problema" é que o servidor tinha que usar a porta 8080, ou qualquer outra porta acima de 1024 (acho que é isso), por alguma limitação de não sei o que. Isso acabava me atrapalhando um bocado na hora de trabalhar com URLs a partir do meu próprio sistema operacional uma vez que o meu framework às vezes não levava isso em consideração.

Eu tentei então montar meu próprio Vagrant. Criei uma nova VM no VirtualBox com uma imagem do Ubuntu Server Edition configurando a rede para funcionar em modo bridge. Depois disso editei o arquivo **/etc/network/interfaces** de forma que ele ficasse com o seguinte conteúdo:

    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    auto eth0
    # iface eth0 inet dhcp
    iface eth0 inet static
        address 192.168.1.200
        netmask 255.255.255.0
        network 192.168.1.1
        broadcast 192.168.1.255
        gateway 192.168.1.1

Ao reiniciar a VM agora ela tinha um IP local fixo na minha rede: 192.168.1.200.

Depois eu criei uma pasta compartilhada no VirtualBox e instalei os Add-ons do VirtualBox na VM para poder usar a nova pasta. Criei um arquivo chamado **./mount** com alguns comandos que me ajudavam a montar e remontar a pasta quando precisasse. O conteúdo do arquivo é o seguinte:

    #!/bin/bash
    sudo umount /home/eber/mnt
    sudo mount -t vboxsf folder /home/eber/mnt

Eu desmonto a pasta antes de montar porque por algum motivo bizarro, os arquivos estáticos (js, css e etc) quebram quando eu atualizo seus conteúdos mas não dou um refresh na pasta montada. Assim fica fácil fazer o refresh sempre que for preciso.

Em seguida eu precisava tornar simples iniciar e finalizar a máquina virtual no Windows. Eu dei uma olhada no codigo do Vagrant e perguntei por aí e consegui fazer dois arquivinhos que me ajudam nisso. Um deles é o **start.bat**:

    "c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm Ubuntu --type headless

E o outro é o **stop.bat**:

    "c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm Ubuntu poweroff

A máquina é iniciada com o tipo headless, ou seja, roda "em background" (embora uma janela do CMD do Windows fique aberta, o que não é tão background assim).

**[EDIT 08/08/2011]**

O único problema de usar o método acima é que, na verdade, o processo da VM não é realmente _headless_ no Windows, já uma janela da linha de comando fica aberta. Eu procurei por alternativas e achei um [ticket](http://vbox.innotek.de/ticket/3549) no tracker do VBox falando exatamente sobre isso. Depois de ler tudo, existia a sugestão de usar um VBS (Visual Basic Script) pra abrir a VM. Aí vai:

    Set objShell = WScript.CreateObject("WScript.Shell")
    objShell.Run """C:\Program Files\Oracle\VirtualBox\VBoxHeadless.exe"" --startvm Ubuntu --vrdp=off", 0

Se o nome de sua VM for composto, tipo "Ubuntu Server" você só precisa colocar o nome entre duas aspas:

    Set objShell = WScript.CreateObject("WScript.Shell")
    objShell.Run """C:\Program Files\Oracle\VirtualBox\VBoxHeadless.exe"" --startvm ""Ubuntu Server"" --vrdp=off", 0

O que o script faz realmente eu não sei, porque não entendo nada de VBS, mas funciona :)

**[/EDIT]**

Pronto! Agora eu consegui ter um ambiente de trabalho muito similar ao que eu tenho em produção e poderia usar o servidor web na porta 80 sem problemas. Tudo o que eu fiz depois disso foi configurar um acesso via SSH usando o Putty e adicionar entradas no arquivo **c:\Windows\System32\Drivers\etc\hosts** apontando para o IP da máquina virtual. Como o IP é fixo, fica fácil manter a relação.

Parece que é muito mais trabalho que usar Vagrant e não têm os benefícios de redistribuição que o Vagrant tem, por isso dependendo da forma como você trabalha, usar Vagrant é muito melhor e mais prático. Mas foi divertido configurar este ambiente e tem funcionado perfeitamente pra mim desde então.
