# paiaspack-server

## modpack server-side para hostear um servidor [Paia's Pack](https://github.com/FlavioN001/paiaspack) gratuitamente e até com CGNAT.

### O ideal é usar isso no [debian](https://www.debian.org/), pois é o que eu usei e é um sistema estável para servidores, mas não é impossível fazer isso em qualquer outra distro linux, desde que você saiba o que está fazendo.

### Dependências:
[Git](https://git-scm.com/install/linux)\
[java 21](https://www.oracle.com/java/technologies/downloads/#java21)\
[Screen](https://wiki.debian.org/screen)\
Editor de texto em terminal de sua preferência, (ex: [nano](https://www.nano-editor.org/))

### Recomendações:

Um computador dedicado rodando uma distro linux com systemd.\
Familiaridade com tty e o terminal linux\
Pelo menos 6gb RAM\
Processamento decente (sandy bridge ou mais novo)

### Instruções
Tenha o [Git](https://git-scm.com/install/linux), o [Java 21](https://docs.oracle.com/en/java/javase/21/install/installation-jdk-linux-platforms.html#JSJIG-GUID-ADC9C14A-5F51-4C32-802C-9639A947317F) e o [Screen](https://wiki.debian.org/screen) instalado.
    
debian:
```
sudo apt install --no-install-recommends git openjdk-21-jdk screen
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

Agora, para automatizar a execução do servidor, será necessária a adaptação dos scripts, nada muito complicado.\
A meta é ligar o computador e o servidor iniciar sozinho de fundo.\
Para isso, deve-se iniciar uma sessão automaticamente em um outro tty. Isso é possível a partir da edição do serviço getty@tty4 (tty4 por ser raramente usado), desta forma:
```
sudo systemctl edit getty@tty4
```
A partir daí, você deve se deparar com um arquivo de texto parecido com isso:
![Exemplo do getty@tty4 antes da edição](/assets/tty4before.png)
        
### Repare que há uma área específica para a inserção de texto, logo após as duas primeiras linhas. Qualquer coisa escrita depois do texto comentado abaixo será apagado pela edição.
Na área indicada, você deve adicionar as seguintes linhas
 ```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin seuusuario --noclear %I 38400 linux
```
*Lembre de alterar o "seuusuario" para inserir o seu usuário do sistema.*

O resultado deve ser algo parecido com isso:
![Exemplo do getty@tty4 depois da edição](/assets/tty4after.png)
        
Depois disso, salve o arquivo e saia (ctrl+o, enter, ctrl+x).
        
Ative o serviço agora editado:
```
sudo systemctl daemon-reload
sudo systemctl restart getty@tty4
sudo systemctl enable getty@tty4
```
Para verificar, tente apertar `ctrl+alt+f4` e vefrifique se seu usuário está logado. \
Caso não, analise o passo a passo e verifique o que foi feito errado.

## Com isso fora do caminho:
- Abra a pasta clonada `paiaspack-server/Scripts\ (linux)/`:

- Edite o script de inicialização `startserver`:\
Na linha `cd DIRETÓRIO/DO/SERVIDOR`, substitua o diretório pelo caminho da pasta `Server/` que você guardou.\
Em seguida, transforme ele em um executável:
```
chmod +x startserver
```

- Edite o serviço de inicialização (mineserver.service):/
Na linha `User=`, troque `seuusuário` pelo seu usuário logado no tty4

            
Finalmente, rode:
```
sudo cp startserver /usr/bin/
sudo cp mineserver.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable mineserver
```
E adicione isso ao fim do seu arquivo `~/.bashrc`
```
#SCREEN NO TTY4
if [ "$(tty)" = "/dev/tty4" ]; then
        screen -r mine
fi
```
Isso fará o terminal do servidor abra ao entrar no tty4.\
            
Agora, ao reiniciar seu computador, o servidor já deve iniciar automaticamente, mas a conexão ainda depende de redirecionamento de portas no seu roteador (port-forwarding) das portas 28282 e 24454, e isso é bem inconveniente, já que provedores de internet nem sempre te dão o seu real endereço IPv4. Para contornar isso, podemos usar o endereço IPv6
```
[seu endereço IPv6]:28282
```    
Entretanto, nem todos os jogadores poderão se conectar ao servidor, pois até hoje alguns provedores de internet não disponibilizam acesso à rede IPv6.
    
A solução definitiva para isso é usar um serviço de tunneling, recomendo o [Play It](playit.gg). pela sua facilidade e bom funcionamento.\
[Instale o playit em seu computador debian:](https://playit.gg/support/run-on-linux/)
```
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/playit.gpg >/dev/null
echo "deb [signed-by=/etc/apt/trusted.gpg.d/playit.gpg] https://playit-cloud.github.io/ppa/data ./" | sudo tee /etc/apt/sources.list.d/playit-cloud.list
sudo apt update
sudo apt install playit
```
        
Em seguida, ative o serviço de inicialização do playit e faça o setup inicial
```
sudo systemctl start playit
sudo systemctl enable playit
playit setup
```
Então, entre na url que aparecerá no seu terminal e faça a configuração do serviço [playit](playit.gg), adicionando um tunnel para a porta `28282`(para o servidor) e outro para `24454`(para o voicechat)

*IMPORTANTE* \
Para o voicechat funcionar, também é necessário adicionar o um ip de tunnel do playit que com a porta 24454 UDP ao arquivo de configuração `Server/config/voicechat/voicechat-server.properties` na linha `voice_host=`

# FINALMENTE.
## Faça as alterações que quiser para o seu caso de uso e use até como base para seu próprio modpack
