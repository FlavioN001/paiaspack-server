# paiaspack-server - INDEV

## modpack server-side para hostear um servidor [paia's pack](https://github.com/FlavioN001/paiaspack).

### O ideal é usar isso no [debian](https://www.debian.org/), já que é o que eu usei e é um sistema estável para servidores, mas não é impossível fazer isso em qualquer outra distro linux, desde que você saiba o que está fazendo.

### Dependências:
[Git](https://git-scm.com/install/linux)
[java 21](https://docs.oracle.com/en/java/javase/21/install/installation-jdk-linux-platforms.html#JSJIG-GUID-ADC9C14A-5F51-4C32-802C-9639A947317F)
Editor de texto em terminal de sua preferência, ex: [nano](https://www.nano-editor.org/)

### Recomendações:

Um computador dedicado rodando uma distro linux com systemd.
Familiaridade com o terminal linux
Pelo menos 6gb RAM
Processamento decente (sandy bridge ou mais novo)

### Instruções
Tenha o [Git](https://git-scm.com/install/linux) e o [java 21](https://docs.oracle.com/en/java/javase/21/install/installation-jdk-linux-platforms.html#JSJIG-GUID-ADC9C14A-5F51-4C32-802C-9639A947317F) instalado.
    
debian:
```
sudo apt install --no-install-recommends git openjdk-21-jdk
```
        
Em seguida, clone o repositório e acesse seu diretório.
```
git clone https://github.com/FlavioN001/paiaspack-server && cd paiaspack-server
```
        
Depois, copie a pasta "Server" para o um diretório de sua escolha, ex: "~/Projetos"
```
cp -r Server/ ~/diretório/de/sua/escolha/
```
        
        
### Automação

agora, para automatizar a execução do servidor, será necessária a adaptação dos scripts, mas relaxa, não é nada muito complicado.
A meta é ligar o computador e o servidor iniciar sem nenhuma dor de cabeça ou execução de comandos.
Para isso, devemos iniciar uma sessão automaticamente em um tty alternativo. Isso é possível a partir da edição do serviço getty@tty4 (usarei o 4 pois é comum estar sem uso), desta forma:
```
sudo systemctl edit getty@tty4
```
A partir daí, você deve se deparar com um arquivo de texto parecido com isso:
![Exemplo do getty@tty4 antes da edição](/assets/tty4before.png)
        
### Repare que há uma área específica para a inserção de texto, logo após as duas primeiras linhas. Qualquer coisa escrita depois do texto comentado abaixo será ignorada pela edição.
Na área indicada, você deve adicionar o seguinte como as primeiras linhas (de edição):
 ```
[Service]
ExecStart=
```
        
Nesse mesmo arquivo, você deve encontrar a linha ExecStart comentada abaixo. Copie ela (ctrl+shift+c) e cole abaixo do ExecStart inserido a área indicada, ficará algo parecido com isso:
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --noreset --noclear --issue-file=/etc/issue:/etc/issue.d:/run/issue.d:/usr/lib/issue.d - ${TERM}
```
        
Enfim, edite o comando copiado para adicionar o parâmetro `--autologin seu-usuário`, substituindo "seu-usuário" pelo seu usuário (duh).
        
O resultado deve ser algo parecido com isso:
![Exemplo do getty@tty4 depois da edição](/assets/tty4after.png)
        
depois disso, salve o arquivo e saia (ctrl+o, enter, ctrl+x).
        
Ative o serviço agora editado:
```
sudo systemctl restart getty@tty4
sudo systemctl enable getty@tty4
```
Para verificar, tente apertar `ctrl+alt+f4` e veja se seu usuário está logado com sucesso.
Caso algo esteja errado, tente analisar novamente o passo a passo e verifique tudo.
            

            
### Agora que a parte mais chata já foi...
Finalmente, na pasta `paiaspack-server/Scripts (linux)/`:
edite o script (startserver):
Na linha `cd DIRETÓRIO/DO/SERVIDOR`, substitua o diretório pelo caminho da pasta Server/ que você guardou.
em seguida, transforme ele em um executável
```
chmod +x startserver
```
            
Finalmente, rode:
```
sudo cp startserver /usr/bin/
sudo cp mineserver.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable minecraft-server
```
            
Agora, ao reiniciar seu computador, o servidor já deve iniciar automaticamente, mas a conexão ainda depende de redirecionamento de portas no seu roteador (port-forwarding), e isso é bem inconveniente, já que provedores de internet nem sempre te dão o seu real endereço IPv4. Para contornar isso, podemos usar o endereço IPv6
```
[seu endereço IPv6]:28282
```    
Entretanto nem todos poderão conectar, pois até hoje alguns provedores de internet não provêm acesso a rede IPv6.
    
A solução definitiva para isso é usar um serviço de tunneling, eu usarei o [playit](playit.gg).
Instale o playit em seu computador debian:
```
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/playit.gpg >/dev/null
echo "deb [signed-by=/etc/apt/trusted.gpg.d/playit.gpg] https://playit-cloud.github.io/ppa/data ./" | sudo tee /etc/apt/sources.list.d/playit-cloud.list
sudo apt update
sudo apt install playit
```
        
Em seguida, copie o script de inicialização (na pasta `paiaspack-server/Scripts (linux)/`) do playit e ative ele pelo systemd:
```
sudo cp startplayit.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable startplayit.service
```
Então, entre no site do [playit](playit.gg) e faça a configuração do serviço.

## FINALMENTE.
Até que enfim, as configurações chegaram ao fim, agora é só mexer no servidor como quiser!
