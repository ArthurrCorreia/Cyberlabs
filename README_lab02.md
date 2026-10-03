# Lab 02 — Execução, Beaconing, Persistência e Investigação em Linux

## Objetivo

Este laboratório teve como objetivo compreender, de forma prática e controlada, como comportamentos comuns em incidentes podem ser observados e investigados em um endpoint Linux.

O foco foi correlacionar:

**Execução → Comunicação periódica → Evidência → Persistência → Investigação → Remoção → Reteste**

Nenhum malware real foi utilizado. O comportamento foi simulado com um script benigno em uma máquina virtual própria.

---

## Ambiente

- Kali Linux em máquina virtual
- Bash
- Python 3
- `curl`
- `ps`
- `pgrep`
- `/proc`
- `stat`
- `ss`
- `tcpdump`
- `systemd --user`

Todo o laboratório foi realizado em ambiente próprio, controlado e autorizado.

---

## Conceitos estudados

- Execution
- Processos e subprocessos
- PID e PPID
- Command line
- File descriptors
- `/proc`
- Artefatos de filesystem
- Telemetria de aplicação
- Telemetria de rede
- Beaconing
- Command and Control como hipótese
- Persistência
- systemd user services
- Reinício automático
- Timeline
- Corroboração de evidências
- Remoção
- Reteste
- Fato x hipótese x conclusão

---

## 1. Servidor de laboratório

Foi iniciado um servidor HTTP local para funcionar como destino de comunicação do agente benigno:

```bash
python3 -m http.server 9000 --bind 127.0.0.1
```

O serviço ficou disponível somente no endereço de loopback.

Fluxo simplificado:

```text
Agente benigno
     ↓
HTTP
     ↓
127.0.0.1:9000
     ↓
Servidor Python
```

---

## 2. Criação do agente benigno

Foi criado o arquivo:

```bash
/tmp/lab02-agent.sh
```

com o seguinte comportamento:

```bash
#!/bin/bash

mkdir -p "$HOME/.config/lab02"

date > "$HOME/.config/lab02/started.txt"
whoami >> "$HOME/.config/lab02/started.txt"
hostname >> "$HOME/.config/lab02/started.txt"

while true; do
    curl -s "http://127.0.0.1:9000/?host=$(hostname)&user=$(whoami)" > /dev/null
    sleep 15
done
```

O script foi tornado executável com:

```bash
chmod +x /tmp/lab02-agent.sh
```

e executado manualmente:

```bash
/tmp/lab02-agent.sh
```

---

## 3. Comportamentos simulados

O script foi criado para produzir alguns comportamentos observáveis.

### Criação de artefatos

```text
~/.config/lab02/
└── started.txt
```

O arquivo continha informações básicas do host e do usuário.

### Execução periódica

A cada aproximadamente 15 segundos, o script realizava uma requisição HTTP ao servidor local.

Esse padrão foi utilizado para simular um comportamento semelhante a beaconing.

---

## 4. Observação do processo

Foi utilizada a árvore de processos:

```bash
ps -ef --forest
```

O objetivo era observar a relação entre:

```text
bash
└── lab02-agent.sh
    ├── curl
    └── sleep
```

Como `curl` era executado por um período muito curto, nem sempre ele podia ser capturado por uma única execução de `ps`.

Isso demonstrou uma limitação importante:

> ferramentas como `ps` e `ss` normalmente representam apenas um estado momentâneo do sistema.

---

## 5. Identificação do PID

O processo foi localizado utilizando:

```bash
pgrep -af lab02-agent
```

Argumentos:

- `-a`: exibe a linha de comando.
- `-f`: pesquisa a linha de comando completa.

Com o PID identificado, foi possível aprofundar a investigação.

---

## 6. Investigação com /proc

O executável associado ao processo foi consultado com:

```bash
readlink /proc/<PID>/exe
```

A command line foi analisada utilizando:

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

Essa etapa permitiu correlacionar:

```text
PID
 ↓
Processo
 ↓
Executável
 ↓
Command line
```

---

## 7. Investigação dos artefatos

O arquivo criado pelo agente foi analisado com:

```bash
stat ~/.config/lab02/started.txt
```

e:

```bash
cat ~/.config/lab02/started.txt
```

O `stat` permitiu observar metadados como:

- owner;
- permissões;
- tamanho;
- timestamps.

Esses dados podem contribuir para a construção de uma timeline.

---

## 8. Telemetria de aplicação

O servidor HTTP registrava as requisições realizadas pelo agente aproximadamente a cada 15 segundos.

Isso permitiu observar um padrão periódico de comunicação.

Fato observado:

```text
requisição
↓
~15 segundos
↓
requisição
↓
~15 segundos
↓
requisição
```

Esse padrão pode levantar a hipótese de beaconing.

Entretanto:

> periodicidade sozinha não prova Command and Control.

Softwares legítimos também podem realizar comunicação periódica.

---

## 9. Telemetria de rede com tcpdump

Para observar a comunicação diretamente na interface de rede, foi utilizado:

```bash
sudo tcpdump -i lo -nn tcp port 9000
```

Argumentos:

- `-i lo`: captura na interface de loopback;
- `-nn`: não resolve nomes de hosts nem serviços;
- `tcp port 9000`: filtra tráfego TCP relacionado à porta 9000.

O `tcpdump` permitiu observar cada comunicação em tempo real.

Dessa forma, o mesmo comportamento passou a ser corroborado por diferentes fontes:

```text
Endpoint
  ↓
ps / /proc

Rede
  ↓
tcpdump

Aplicação
  ↓
logs HTTP
```

---

## 10. Persistência com systemd

Depois da análise inicial, foi criado um serviço de usuário do systemd.

Diretório:

```bash
mkdir -p ~/.config/systemd/user
```

Unidade criada:

```text
~/.config/systemd/user/lab02-agent.service
```

Conteúdo:

```ini
[Unit]
Description=Lab 02 benign beaconing agent

[Service]
Type=simple
ExecStart=/tmp/lab02-agent.sh
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

---

## 11. Significado da unidade

### ExecStart

```ini
ExecStart=/tmp/lab02-agent.sh
```

Define qual comando será executado pelo serviço.

### Restart

```ini
Restart=always
```

Instrui o systemd a iniciar um novo processo caso o processo principal termine.

Importante:

> isso não impede que o processo morra; o processo pode terminar normalmente. O systemd apenas cria outro processo em seguida.

### RestartSec

```ini
RestartSec=5
```

Define o intervalo antes de reiniciar o serviço.

### WantedBy

```ini
WantedBy=default.target
```

Permite associar a unidade ao target padrão do usuário quando ela é habilitada.

---

## 12. Habilitação do serviço

O systemd foi instruído a reler as unidades:

```bash
systemctl --user daemon-reload
```

Depois, a unidade foi habilitada e iniciada:

```bash
systemctl --user enable --now lab02-agent.service
```

O estado foi verificado com:

```bash
systemctl --user status lab02-agent.service
```

---

## 13. Processo x mecanismo de persistência

O laboratório permitiu diferenciar dois conceitos.

### Processo

Uma instância de um programa em execução.

### Mecanismo de persistência

A configuração responsável por permitir que aquela execução retorne futuramente.

Neste caso:

```text
Processo
   ↓
lab02-agent.sh

Persistência
   ↓
systemd user service habilitado
```

Além disso:

```text
processo termina
      ↓
Restart=always
      ↓
systemd cria outro processo
```

Isso representa reinício automático.

Já a unidade habilitada permite que o serviço volte a ser iniciado em sessões/inicializações futuras do contexto do usuário.

---

## 14. Teste de respawn

O PID atual foi identificado:

```bash
pgrep -af lab02-agent
```

Em seguida, o processo foi encerrado:

```bash
kill <PID>
```

Após alguns segundos, o processo voltou a aparecer com um novo PID.

Isso demonstrou que:

> matar apenas o processo não eliminava a causa responsável por recriá-lo.

A causa estava no mecanismo configurado pelo systemd.

---

## 15. Investigação do artefato de persistência

A unidade foi analisada com:

```bash
cat ~/.config/systemd/user/lab02-agent.service
```

e:

```bash
stat ~/.config/systemd/user/lab02-agent.service
```

Foram observados:

- caminho do `ExecStart`;
- política de restart;
- intervalo de restart;
- target associado;
- timestamps;
- owner;
- permissões.

Em uma investigação real, também seria necessário analisar:

- hash e origem do executável;
- package manager;
- logs do systemd;
- usuário responsável;
- processo pai;
- conexões de rede;
- comportamento esperado do host;
- presença do mesmo serviço em outros endpoints.

---

## 16. Remoção

Após concluir a investigação, a persistência foi removida de forma controlada:

```bash
systemctl --user disable --now lab02-agent.service
```

Depois:

```bash
rm ~/.config/systemd/user/lab02-agent.service
```

e:

```bash
systemctl --user daemon-reload
```

Os artefatos do laboratório também foram removidos:

```bash
rm -rf ~/.config/lab02
rm /tmp/lab02-agent.sh
```

O servidor HTTP foi encerrado manualmente.

---

## 17. Reteste

Após a remoção, foram utilizados:

```bash
pgrep -af lab02-agent
```

```bash
systemctl --user status lab02-agent.service
```

```bash
ss -ltn
```

O objetivo do reteste foi confirmar que:

- o agente não estava mais em execução;
- a unidade não estava mais ativa/carregada;
- o serviço HTTP do laboratório também havia sido encerrado.

---

## 18. Fato, hipótese e conclusão

O laboratório reforçou uma disciplina importante para investigação.

### Fato observado

Aquilo que a evidência realmente demonstra.

Exemplo:

```text
O processo realizou conexões HTTP aproximadamente a cada 15 segundos.
```

### Hipótese

Uma possível explicação para o fato.

Exemplo:

```text
O padrão pode representar beaconing.
```

### Conclusão

Uma interpretação sustentada por evidências suficientes e contexto.

Exemplo:

```text
A comunicação fazia parte do agente benigno criado para o laboratório.
```

É importante evitar transformar uma única evidência em conclusão definitiva.

---

## 19. Purple Team Workflow

O fluxo completo do laboratório foi:

```text
EXECUÇÃO
   ↓
PROCESSO
   ↓
CRIAÇÃO DE ARTEFATOS
   ↓
COMUNICAÇÃO PERIÓDICA
   ↓
TELEMETRIA DE APLICAÇÃO
   ↓
TELEMETRIA DE REDE
   ↓
PERSISTÊNCIA
   ↓
INVESTIGAÇÃO
   ↓
REMOÇÃO
   ↓
RETESTE
```

De forma resumida:

**Execution → Evidence → Persistence → Investigation → Removal → Retest**

---

## Principais aprendizados

### 1. Comunicação periódica não prova C2

Beaconing é uma hipótese que precisa ser validada com contexto e outras evidências.

### 2. Um processo não é a mesma coisa que sua persistência

Encerrar o PID não remove necessariamente a configuração que pode recriá-lo.

### 3. Mecanismos legítimos podem ser abusados

systemd é uma ferramenta legítima de gerenciamento de serviços.

O fato de um serviço existir não significa automaticamente que ele seja malicioso.

### 4. Corroboração aumenta a confiança

Neste laboratório, o mesmo comportamento pôde ser observado por:

- processos;
- `/proc`;
- filesystem;
- logs HTTP;
- `tcpdump`;
- systemd.

### 5. Reteste faz parte da mitigação

Uma alteração não deve ser considerada eficaz apenas porque foi aplicada.

Ela precisa ser validada.

---

## Segurança e ética

Nenhum malware real foi utilizado.

Todos os comportamentos foram simulados por meio de um script benigno em uma máquina virtual própria e controlada.

Nenhum sistema externo ou de terceiros foi utilizado.

---

## Próximos passos

Continuar aprofundando a relação entre:

**Delivery → Execution → Persistence → Command and Control → Evidência → Investigação → Defesa**

sempre separando:

**Fato → Hipótese → Validação → Conclusão**
