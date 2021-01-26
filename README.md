Cluster HA DRBD(Dual-primary),Pacemaker/Corosync/Stonith-fence_xvm, DLM/CLVM e gfs2fs
=====================================================================================
Tutorial com base em algumas modificações de diversas pesquisas nos links:

## 1. Links e tutoriais usados
- http://www.voleg.info/Linux_RedHat6_cluster_drbd_GFS2.html
- https://www.tecmint.com/setup-drbd-storage-replication-on-centos-7/
- https://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/_initialize_drbd.html
- https://www.learnitguide.net/2016/07/how-to-install-and-configure-drbd-on-linux.html
- https://www.atlantic.net/cloud-hosting/how-to-drbd-replication-configuration/
- https://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/_initialize_drbd.html
- https://www.golinuxcloud.com/ste-by-step-configure-high-availability-cluster-centos-7/
- https://www.justinsilver.com/technology/linux/dual-primary-drbd-centos-6-gfs2-pacemaker/
- https://www.ibm.com/developerworks/community/blogs/mhhaque/entry/how_to_configure_red_hat_cluster_with_fencing_of_two_kvm_guests_running_on_two_ibm_powerkvm_hosts?lang=en
- https://www.osradar.com/installing-and-configuring-a-drbd-cluster-in-centos-7/
- http://www.tadeubernacchi.com.br/desabilitando-o-firewalld-centos-7/
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Changing_SELinux_Modes#sect-Security-Enhanced_Linux-Enabling_and_Disabling_SELinux-Permissive_Mode
- https://www.learnitguide.net/2016/07/how-to-install-and-configure-drbd-on-linux.html
- https://www.atlantic.net/cloud-hosting/how-to-drbd-replication-configuration/
- https://www.tecmint.com/setup-drbd-storage-replication-on-centos-7/
- https://major.io/2011/02/13/dual-primary-drbd-with-ocfs2/
- https://www.golinuxcloud.com/configure-gfs2-setup-cluster-linux-rhel-centos-7/
- https://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/ch09.html
- https://www.lisenet.com/2016/o2cb-cluster-with-dual-primary-drbd-and-ocfs2-on-oracle-linux-7/
- http://www.voleg.info/stretch-nfs-cluster-centos-drbd-gfs2.html
- http://jensd.be/186/linux/use-drbd-in-a-cluster-with-corosync-and-pacemaker-on-centos-7
- https://icicimov.github.io/blog/high-availability/Clustering-with-Pacemaker-DRBD-and-GFS2-on-Bare-Metal-servers-in-SoftLayer/
- https://www.justinsilver.com/technology/linux/dual-primary-drbd-centos-6-gfs2-pacemaker/
- http://www.tadeubernacchi.com.br/desabilitando-o-firewalld-centos-7/
- http://tutoriaisgnulinux.com/2013/06/08/_redhat-cluster-configurando-fence_virt/
- https://www.ntppool.org/zone/br
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Changing_SELinux_Modes
- https://www.golinuxcloud.com/ste-by-step-configure-high-availability-cluster-centos-7/,https://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/_install_the_cluster_software.html

Esquema similar ao apresentado no link:
- https://icicimov.github.io/blog/high-availability/Clustering-with-Pacemaker-DRBD-and-GFS2-on-Bare-Metal-servers-in-SoftLayer/

## 2. Configuração/discos/programas/ferramentas
2 mvs/vms de 50,3Gb com partições de 10,7Gb cada. Na mesma rede usando nada além de SHELL/BASH/SSH. SO e recursos usandos: CentOS 7, Pacemaker, Corosync, Stonith, Fence, DLM, CLVM, gfs2fs. 

## 3. Configuções de rede
Iniciando a configuração, o primeiro passo é definir os seus IP’s como estáticos, para criar uma conexão ssh estável entre elas.
Caso você não saiba usar o SSH veja este tutorial:(http://rberaldo.com.br/usando-o-ssh/).Ex:
```
hostname:vm1 - ip:10.255.255.xxx → ssh vm1@10.255.255.xxx
```
Lembrete, mantenha sempre o seu sistema atualizado!
Mudar o IP destas maquinas via arquivo para modo estático, ex:
```
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
Agora, com as placas de rede já configuradas, precisa-se redefinir o nome das maquinas virtuais no arquivo: /etc /hosts.
IPC: Exitem varias formas, por exemplo: hostnamectl set-hostname “vm0.cluster”. Em que “vm0.cluster” é o novo hostname completo da mv.

## 4. Etapas da configuração
- Etapa 1: Preparação da maquina quanto a rede e nome, que acabei de mostrar. Juntamente com a instalação do DRBD e a sua configuração em modo (DUAL-PRIMARY).E depois a instalação e configuração dos gerenciadores de cluster, o COROSYNC e o PACEMAKER. 
- Etapa 2: Instalação do CLUSTER PCS e a configuração deste cluster com o Fence e Stonith utilizando o agente (fence_xvm) que é voltado a mvs/vms, se você não esta usando mvs/vms, procure por seu agente especifico. DLM, CLVM(LVM). O primeiro trata os blocos assim como o nome sugere, porque, se um dos nós do cluster cair, é nosso dever manter o outo nó do cluster limpo.(LVM/CLVM) nada mais são que gerenciadores de volume lógico. Se vários nós do cluster exigirem acesso simultâneo de leitura / gravação a volumes LVM em um sistema ativo / ativo, você deverá usar o CLVMD.
O CLVMD fornece um sistema para coordenar a ativação e as alterações nos volumes de LVM nos nós de um cluster simultaneamente.
O serviço de bloqueio em cluster da CLVMD fornece proteção aos metadados do LVM, pois vários nós do cluster interagem com os volumes e fazem alterações em seu layout.
- Etapa 3: Criação dos ultimos recursos para a conclusão do projeto. Dentre eles temos o PV(Phisical Volume), o VG(Volume Group) e o LV(logical volume). No qual será montado o DRBD, e para isso mudaremos a configuração inical feita no arquivo de recurso("r0.res"). E formatação do "disco" simulado pelo DRBD com um sistema de arquivos GFS2FS e montagem de uma partição nele. Vale a pena mencionar que esta ultima etapa só deve ser feita na(o) mv/vm/server/nó primario (por que a mudança será replicada automaticamente).
IPC: A montagem e a configuração do DRBD foi feita no inicio por preferência minha, ela pode ser feita no final quando for necessaria sua implementação no código.

## 5. Instalação das dependencias DRBD
Adicionar o repositorio dos pacotes para o funcionamento do DRBD(Em todos servers/mvs/vms/nós!):
```
$ sudo rpm -ivh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
$ sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
$ sudo yum install drbd kmod-drbd
$ sudo yum install drbd90-utils.x86_64 drbd84-utils-sysvinit.x86_64 kmod-drbd84.x86_64”
```
Este ultimo comando deverá, depois de uma pergunta que deve ser respondida com (y), exibir uma resposta do tipo “Concluído”. 

## 6. Preparando os discos e partições(Replicar em todos servers/mvs/vms/nós!)
Agora o DRBD já está instalado, prepare o disco e as partiçoes para o arquivo de configuração do recurso (não precisa se preocupar com a identificação do modulo o mesmo será carregado no próximo boot e se os arquivos de configurção estiverem corretos, tudo funcionará).
```
$ sudo cfdisk /dev /nomedapartição 
```
Atente-se ao nome de seu disco/partição que pode ser diferente dependendo da distribuição linux que esta sendo usada("No meu caso: xvdb").
Depois de acionar esse comado, selecione a opção, NOVA (para criar uma nova partição), e depois a opção GRAVAR, para confirmar a criação dessa nova partição em disco. Mude a saida do comando: “lsblk” para este estado, pode se usar o comado: fdisk também.Ex:
```
[user@hostname ~]$ lsblk
NAME                           MAJ:MIN  RM    SIZE  RO TYPE  MOUNTPOINT
sr0                             11:0     1   1024M   0 rom   
xvda                              8:0    0     50G   0 disk  
├─xvda1                           8:1    0      1G   0 part  /boot
└─xvda2                           8:2    0     49G   0 part  
  ├─vg_centos65-lv_root (dm-0) 253:0     0   45,1G   0 lvm   /
  └─vg_centos65-lv_swap (dm-1) 253:1     0    3,9G   0 lvm   [SWAP]
xvdb                            132:16   0     10G   0 disk  
└─xvdb1                         132:17   0     10G   0 part
```

## 7. SELinux permissivo ou desativado(Replicar em todos servers/mvs/vms/nós!)
Ou mude o estado do SELinux para permissivo, com os comandos abaixo.
```
$ sudo setenforce permissive
$ sudo sestatus
SELinux status:                                      enabled
SELinuxfs mount:                          /sys/fs/selinux
SELinux root directory:                  /etc/selinux
Loaded policy name:                            targeted
Current mode:                                     permissive
Mode from config file:                        enforcing
Policy MLS status:                                enabled
Policy deny_unknown status:              allowed
Max kernel policy version:                      31
```
Ou desative o SElinux permanentemente com a sequencia de comandos.
```
Disabling SELinux permanently
Edit the /etc/selinux/config file, run:
$ sudo vi /etc/selinux/config
Set SELINUX to disabled:
SELINUX=disabled
Save and close the file in vi/vim. Reboot the Linux system:
$ sudo reboot
```
Para saber mais acesse o link: https://www.cyberciti.biz/faq/disable-selinux-on-centos-7-rhel-7-fedora-linux/

## 8. Revisando toda configuração até aqui.Firewall(Replicar em todos servers/mvs/vms/nós!)
Configuração de Firewall
Consulte a documentação do seu firewall para saber como abrir / permitir portas. Você precisará das seguintes portas abertas para seu cluster funcionar corretamente. 
Portas:
| # | Component   | Protocol    | Port              |
|---|-------------|-------------|-------------------|
| 1 | DRBD        |     TCP     | 7788              |
| 2 | Corosync    |     UDP     | 5404, 5405        |
| 3 | GFS2        |     TCP     | 2224, 3121, 21064 |
```
$ sudo iptables -I INPUT -p tcp --dport 2224 -j ACCEPT   ---   iptables -nL | grep 2224
$ sudo iptables -I INPUT -p tcp --dport 3121 -j ACCEPT   ---   iptables -nL | grep 3121
$ sudo iptables -I INPUT -p tcp --dport 21064 -j ACCEPT ---   iptables -nL | grep 21064
$ sudo iptables -I INPUT -p udp --dport 5404 -j ACCEPT  ---   iptables -nL | grep 5404
$ sudo iptables -I INPUT -p udp --dport 5405 -j ACCEPT  ---   iptables -nL | grep 5405
```
Tambem habilite a porta 7788 no firewall, de ambas as maquinas para não sofrer futuros erros de validação, faça os comandos abaixos em todos os nós do projeto.
```
$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.255.255.231" port port="7788" protocol="tcp" accept'
$ sudo firewall-cmd reload
```

## 9. Instale o DRBD:(Replicar em todos servers/mvs/vms/nós!)
```
$ sudo rpm -ivh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
$ lsmod | grep -i drbd
```
Verifique se todos os hosts estão com os nomes e ips devidamente configurados.
```
$ cat /etc/hostname
$ cat /etc/hosts
```

## 10. Arquivos de configuração(Replicar em todos servers/mvs/vms/nós!)
Edite o arquivo “/etc/drbd.d/global_common.conf” e modifique a opção “usage-count de yes para no” e salve o arquivo, em todos os nós(mvs) do DRBD.
E em todos os nós/vms/etc do cluster crie o arquivo, “r0.res” dentro do diretório, “/etc/drbd.d/”.
Mova o arquivo de loop que deve ficar oo diretório: “/etc/init.d/loop-for-drbd” Para manter o modo dual-primary(no meu caso) após o reboot.

## 11. Execução do script(De acordo com seu ambiente)
Com esses arquivos em seus respectivos lugares, inicie a configuração do DRBD para uso em modo (DUAL-PRIMARY).
Não ative o DRBD nesta etapa, para que nada dê errado no momento da ativação do cluster.
As instruções devem ser implementadas em todas as mvs/nós e etc. Embora, neste guia só seja mostrado rodando em um conjunto de 2 maquinas virtualizadas com vmware em uma mesma rede.
*Dentro do script estão comentados os comandos das mvs/vms/nós secundarios, leia para fazer modificações de acordo com seu cenário.*

## 12. Isso é tudo.. até agora
![That's all folks](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRjTP8kaxaOmV1_V4FYGLwJ27se8-5WUl-IyQ&usqp=CAU)
