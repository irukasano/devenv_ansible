# ansible で環境セットアップする
build up development environment by ansible

for centos7


## 開発環境(Vagrant)

clone this repository to vagrantfile dir

Vagrantfile

    config.vm.provision "file", source: "./ansible/", destination: "/home/vagrant/"

    config.vm.provision "shell", inline: <<-SHELL
      yum -y install ansible --exclude=kernel
      ansible-playbook /home/vagrant/ansible/main.yml
    SHELL

Edit vars/main.yml

## 本番環境の手順(さくらインターネット VPS)

### インストール後にサーバーへログイン

wsl$ ssh root@{serveripv4}

### 管理者ユーザーを追加

[root@{servername} ~]# useradd {username}
[root@{servername} ~]# passwd {username}
[root@{servername} ~]# usermod -G wheel {username}
[root@{servername} ~]# vi /etc/pam.d/su

    #auth       required     pam_wheel.so use_uid
    ↓
    auth       required     pam_wheel.so use_uid　←　コメント解除


### 鍵認証ログインするので、クライアントで鍵作成し転送する

wsl$ cd ~/.ssh/idfiles
wsl$ ssk-keygen -t rsa -b 4096 -f {servername}.id_rsa

wsl$ ssh-copy-id -i ~/.ssh/idfiles/{servername}.id_rsa.pub {username}@{serveripv4}

< 接続確認 >

wsl$ ssh -i ~/.ssh/idfiles/{servername}.id_rsa {username}@{serveripv4}

### 鍵認証できたので、パスワード認証を無効にする

[{username}@{servername} ~]$ sudo vi /etc/ssh/sshd_config

    #Protocol 2,1
    ↓
    Protocol 2　←　SSH2のみで接続を許可

    #Port 22
    ↓
    Port 29992 ←  SSH ポートを変更(29992 は一例です、22や予約されているポート(80,443,23など)以外の他の番号を使ってください)

    #PermitRootLogin yes
    ↓
    PermitRootLogin no　←　rootでのログインを禁止

    #PasswordAuthentication yes
    ↓
    PasswordAuthentication no　←　パスワードでのログインを禁止(鍵方式によるログインのみ許可)

    #PermitEmptyPasswords no
    ↓
    PermitEmptyPasswords no　←　パスワードなしでのログインを禁止


[{username}@{servername} ~]$ sudo systemctl restart sshd

< sshd の状態が以下のように active 表示なら正常 >

[{username}@{servername} ~]$ sudo systemctl status sshd

    ● sshd.service - OpenSSH server daemon
       Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
       Active: active (running) since 月 2021-02-08 10:16:54 JST; 4s ago
         Docs: man:sshd(8)
               man:sshd_config(5)
     Main PID: 1433 (sshd)
       CGroup: /system.slice/sshd.service
               ├─1421 sshd: [accepted]
               └─1433 /usr/sbin/sshd -D

< パスワードログインできないことを確認 >

[{username}@{servername} ~]$ exit
wsl$ ssh {username}@{serveripv4} -p 29992
{username}@{serveripv4}: Permission denied (publickey).  ← こう表示されればパスワードログイン不可

### 各種設定

[{username}@{servername} ~]$ su -
[root@{servername} ~]$ wget https://github.com/irukasano/devenv_ansible.git
[root@{servername} ~]$ yum -y install ansible --exclude=kernel
[root@{servername} ~]$ cd /root/devenv_ansible
[root@{servername} ~]$ cp -p ./vars/main_prod.yml.sample ./vars/main.yml
[root@{servername} ~]$ ansible-playbook main_prod.yml




