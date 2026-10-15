# Cisco Packet Tracer — Configure o Controle de Acesso

## Visão geral

Neste laboratório, a rede simulada é configurada para controlar quem pode acessar serviços e quais operações cada usuário pode realizar. A atividade utiliza o Cisco Packet Tracer para trabalhar com:

- **AAA/RADIUS e Wi-Fi:** autenticação dos laptops na rede sem fio.
- **DHCP:** obtenção automática de endereços IP pelos laptops.
- **E-mail:** configuração de contas e validação do envio e recebimento de mensagens por SMTP e POP3.
- **FTP:** autenticação de usuários, transferência de arquivos e validação de permissões de leitura, gravação, listagem, exclusão e renomeação.

O foco principal é a diferença entre **autenticação** (confirmar a identidade de quem está tentando acessar) e **autorização** (definir o que essa identidade pode fazer depois de autenticada).

## Ambiente e escopo

| Item | Descrição |
|---|---|
| Ferramenta | Cisco Packet Tracer |
| Atividade | `Packet Tracer - Configure o Controle de Acesso` |
| Modo | Físico ou Lógico |
| Rede Wi-Fi | SSID `HQ-INT` |
| Rede IP esperada dos laptops | `192.168.50.0/24` |
| Servidor FTP | `192.168.75.2` |
| Serviços trabalhados | AAA/RADIUS, DHCP, SMTP, POP3 e FTP |

> **Limite da evidência:** o material recebido é uma exportação HTML do enunciado da Cisco. Ele não inclui o arquivo `.pkt`, capturas de tela do estado final nem uma confirmação de que os testes foram executados. Os resultados abaixo são os resultados esperados definidos ou descritos pelo roteiro.

## Conceitos técnicos

### Autenticação, autorização e AAA

- **Autenticação:** verifica a identidade apresentada por um usuário, por exemplo, por meio de nome de usuário e senha.
- **Autorização:** determina quais recursos e operações o usuário autenticado pode utilizar.
- **AAA:** sigla para *Authentication, Authorization and Accounting* — autenticação, autorização e contabilização. O exercício configura o serviço AAA no servidor identificado como `AAA-RADIUS` e utiliza contas individuais para autenticação sem fio.
- **RADIUS:** protocolo normalmente utilizado para centralizar a autenticação de acesso à rede. Neste cenário, o servidor AAA/RADIUS é usado pelo exercício para o acesso Wi-Fi.
- **WPA2:** mecanismo de segurança da rede sem fio selecionado nos laptops. O roteiro pede o uso de credenciais de usuário em vez de tratar o acesso como uma rede aberta.
- **DHCP:** fornece automaticamente parâmetros de rede, incluindo endereço IP. O laboratório pede que os dois laptops recebam endereços da rede `192.168.50.0/24`.

### Serviços de e-mail

- **SMTP:** utilizado para enviar mensagens de e-mail.
- **POP3:** utilizado pelo cliente para recuperar mensagens armazenadas no servidor.
- **Conta de e-mail:** relaciona endereço, nome de usuário e senha. O teste funcional é enviar uma mensagem e confirmar sua chegada à caixa postal do destinatário.

### FTP e privilégios

FTP é um protocolo de transferência de arquivos. No laboratório, os usuários recebem permissões distintas no servidor. Um usuário pode autenticar-se com sucesso e, ainda assim, não ter autorização para executar determinada operação. A conta `malia` exemplifica isso: ela não deve ter o privilégio de excluir arquivos, mas deve poder renomeá-los.

> FTP tradicional não protege credenciais e conteúdo em trânsito com criptografia. Esta atividade é uma simulação educacional, não uma recomendação para expor FTP em uma rede real.

## Procedimento documentado

As caixas de seleção abaixo estão inicialmente desmarcadas porque o HTML recebido não comprova a conclusão prática dessas ações.

### Parte 1 — Configurar e usar credenciais de autenticação AAA

#### 1. Habilitar AAA e criar as contas

No armário de fiação (*Wiring Closet*) da sede, abrir o servidor `AAA-RADIUS`, acessar **Services → AAA**, ativar o serviço e cadastrar as contas indicadas no roteiro.

| Usuário | Senha indicada no roteiro |
|---|---|
| `user1` | `PASSuser1!` |
| `user2` | `PASSuser2!` |

Essas credenciais pertencem ao laboratório simulado. Não as reutilize em sistemas reais.

#### 2. Configurar `HQ-Laptop-1`

1. Abrir `HQ-Laptop-1` e acessar **Config → Wireless0**.
2. Definir o SSID como `HQ-INT`.
3. Selecionar `WPA2` na área de autenticação.
4. Informar o usuário `user1` e a senha correspondente.
5. Selecionar DHCP na configuração IP.
6. Confirmar que o endereço recebido pertence à rede `192.168.50.0/24`.

#### 3. Configurar `HQ-Laptop-2`

Repetir o procedimento de `HQ-Laptop-1`, usando as credenciais de `user2`. Verificar também se o endereço IP atribuído pertence à rede `192.168.50.0/24`.

O roteiro observa que pode ser necessário alternar entre configuração estática e DHCP, ou usar **Verificar resultados**, para que o Packet Tracer atualize a configuração. Isso é uma dica de convergência do simulador, não evidência por si só de que a autenticação foi bem-sucedida.

**Validação da Parte 1**

- [ ] O serviço AAA está ativado e as duas contas foram cadastradas.
- [ ] `HQ-Laptop-1` utiliza `HQ-INT`, WPA2 e as credenciais de `user1`.
- [ ] `HQ-Laptop-2` utiliza `HQ-INT`, WPA2 e as credenciais de `user2`.
- [ ] Ambos os laptops receberam endereços dentro de `192.168.50.0/24`.
- [ ] Os itens da Parte 1 foram conferidos em **Verificar resultados**.

### Parte 2 — Configurar e usar serviços de e-mail

#### 1. Habilitar os serviços e criar as contas

No servidor de e-mail, acessar **Services → EMAIL**, ativar **SMTP** e **POP3**, definir o domínio como `mail.cyberhq.com` e criar as contas abaixo.

| Usuário | Senha indicada no roteiro |
|---|---|
| `HQuser1` | `Cisco123!` |
| `HQuser2` | `Cisco123~` |
| `BRuser1` | `Cisco123-` |
| `BRuser2` | `Cisco123 +`* |

\* O HTML exportado apresenta a senha de `BRuser2` com um espaço antes do sinal `+` (`Cisco123 +`). Como espaços podem ser relevantes em uma senha, confira a grafia exibida no próprio Packet Tracer caso a autenticação falhe. Não assuma que o espaço é ignorado.

#### 2. Configurar os clientes

No cliente de e-mail de cada dispositivo, abrir **Desktop → Email** e informar os dados da tabela. O roteiro fornece o servidor de entrada e saída como `email.cyberhq.com`.

| Dispositivo | Nome exibido | Endereço de e-mail | Usuário | Senha indicada no roteiro |
|---|---|---|---|---|
| `PC 1-1` | Suk-Yi | `HQuser1@mail.cyberhq.com` | `HQuser1` | `Cisco123!` |
| `PC 2-3` | Ajulo | `BRuser1@mail.cyberhq.com` | `BRuser1` | `Cisco123-` |
| `HQ-Laptop-1` | Malia | `BRuser2@mail.cyberhq.com` | `BRuser2` | `Cisco123 +`* |
| `Net-Admin` | Cisco | `HQuser2@mail.cyberhq.com` | `HQuser2` | `Cisco123~` |

**Observação sobre os nomes de servidor:** o roteiro especifica o domínio de e-mail como `mail.cyberhq.com`, mas orienta preencher os campos de servidor de entrada e saída com `email.cyberhq.com`. São valores diferentes no próprio material exportado. Para reproduzir a atividade, comece pelos valores que a interface/enunciado solicita e valide a comunicação; não trate essa diferença como resolvida sem verificar o arquivo `.pkt`.

#### 3. Testar o envio e o recebimento

1. No `PC 1-1`, abrir o cliente de e-mail e selecionar **Escrever**.
2. Enviar uma mensagem com assunto e conteúdo à escolha para `BRuser1@mail.cyberhq.com`.
3. No `PC 2-3` (Ajulo), abrir o cliente e selecionar **Receber**.
4. Confirmar que a mensagem de Suk-Yi chegou à caixa postal.

**Validação da Parte 2**

- [ ] SMTP e POP3 estão ativados.
- [ ] As quatro contas foram criadas no servidor.
- [ ] Os quatro clientes estão configurados com as contas correspondentes.
- [ ] Uma mensagem foi enviada do `PC 1-1` para `PC 2-3`.
- [ ] A mensagem foi recebida e aberta no dispositivo de destino.

O roteiro informa que pode haver um pequeno atraso de convergência antes de o Packet Tracer mostrar a confirmação de envio. A confirmação visual de envio deve ser complementada pela verificação da caixa de entrada do destinatário.

### Parte 3 — Configurar e usar serviços de FTP

#### 1. Ativar o FTP e criar usuários

No servidor FTP, abrir **Services → FTP** e ativar o serviço. Criar as contas conforme a tabela do exercício.

| Usuário | Senha indicada no roteiro | Privilégios configurados |
|---|---|---|
| `sukyi` | `cisco123` | `RWDNL` — escrita, leitura, exclusão, renomeação e listagem |
| `ajulo` | `cisco321` | `RWDNL` — escrita, leitura, exclusão, renomeação e listagem |
| `malia` | `cisco123` | `RWNL` — escrita, leitura, renomeação e listagem; **sem exclusão** |

Conferir cada conta após adicioná-la. A restrição essencial é que `malia` **não** tenha o privilégio de exclusão (*Delete*).

#### 2. Baixar um arquivo usando `Net-Admin`

No dispositivo `Net-Admin`, abrir **Desktop → Command Prompt** e conectar ao servidor FTP:

```text
ftp 192.168.75.2
```

Quando solicitado, autenticar com o usuário `sukyi` e a senha definida no roteiro. Depois, executar os comandos da sessão FTP:

```text
dir
get aMessage.txt
```

- `ftp 192.168.75.2`: inicia uma sessão FTP com o servidor no endereço informado.
- `dir`: lista os arquivos disponíveis no servidor.
- `get aMessage.txt`: baixa o arquivo para o dispositivo cliente.
- `quit` ou `bye`: encerra a sessão FTP, conforme suportado pelo cliente do simulador.

Depois de sair da sessão, abrir o **Text Editor** no `Net-Admin` e visualizar `aMessage.txt`. A resposta indicada pelo material Cisco é:

```text
Saudações. Você acessou o servidor FTP.
```

#### 3. Criar e enviar um novo arquivo

1. No **Text Editor** do `Net-Admin`, criar um documento de texto com uma mensagem à escolha.
2. Salvar o arquivo com o nome exato `aMessage_new.txt`.
3. Abrir novamente o Prompt de Comando e autenticar-se no servidor FTP como `sukyi`.
4. Enviar o arquivo com:

```text
put aMessage_new.txt
```

5. Encerrar a sessão FTP.

O `put` envia um arquivo local para o servidor. A transferência deve ser validada pela resposta do cliente ou pela listagem dos arquivos do servidor.

#### 4. Validar as permissões da conta `malia`

No `HQ-Laptop-1`, abrir **Desktop → Command Prompt**, conectar ao servidor `192.168.75.2` e autenticar-se como `malia` com a senha do roteiro. Em seguida, tentar excluir o arquivo enviado:

```text
delete aMessage_new.txt
```

**Resultado esperado:** a exclusão deve falhar, pois `malia` não tem o privilégio `D` (*Delete*).

Ainda na sessão FTP, testar a renomeação:

```text
rename aMessage_new.txt aMessage_rename.txt
```

**Resultado esperado:** a renomeação deve funcionar, porque o usuário possui o privilégio `N` (*Rename*). Essa comparação mostra que as permissões são verificadas por operação: ter acesso ao FTP não significa ter autorização para realizar todas as ações.

**Validação da Parte 3**

- [ ] O serviço FTP está ativado.
- [ ] `sukyi`, `ajulo` e `malia` foram criados com os privilégios corretos.
- [ ] `Net-Admin` conectou ao servidor `192.168.75.2` usando `sukyi`.
- [ ] `dir` listou os arquivos e `get` baixou `aMessage.txt`.
- [ ] O conteúdo de `aMessage.txt` foi conferido.
- [ ] `aMessage_new.txt` foi criado localmente e enviado por `put`.
- [ ] A tentativa de exclusão com `malia` foi recusada.
- [ ] A renomeação para `aMessage_rename.txt` foi permitida.

## Resumo dos resultados esperados

| Teste | Resultado esperado | O que demonstra |
|---|---|---|
| Login Wi-Fi como `user1` e `user2` | Autenticação aceita e IP via DHCP na rede `192.168.50.0/24` | Identidade e acesso à rede |
| Envio de e-mail de Suk-Yi para Ajulo | Mensagem chega ao `PC 2-3` | Funcionamento do envio e recebimento de e-mail |
| FTP com `sukyi` | Listar, baixar e enviar arquivos | Uso de permissões suficientes para operações de arquivo |
| FTP com `malia`: `delete` | Operação recusada | A autenticação não concede automaticamente permissão de exclusão |
| FTP com `malia`: `rename` | Operação permitida | Uma conta pode ter permissão para renomear sem poder excluir |

## Análise de segurança

O laboratório ilustra três controles importantes:

1. **Identidades individuais:** contas separadas tornam possível distinguir usuários e aplicar credenciais diferentes.
2. **Serviços autenticados:** Wi-Fi, e-mail e FTP solicitam credenciais, em vez de depender apenas de conectividade de rede.
3. **Privilégio mínimo:** a conta `malia` recebe somente as operações previstas para ela. Retirar o privilégio de exclusão reduz as ações que essa conta pode executar, mesmo quando o login é válido.

Em uma implementação real, também seria necessário usar credenciais exclusivas e fortes, proteger os dados em trânsito com protocolos seguros (por exemplo, alternativas criptografadas ao FTP tradicional), manter registros de autenticação e revisar permissões periodicamente. Essas recomendações são contexto de segurança adicional; o exercício em si trabalha a configuração simulada descrita acima.

## Evidências para adicionar

Adicione capturas de tela suas ao diretório `images/`. Sugestão de organização:

```text
.
├── README.md
└── images/
    ├── 01-topologia.png
    ├── 02-servidor-aaa.png
    ├── 03-wifi-dhcp.png
    ├── 04-email-recebido.png
    ├── 05-usuarios-ftp.png
    ├── 06-transferencia-ftp.png
    └── 07-permissao-malia.png
```

Pontos recomendados para capturar:

- Visão da topologia no Packet Tracer.
- Serviço AAA ativado e contas cadastradas (oculte dados de autenticação se compartilhar uma captura publicamente).
- SSID, método de autenticação e endereço DHCP dos laptops.
- Mensagem recebida por Ajulo.
- Contas FTP e privilégios, especialmente a ausência do privilégio de exclusão em `malia`.
- Saída da tentativa de `delete` recusada e da renomeação permitida.

Depois de salvar as imagens no diretório `images/`, você pode inserir, por exemplo:

```markdown
![Topologia da atividade no Packet Tracer](images/01-topologia.png)
```

## Conclusão

O propósito do laboratório é demonstrar que controlar o acesso exige mais do que aceitar um nome de usuário e uma senha. As credenciais identificam a conta; as permissões determinam as operações permitidas. A configuração de AAA para o Wi-Fi, o teste de e-mail e a comparação entre os privilégios de `sukyi` e `malia` conectam esses conceitos a serviços de rede concretos.


## Referência

- Cisco Networking Academy — *Packet Tracer - Configure o Controle de Acesso*
