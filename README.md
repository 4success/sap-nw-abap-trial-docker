# SAP NW ABAP 7.52 SP04 Trial in Docker

SAP NW ABAP 7.52 SP04 rodando no Docker para fins de treinamento. Isso irá facilitar o processo de instalação e, quando não for mais necessário, desinstalação. Também é possível rodar com uma VM no VirtualBox ou similares.


**Dicas:**

Para instrução adicionais sobre a instalação do NW ABAP 7.52 SP04, confira o [anúncio ofical da SAP feito por Julie Plummer](https://blogs.sap.com/2019/07/01/as-abap-752-sp04-developer-edition-to-download/).

Confira o passo-a-passo em vídeo que gravamos em nosso [canal do Youtube](https://youtu.be/3zPbJcHZns4).
## Inspirações

Esse Dockerfile foi baseado:

- [Nesse repositório do Nabi Zamani](https://github.com/nzamani/sap-nw-abap-trial-docker)

## Instruções

1. Instale o [Docker CE](https://www.docker.com/community-edition)

1. Aumente o parâmetro `Disc Image Size` nas suas preferências do Docker no menu "Resources > Advanced"
    - Adicione pelo menos 100 GB ao valor já cadastrado
    - Aumente o parâmetro `Memory` para no mínimo `6 GiB`

    **IMPORTANTE:** se você pular esse passo, poderá ter erros durante o processo de instalação.

1. Mudar o valor do parâmetro **vm.max_map_count** para evitar erros de instalação

    Para o **NW ABAP 7.52** a instalação tentará modificar o parâmetro **vm.max_map_count** se verificar que o valor está abaixo do esperado. Porém, se isso acontecer ocorrerá um erro **sysctl: setting key "vm.max_map_count": Read-only file system**. Sendo assim, é importante mudar o valor do parâmetro **vm.max_map_count** antes de começar a instalação para um valor maior ou igual ao necessário, para a instalação não ficar tentando indefinidamente modificar esse valor sem sucesso. Nos testes realizados o valor 1000000 funcionou sem problemas. Abaixo o procedimento para modificar esse parâmetro:

    - Linux:

        ```sh
        sysctl -w vm.max_map_count=1000000
        ```

    - macOS, com o Docker for Mac

        ```sh
        screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
        sysctl -w vm.max_map_count=1000000
        ```
    - Windows, com o Docker Desktop rodando em Hyper-V
   
       ```sh
       # Inicia um container com acesso privilegiado para o Docker daemon
       docker run --privileged -it --rm -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker alpine sh
       exit
       
       # Rode um container com acesso total ap MobyLinuxVM
       docker run --net=host --ipc=host --uts=host --pid=host -it --security-opt=seccomp=unconfined --privileged --rm -v /:/host alpine /bin/sh
       
       # Trocando para o FS do host
       chroot /host
       
       # Ajustando o parâmetro
       sysctl -w vm.max_map_count=1000000
    
    - Windows, com o Docker Desktop baseado em WSL

        ```sh
        wsl -d docker-desktop
        sysctl -w vm.max_map_count=1000000
        ```

    - Windows e macOS com Docker Toolbox (legados)

        ```sh
        docker-machine ssh
        sudo sysctl -w vm.max_map_count=1000000
        ```

    Depois de executado, você pode verificar se deu certo com o comando `sysctl vm.max_map_count`. Para sair, aperte `crtl+a` e `ctrl+d`.

    Referências adicionais [aqui](https://blogs.sap.com/2019/12/11/running-sap-nw-7.52-sp4-trial-in-docker-in-windows-subsystem-for-linux-2/), [aqui](https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html#docker-cli-run-prod-mode), [aqui](https://deployeveryday.com/2016/09/23/quick-tip-docker-xhyve.html), e na [SAP Note 900929](https://launchpad.support.sap.com/#/notes/900929) que recomenda a utilização do valor máximo de  2147483647 por uma questão de **'simplicidade'**
    
    Se mesmo assim ainda estiver encontrando algun erro, adicione o parâmetro `--privileged` no seu comando `docker run`  

1. Instalar o [Git](https://git-scm.com)

    No Windows, eu sugiro também instalar o Git Bash (será uma opção durante o processo de instalação).

    **Obs:** Não é necessário instalar o Git. Você pode apenas baixar esse repositório na sua máquina via página do GitHub.

1. Clone esse repositório

    ```sh
    git clone https://github.com/4success/sap-nw-abap-trial-docker.git
    cd sap-nw-abap-trial-docker
    ```

1. Faça o download do [SAP NW ABAP 7.52 SP04 Trial do site da SAP SAP](https://developers.sap.com/trials-downloads.html?search=7.52%20SP04) (search for **7.52 SP04**), then:
    - Crie um diretório `sapdownloads` dentro do repositório clonado
        - `mkdir sapdownloads`
    - Extraia o conteúdo dos arquivos rar baixados (extraia apenas o primeiro arquivo rar que ele fará o processo automaticamente). É necessário que o WinRAR esteja instalado ou outro software compatível
        - `unrar x TD752SP01.part01.rar ./sapdownloads`

    **Dica:** A SAP quer saber que faz o download do NW ABAP Trial. Sendo assim, você precisa se cadastrar e logar antes de fazer os downloads. A criação da conta é grátis, assim como o download. A conta pode ser a mesma utilizada no SAP Communitiy / SCN.

1. Faça o build da imagem

    - Sem proxy de rede

        ```sh
        docker build -t nwabap:7.52 .
        ```

    - Com proxy de rede

        ```sh
        docker build --build-arg http_proxy=http://proxy.dominio.corp:1234 --build-arg https_proxy=http://proxy.dominio.corp:1234 -t nwabap:7.52 .
        ```

        **Dica:** Se sua rede tiver algum tipo de proxy, o comando `docker build` irá falhar caso não seja informado os dados corretos de proxy conforme exemplo acima. Também leve em consideração que você pode precisar alterar alguma configuração de rede dentro do container. O recomendado é usar essa imagem fora de uma rede com proxy.

1. Crie / inicie com um dos comandos abaixo:

    - Use esse para usar o mapeamento de portas com as mesmas numerações no localhost (recomendado)

        ```sh
        docker run -p 8000:8000 -p 44300:44300 -p 3300:3300 -p 3200:3200 -p 50000:50000 -p 50001:50001 -p 8101:8101 -h vhcalnplci --name nwabap752 -it nwabap:7.52 /bin/bash
        ```

    - Use esse para gerar portas "aleaórias" no localhost, se quiser assim

        ```sh
        docker run -P -h vhcalnplci --name nwabap752 -it nwabap:7.52 /bin/bash
        ```

    **Dica 1:** Se você enfrentar algum erro relacionado a item 3 (vm.max_map_count), execute o comando com `--privileged`      
    ```sh
    docker run --privileged -p 8000:8000 -p 44300:44300 -p 3300:3300 -p 3200:3200 -p 50000:50000 -p 50001:50001 -p 8101:8101 -h vhcalnplci --name nwabap752 -it nwabap:7.52 /bin/bash
    ```

    **Dica 2:** Você também pode usar `--rm` para o container ser excluído logo após você sair do terminal ou ele for parado 

      ```sh
      docker run -p 8000:8000 -p 44300:44300 -p 3300:3300 -p 3200:3200 -p 50000:50000 -p 50001:50001 -p 8101:8101 -h vhcalnplci --rm --name nwabap752 -it nwabap:7.52 /bin/bash
      ```
1. Para começar a instalação do SAP NW ABAP 7.52 Trial:

    ```sh
    /usr/sbin/uuidd
    ./install.sh
    ```

    Sua instalação foi finalizada com sucesso se você visualizar a seguinte mensagem: **Installation of NPL successful**

    **Dica:** Essa instalação leva cerca de 20-30 minutos. Quando finalizada, seu SAP estará rodando. Em seguida, pare o sistema e saia do container.

## Iniciando e parando o NW ABAP 7.52 Trial

1. Para iniciar o SAP NW ABAP Trial: (depois de instalado, use esse comando ao invés de `docker run ...`)

    ```sh
    docker start -i nwabap752
    /usr/sbin/uuidd
    su npladm
    startsap ALL
    ```

1. Para parar o SAP NW ABAP Trial e o container (`ALL` pode ser omitido)

    ```sh
    su npladm
    stopsap ALL
    exit
    exit
    ```

    **Dica:** Depois do segundo `exit` o container Docker já está parado.

## Passos importantes depois da instalação

1. Atualizando a licença

    - Abra o SAP GUI and entre na instância
        - **Usuário:** SAP*
        - **Senha:** Down1oad
        - **Client:** 000

    - Abra a transação `SLICENSE`
    - Copie o valor do campo `Active Hardware Key` exibido na tela
    - Navega até a página [SAP License Keys for Preview, Evaluation, and Developer Versions](https://go.support.sap.com/minisap/#/minisap) no seu navegador
    - Escolha `NPL - SAP NetWeaver 7.x (Sybase ASE)`
    - Preencha os campos. Use o `Hardware Key` que você copiou da `SLICENSE`
    - Baixe o arquivo `NPL.txt` e volte a transação `SLICENSE`
    - Apague a `Installed License` da tabela
    - Aperte o botão `Install` na parte inferior da tabela
    - Escolha o arquivo `NPL.txt` baixado anteriormente
    - Ok, finalizado! Agora você pode logar com o usuário normal.

    Você pode entrar no `client 001` com qualquer um dos usuários abaixo (todos compartilham a mesma senha `Down1oad`, geralmente você irá usar o `DEVELOPER`):

      - **User:** DEVELOPER (Developer User)
      - **User:** BWDEVELOPER (Developer User)
      - **User:** DDIC (Data Dictionary User)
      - **User:** SAP* (Administrador SAP)

1. Gerando dados de teste

    Para gerar dados de exemplo, você deve executar uma das opções abaixo:

      - **Report:** SAPBC_DATA_GENERATOR
      - **Transaction Code:** SEPM_DG

1. Sugestão: Ativar o serviço ping

    - Vá até a transação `SICF`
    - Ative o nó `/sap/public/ping` (default_host)
    - Teste as conexões HTTP e HTTPS diretamente do seu navegador

        - **HTTP:**  [http://localhost:8000/sap/public/ping](http://localhost:8000/sap/public/ping)
        - **HTTPS:** [https://localhost:44300/sap/public/ping](https://localhost:44300/sap/public/ping)

## Arquivos de log

Assumindo que você iniciou o seu container anteriormente e está com o usuário `npladm`:

  ```sh
  docker start -i nwabap752
  /usr/sbin/uuidd
  su npladm
  ```

Depois, escreva `alias` para você ver alguns atalhos que a SAP deixou criado:

  ```sh
  alias
  ```

Um deles é o `cdDi`, que você pode executar no seu terminal:

  ```sh
  cdDi
  ```

Então, depois de executar o comando `ls -ahl`, você sabe que existe um diretório `work`:

  ```sh
  cd work
  ```

Esse diretório tem arquivos de log importantes, como `dev_icm` e `dev_w0`:

  ```sh
  vi dev_icm
  vi dev_w0
  ```

Verifique o conteúdo (com vi ou tail, por exemplo), caso esteja enfrentando problemas com o seu NW ABAP, que você não pode explicar. Por exemplo, caso seu NW ABAP tenha sido iniciado com sucesso, mas você não consegue acessar o serviço de ping via HTTP / HTTPS, poderá encontrar a resposta em um desses arquivos.
