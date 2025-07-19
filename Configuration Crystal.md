# Roteiro de Instalação e Configuração do Crystal Reports


**Versão 1.0**: 04 de Julho de 2025, Sexta-feira.

### Fase 0: Introdução

Você precisará de **dois instaladores diferentes**:

1. **Crystal Reports for Visual Studio (CR4VS):** Esta é a ferramenta de _desenvolvimento_. Ela se integra ao seu Visual Studio 2022 e permite que você abra, edite e compile os arquivos de relatório (`.rpt`).

2. **Crystal Reports Runtime:** Este é o "motor" de execução. Ele precisa ser instalado na máquina onde a aplicação vai rodar (no seu caso, sua própria máquina, para o IIS Local) para que o site consiga processar e exibir os relatórios para o usuário.

---
### Fase 1: Instalação das Ferramentas (Obrigatório)

1. **Feche o Visual Studio:** Antes de instalar, feche todas as instâncias do Visual Studio 2022. Não abra-o novamente até ser solicitado.

2. **Instalar o Crystal Reports for Visual Studio:**

    - **O quê:** Baixe o **"SAP Crystal Reports for Visual Studio (SP38)"** ou a versão mais recente disponível. SP significa "Service Pack".
    
    - **Onde:** No site https://origin.softwaredownloads.sap.com/public/site/index.html ou na pasta enviada (`Crystal Configuration`) utilize o arquivo nomeado como: `CRforVS6413SP38_0-80007712`.
    
    - **Qual:** Instalador para o VS 2022 `CR for Visual Studio SP38 64b installer (VS 2022 and above).exe`
    
    - **Como:** Baixe e execute o instalador. É uma instalação padrão (next, next, finish). Após a conclusão, ao abrir seu projeto no VS 2022, você conseguirá visualizar os arquivos `.rpt` (Não precisa abrir/testar por agora).
	
	- No final irá pedir para instalar o Runtime 32 bits, pode deixar selecionado e prosseguir com a instalação. Isso porque:
		- **Ambiente de Desenvolvimento (Dentro do Visual Studio):** O designer do Crystal Reports, que roda dentro do Visual Studio para você poder editar os relatórios, muitas vezes depende de componentes 32-bit por questões de legado. Por isso, o instalador principal sugere ou instala o runtime de 32-bit. Ele garante que o desenvolvedor consiga visualizar e trabalhar nos relatórios dentro da sua ferramenta.

3. **Instalar o Crystal Reports Runtime:**

    - **O quê:** Na mesma página de downloads, baixe o **"Crystal Reports Runtime"**. Existem versões de 32 bits (x86) e 64 bits (x64).
	
	- **Onde:** No site https://origin.softwaredownloads.sap.com/public/site/index.html ou na pasta enviada (`Crystal Configuration`) utilize o arquivo nomeado como: `CR13SP38MSI64_0-80007712`.
    
    - **Qual:** Runtime 64-bit `CR for Visual Studio SP38 CR Runtime 64-bit MSI`
    
    - **Como:** Execute o instalador. Ele instalará as DLLs necessárias para que o IIS consiga processar os relatórios.
	
	- **Por quê:** **Ambiente de Execução (No seu site rodando no IIS).** A sua aplicação web, quando roda no IIS (mesmo o local), utiliza um "Pool de Aplicativos". Em qualquer sistema Windows moderno, esse pool é, por padrão, **64-bit**. Para que o seu site consiga gerar o relatório para o usuário final, o processo 64-bit do IIS precisa da versão **64-bit** do runtime do Crystal.

Após instalar esses dois componentes, sua máquina estará pronta para trabalhar com Crystal Reports. Próxima fase!

---
### Fase 2: Configuração do IIS Local

Mapeando o projeto `ExtintorWS` no seu IIS local para que ele seja encontrado pelo restante da solução.

1. **Habilitar o IIS (se ainda não estiver ativo):**

    - Vá em "Painel de Controle" -> "Programas" -> "Ativar ou desativar recursos do Windows".
    
    - Na janela que abrir, navegue até "Serviços de Informações da Internet" e certifique-se de que as seguintes caixas estão marcadas:
    
        - Em "Ferramentas de Gerenciamento da Web", marque "Console de Gerenciamento do IIS".
        
        - Em "Serviços da World Wide Web" -> "Recursos de Desenvolvimento de Aplicativos", marque **"ASP.NET 4.8"** (ou a versão mais alta do ASP.NET que aparecer).
    
    - Clique em OK e aguarde a instalação.

2. **Preparar a Pasta do Projeto:**

    - Localize no seu computador a pasta com os arquivos do projeto `ExtintorWS`.
    
    - Copie toda essa pasta para dentro do diretório raiz do IIS. O caminho final deve ser algo como: `C:\inetpub\wwwroot\ExtintorWS`.

3. **Criar a Aplicação no IIS:**

    - Abra o "Gerenciador de Serviços de Informações da Internet (IIS)". Você pode encontrá-lo pesquisando por "IIS" no menu Iniciar.
    
    - No painel esquerdo, navegue até `Seu Computador -> Sites -> Default Web Site`.
    
    - Clique com o botão direito em "Default Web Site" e selecione **"Adicionar Aplicativo..."**.
    
    - Preencha os campos da janela:
    
        - **Alias:** Digite `ExtintorWS`. Este nome é **crucial**, pois é a parte do URL que o sistema usa para encontrar a aplicação (`/ExtintorWS/`).
        
        - **Pool de Aplicativos:** Deixe o padrão (`DefaultAppPool`) ou selecione um que esteja configurado para `.NET Framework v4.0`.
        
        - **Caminho Físico:** Clique no botão `...` e navegue até a pasta que você copiou no passo anterior: `C:\inetpub\wwwroot\ExtintorWS`.
    
    - Clique em **OK**.

Pronto! Agora o seu IIS local sabe que quando uma requisição chegar para `http://localhost/ExtintorWS/...`, ele deve procurar os arquivos dentro da pasta `C:\inetpub\wwwroot\ExtintorWS`. Próxima fase!

---
### Fase 3: Verificação e Teste Final

1. **Verifique o `Web.config`:** Abra o arquivo `Web.config` dentro da pasta `C:\inetpub\wwwroot\ExtintorWS`. Verifique se ele contém as seções de `handlers` e `assemblies` para o Crystal Reports. O instalador geralmente cuida disso, mas é bom confirmar.

	- Por via das dúvidas copie e cole dentro da pasta `ExtintorWS` o arquivo `web.config` que está localizado na pasta que recebeu (`Crystal Configuration`).
	
	- Para verificar detalhes das modificações realizar no `web.config` do `ExtintorWS` vá para a seção abaixo nomeada como **Observações**.

2. **Teste Direto:** Abra seu navegador e tente acessar o URL do relatório diretamente, usando `localhost`: `http://localhost/ExtintorWS/Relatorios/Manutencao_Responsavel_Tecnico_Novo.aspx`

3. **Resultado:** Se tudo foi configurado corretamente, em vez de um erro 404 ou outro qualquer, você deverá ver a página do relatório (que pode pedir algum parâmetro ou dar outro tipo de erro específico, o importante é ela ser **encontrada**). Se aparecer uma página em branco provavelmente a instalação e configuração deu certo.

Próxima fase!

---
### Fase 4: Testes no Visual Studio

Neste momento, pode abrir o Visual Studio e prosseguir com os testes desejados. O roteiro abaixo são apenas sugestões/observações.

1. Confira se o `ProjetoExtintor` está mapeado na sua máquina.

	- Abra a solução;
	- Clique para abrir a hierarquia do projeto `http://localhost/ExtintorWS`. Na pasta `Relatorios` você pode editar o código de produção dos relatórios e na pasta `rpt` você pode editar o design deles.

2. Abra a Solução `SGE-Extintor`, rode a aplicação e tente emitir um relatório.

Fim! Uma vez que o acesso direto funcione, a sua aplicação principal (`SGE.Extintor.Site`) também conseguirá encontrar o relatório e montá-lo devidamente, possibilitando sua impressão, por exemplo.

```
>>>>>> ATENÇÃO!!

>>>>>> O atual roteiro pode sofrer variações ao longo do tempo.
>>>>>> Mantenha-se ativamente atualizado.
```

---
# >> Observações:

## 1. Resumo das Alterações no `Web.config` (`ExtintorWS`):

O objetivo principal das alterações foi atualizar o projeto, que estava configurado para uma versão muito antiga do Crystal Reports, para que ele pudesse funcionar com os novos componentes compatíveis com o Visual Studio 2022 e o .NET Framework 4.8.

#### 1.1 Atualização da Versão das Assemblies do Crystal Reports

- **O Quê:** Todas as referências às DLLs do Crystal Reports (ex: `CrystalDecisions.Web`, `CrystalDecisions.Shared`, etc.) tiveram sua versão alterada de **`13.0.2000.0`** para **`13.0.4000.0`**.

- **Porquê:** Para corresponder à nova versão do Crystal Reports que você instalou (SP35 ou superior). A versão `.2000` é da época do Visual Studio 2010. A versão `.4000` é a correta para o ambiente .NET 4.x. Sem essa atualização, a aplicação tentaria carregar DLLs antigas que não existem mais, causando erros de configuração.

#### 1.2 Adição de Redirecionamento de Assemblies (`bindingRedirect`)

- **O Quê:** Foi adicionada uma seção `<assemblyBinding>` completa dentro da tag `<runtime>` para todas as principais DLLs do Crystal Reports.

- **Porquê:** Isso funciona como uma "apólice de seguro". Mesmo que alguma parte do código compilado ainda peça a versão antiga (`13.0.2000.0`), este redirecionamento força o .NET a carregar a nova versão (`13.0.4000.0`) em seu lugar. É a maneira padrão e mais robusta de resolver conflitos de versão de DLLs no .NET.

#### 1.3 Remoção da Referência Explícita ao `log4net`

- **O Quê:** A linha `<add assembly="log4net, Version=1.2.10.0, ...">` foi removida/comentada da seção `<assemblies>`.

- **Porquê:** A aplicação estava tentando carregar uma versão específica e customizada do `log4net` que acompanhava o Crystal Reports antigo. A nova versão do Crystal Reports utiliza uma versão diferente e mais moderna do `log4net`. Manter a referência antiga causava um conflito de assinatura (`PublicKeyToken`), resultando em um erro de carregamento. Ao remover a linha, permitimos que a aplicação utilize a DLL correta do `log4net` que está na pasta `bin` do projeto.

#### 1.4 Atualização do Target Framework

- **O Quê:** A propriedade `targetFramework` na seção `<compilation>` foi alterada de `"4.0"` para `"4.8"`.

- **Porquê:** Para alinhar o projeto de relatórios com o restante da sua solução e com os requisitos dos novos componentes do Crystal Reports. Isso garante máxima compatibilidade, segurança e acesso aos recursos mais recentes do .NET Framework que sua aplicação principal já utiliza.

## 2. Distribuição

Os arquivos "CR for Visual Studio SP38 64b installer (VS 2022 and above)" e "CR for Visual Studio SP38 CR Runtime 64-bit MSI" são **gratuitos para uso e redistribuição** com suas aplicações, mas com algumas condições.

Aqui está um detalhamento:

##### CR for Visual Studio SP38 64b installer (VS 2022 and above)

- Este é o instalador do desenvolvedor. Ele se integra ao Visual Studio para que você possa criar e editar relatórios do Crystal Reports diretamente em seu ambiente de desenvolvimento.

- **Licenciamento:** É **gratuito**. Você pode baixá-lo e usá-lo para desenvolver suas aplicações sem custo.

##### CR for Visual Studio SP38 CR Runtime 64-bit MSI

- Este é o pacote de tempo de execução (runtime). Ele contém as bibliotecas necessárias para que suas aplicações, que utilizam Crystal Reports, possam ser executadas em máquinas onde o Visual Studio não está instalado (como servidores ou computadores de clientes).

- **Licenciamento:** Também é gratuito para redistribuir junto com suas aplicações, tanto para uso interno na sua empresa quanto para distribuição a terceiros.