# Roteiro: Acessando seu Site IIS Express pelo Smartphone

**Versão 1.0**: 25 de Julho de 2025, Sexta-feira.

> [!WARNING] Atenção
> 
> O roteiro passará por revisões e atualizações ao longo do tempo, no entanto é importante ficar atento que podem ocorrer variações entre uma revisão e outra, seja por atualização de alguma ferramenta e/ou processo que independe dos passos descritos. Mantenha-se ativamente atualizado.
> 
> Qualquer dúvida: raphael.lorenzo@zencheck.com.br

Este roteiro detalha o processo para configurar seu ambiente de desenvolvimento Windows e Visual Studio para permitir o acesso a uma aplicação web rodando no IIS Express a partir de um smartphone na mesma rede local.

## Pré-requisitos

- Computador e smartphone na **mesma rede Wi-Fi**.

- Visual Studio executado como **Administrador**.

> **Spoiler:** Alternativa Rápida no final do documento. Verifique sua necessidade.

---

## Parte 01: A Configuração Essencial

Estes são os primeiros passos para a configuração.

### 1. Obter o Endereço IP do Computador

O smartphone precisa do endereço do seu computador na rede.

- **Ação:** Abra o Prompt de Comando (`cmd`) como **Administrador**  e execute o comando:

```
 ipconfig
```

_Abrir Prompt de Comando:_ Pressione `Windows + x` e selecione "Prompt de Comando (Admin)" ou "Terminal (Admin)"

- **Dado a Obter:** Anote o `Endereço IPv4` do seu adaptador de rede ativo (geralmente Wi-Fi ou Ethernet). Ex: `192.168.2.173`.
- Deixe o Prompt de Comando aberto.

### 2. Configurar os Bindings do IIS Express

Por padrão, o IIS Express só aceita conexões de `localhost`. É preciso adicionar um _binding_ para o seu IP.

- **Ação:** Abra o arquivo `applicationhost.config` localizado na pasta oculta da sua solução:

```
/{PASTA_DA_SUA_SOLUCAO}/.vs/config/applicationhost.config
```

- **Modificação:** Encontre a seção `<bindings>` do seu site e adicione uma nova linha com seu IP, mantendo a mesma porta (a porta e o ip abaixo são exemplos).

```
<bindings>
  <binding protocol="http" bindingInformation="*:57570:localhost" />

  <binding protocol="http" bindingInformation="*:57570:192.168.2.173" />
</bindings>
```

### 3. Criar a Regra no Firewall do Windows

O firewall bloqueará a conexão por padrão. Crie uma regra de entrada para permitir o acesso à porta da sua aplicação.

- **Ação:** Abra o Prompt de Comando **como Administrador** e execute o comando abaixo, substituindo os valores `SUA_PORTA` e `SEU_PERFIL_DE_REDE` (`private` ou `public`).

- **De onde vem a `SUA_PORTA`?**

É a porta que sua aplicação usa, definida pelo Visual Studio. Você pode encontrá-la no passo anterior, no arquivo `applicationhost.config`. No exemplo acima, a porta é `57570`. Você também pode verificá-la nas propriedades do seu projeto web no Visual Studio (geralmente na aba "Debug" ou "Web").

- **O que é o `SEU_PERFIL_DE_REDE` e como verificar qual é o seu?**

O Windows classifica sua conexão de rede como "Privada" ou "Pública" por segurança.
**Privada:** A comunicação entre dispositivos é mais permissiva (para dispositivos na mesma rede, o que é exatamente o ponto de atenção).
**Pública:** A comunicação é restrita para proteger seu computador (por isso nas propriedades aparece como opção recomendada).
**Para Verificar:** Vá em `Configurações > Rede e Internet > Wi-Fi (ou Ethernet)`.

_Recomendo prosseguir com o perfil "Público" e caso ocorra alguma questão continue seguindo os passos do roteiro._

Comando:

```
netsh advfirewall firewall add rule name="IISExpressWebApp" dir=in protocol=TCP localport=SUA_PORTA profile=SEU_PERFIL_DE_REDE action=allow
```

- **Exemplo:**

```
netsh advfirewall firewall add rule name="IISExpressWebApp" dir=in protocol=TCP localport=57570 profile=private action=allow
```

> [!SUCCESS] Concluído!
> 
> Após esses 3 passos, inicie sua aplicação no Visual Studio e acesse http://SEU_IP:SUA_PORTA no navegador do smartphone. Se tudo correu bem, o site irá carregar.

---

## Parte 02: Diagnósticos e Solução de Problemas

Se a configuração essencial não funcionou, siga estes passos para encontrar a causa.

#### Problema: A conexão não completa ou apresenta "Timeout"

Isto quase sempre indica que algo está bloqueando a comunicação na rede.

#### Diagnóstico 1: O Firewall é o Culpado?

O teste mais rápido para isolar a causa.

- **Como Verificar:** Desative temporariamente o Firewall do Windows para o seu perfil de rede (Privado ou Público) e tente conectar pelo celular.

- **Resolução:**
  
  - Se **funcionou**, o problema é 100% o firewall.
  
  - **Reative o firewall imediatamente** e siga para o próximo diagnóstico para criar a regra correta em vez de deixar seu sistema desprotegido.

#### Diagnóstico 2: O Perfil da Rede está "incorreto"

O Windows aplica regras diferentes para cada perfil. O principal ponto é o perfil de rede estar diferente da regra configurada para o firewall, seja pela linha de comando ou pela interface gráfica. Pode ser feita tentativa de deixar ambos no perfil privado (firewall e rede), porém é necessário cautela, de modo geral é recomendado utilizar o perfil público.

- **Como Verificar:** Vá em `Configurações > Rede e Internet > Ethernet (ou Wi-Fi) > Propriedades`. Verifique o **"Tipo de perfil de rede"**.

- **Resolução:** (Teste provisório) Se estiver como "Pública", mude para "Privada". Após a mudança, certifique-se de que a regra do firewall criada no Passo 3 da configuração essencial está em conformidade.

#### Diagnóstico 3: A Regra do Firewall foi criada manualmente e está errada

Se você tentou criar a regra pela interface gráfica e não funcionou, provavelmente foi por um destes motivos.

- **Como Verificar:** Abra o "Firewall do Windows Defender com Segurança Avançada" e encontre sua regra de entrada.

- **Erros Comuns:**
  
  - `Protocolo:` definido como `Qualquer` em vez de `TCP`.
  
  - `Porta Local:` definida como `Qualquer` em vez do número da porta específico (ex: `57572`).

- **Resolução:** Exclua a regra incorreta e crie uma nova, especificando o **Protocolo (TCP)** e a **Porta Local** corretamente, ou simplesmente use o comando `netsh` do Passo 3, que é mais rápido e menos propenso a erros.

#### Diagnóstico 4: O Servidor não está "Ouvindo" (Listening)

Útil para confirmar que o problema não é o IIS Express ou o `applicationhost.config`.

- **Como Verificar:** Com a aplicação rodando no Visual Studio, execute no `cmd`:

```
  netstat -an | find "SUA_PORTA"
```

- **Análise do Resultado:**
  
  - **Saída Correta:** Você verá múltiplas linhas para a sua porta com o status `LISTENING`. Isso significa que o servidor está funcionando perfeitamente.
  
  - **Saída Incorreta:** Se nada aparecer ou não houver `LISTENING`, o erro está na configuração dos bindings no seu arquivo `applicationhost.config`.

#### Diagnóstico 5: Outro Software de Segurança (Antivírus)

Se tudo acima foi verificado e ainda falha, o culpado é quase sempre um segundo firewall.

- **Observação Técnica:** Suítes de segurança (Avast, Norton, Kaspersky, etc.) possuem seus próprios firewalls que rodam em paralelo e podem sobrescrever as regras do Windows.

- **Resolução:** Desative temporariamente o módulo de firewall do seu antivírus. Se funcionar, você precisará encontrar a seção de "Regras de Aplicativo" ou "Exceções de Firewall" dentro do seu antivírus e adicionar uma permissão para o `iisexpress.exe` ou para a porta TCP que você está usando.

---

### Alternativa Rápida: `ngrok`

Se a configuração de rede local for muito complexa ou restritiva, `ngrok` é uma ferramenta que resolve o problema em um minuto.

> [!TIP] O que o ngrok faz?
> 
> Ele cria um túnel público seguro na internet que aponta diretamente para a sua aplicação local (localhost:porta), excluindo completamente a necessidade de configurar IPs, bindings ou firewalls.

- **Como Usar:**
  
  1. Baixe o `ngrok`.
  2. Execute sua aplicação no VS.
  3. Execute no `cmd`: `ngrok http SUA_PORTA`
  4. Acesse a URL `https://aleatorio.ngrok.io` que ele fornecer no seu smartphone.
