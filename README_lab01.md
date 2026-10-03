# Lab 01 — Superfície de Ataque, Reconhecimento e Investigação de Endpoint

## Objetivo

Este laboratório teve como objetivo compreender, na prática, como um serviço cria uma superfície de ataque, como essa superfície pode ser identificada por meio de reconhecimento de rede e como o processo responsável pode ser investigado diretamente no endpoint.

O laboratório também foi utilizado para relacionar as perspectivas ofensiva e defensiva através do fluxo que vou utilizar em meus estudos do dia a dia:

**Reconhecimento → Evidência → Investigação → Mitigação → Reteste**

---

## Ambiente

- Kali Linux em máquina virtual
- Python 3
- Nmap
- `ss`
- `ps`
- `/proc`
- `curl`

Todo o laboratório foi realizado em ambiente próprio e controlado.

---

## Conceitos estudados

- Ativo
- Ameaça
- Vulnerabilidade
- Risco
- Controle de segurança
- Superfície de ataque
- Socket
- Porta TCP
- Listener
- Reconhecimento ativo
- Enumeração de serviço
- Telemetria
- Processo
- PID e PPID
- File Descriptor
- `/proc`
- Redução da superfície de ataque
- Reteste

---

## 1. Baseline

Antes de criar qualquer serviço, foi verificado o estado atual dos sockets do sistema:

```bash
ss -tuln
```

Também foi realizado um scan TCP contra o endereço de loopback:

```bash
nmap -sT 127.0.0.1
```

Neste momento, não havia serviços TCP escutando nas portas analisadas.

O Nmap retornou as portas como fechadas.

Esse estado foi utilizado como baseline para comparar as alterações realizadas posteriormente.

---

## 2. Criação de um serviço local

Foi criado um pequeno servidor HTTP utilizando um módulo nativo do Python:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Nesse estado, o serviço estava associado somente ao endereço de loopback:

```text
127.0.0.1:8000
```

Isso significa que o serviço poderia ser acessado pelo próprio host, mas não diretamente através das demais interfaces IPv4 da máquina.

A presença do listener foi confirmada utilizando:

```bash
ss -ltn
```

---

## 3. Reconhecimento

Foi realizada uma varredura específica da porta TCP 8000:

```bash
nmap -sT -p 8000 127.0.0.1
```

O Nmap identificou a porta como aberta.

Esse teste demonstrou a diferença entre duas perspectivas:

- `ss`: visão interna do sistema operacional sobre seus sockets.
- Nmap: visão obtida através de interação com a pilha de rede.

Uma porta aberta representa uma superfície de interação, mas não significa automaticamente que exista uma vulnerabilidade.

---

## 4. Ampliação da superfície de exposição

O servidor foi reiniciado utilizando:

```bash
python3 -m http.server 8000 --bind 0.0.0.0
```

Nesse contexto, `0.0.0.0` faz com que o processo escute em todos os endereços IPv4 locais disponíveis.

A alteração foi verificada utilizando:

```bash
ss -ltnp
```

O serviço passou a aparecer associado a:

```text
0.0.0.0:8000
```

Isso aumentou sua superfície potencial de exposição.

---

## 5. Investigação do processo

Após identificar o socket, foi investigado o processo responsável por ele.

O `ss` permitiu relacionar:

```text
porta → socket → processo → PID
```

O processo foi então analisado utilizando:

```bash
ps -fp <PID>
```

Além disso, foram consultadas informações como:

- PID
- PPID
- usuário
- EUID
- EGID
- grupo
- prioridade
- comando executado

---

## 6. Investigação com /proc

O pseudo-filesystem `/proc` foi utilizado para obter evidências adicionais sobre o processo.

### File descriptors

```bash
ls -l /proc/<PID>/fd/
```

Foi possível verificar que um dos file descriptors do processo apontava para um socket.

Isso permitiu corroborar a informação apresentada anteriormente pelo `ss`.

Fluxo observado:

```text
Processo Python
      ↓
     PID
      ↓
File Descriptor
      ↓
    Socket
      ↓
TCP :8000
```

### Executável

Também foi analisado:

```bash
readlink /proc/<PID>/exe
```

Isso permitiu identificar o executável real associado ao processo.

A investigação mostrou a importância de não depender de uma única fonte de evidência.

---

## 7. Service Detection

Foi utilizado o recurso de detecção de serviço do Nmap:

```bash
nmap -sV -p 8000 <alvo>
```

Diferentemente de uma simples verificação de porta, o `-sV` envia probes adicionais para tentar identificar qual serviço ou software está respondendo.

Durante o teste, o servidor HTTP registrou respostas como:

```text
404 File Not Found
501 Unsupported Method
```

Essas respostas demonstraram que uma atividade de enumeração pode gerar evidências diferentes de uma requisição HTTP comum.

---

## 8. Comparação com uma requisição legítima

Foi realizada uma requisição HTTP utilizando:

```bash
curl http://127.0.0.1:8000/
```

O servidor registrou a requisição HTTP normalmente.

Isso permitiu comparar:

```text
Requisição HTTP comum
        vs
Probes de identificação de serviço
```

Os dois tipos de interação produziram telemetria diferente na aplicação.

---

## 9. Mitigação

Como o serviço não precisava estar disponível através das demais interfaces IPv4, sua exposição foi reduzida.

O servidor foi alterado de:

```text
0.0.0.0:8000
```

para:

```text
127.0.0.1:8000
```

utilizando:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

A aplicação continuou funcionando localmente, mas deixou de estar disponível pelas demais interfaces IPv4.

---

## 10. Reteste

Após aplicar a mitigação, foram realizados novos scans.

O objetivo era verificar se o controle realmente produziu o comportamento esperado.

O serviço continuou acessível através do loopback:

```text
127.0.0.1:8000
```

mas deixou de aparecer como aberto através do endereço IPv4 da interface de rede.

O reteste é importante porque uma mitigação não deve ser considerada eficaz apenas porque uma configuração foi alterada.

Ela precisa ser validada.

---

## Purple Team Workflow

O laboratório completo pode ser representado como:

```text
BASELINE
   ↓
CRIAÇÃO DO SERVIÇO
   ↓
EXPOSIÇÃO
   ↓
RECONHECIMENTO
   ↓
ENUMERAÇÃO
   ↓
TELEMETRIA
   ↓
INVESTIGAÇÃO
   ↓
MITIGAÇÃO
   ↓
RETESTE
```

Ou, resumidamente:

**Attack → Evidence → Investigation → Defense → Retest**

---

## Principais aprendizados

Uma porta aberta não representa automaticamente uma vulnerabilidade.

Antes de chegar a uma conclusão, é necessário investigar:

```text
O que está exposto?
        ↓
Qual processo é responsável?
        ↓
Quem iniciou esse processo?
        ↓
Qual executável está sendo utilizado?
        ↓
A exposição é necessária?
        ↓
Que evidências existem?
        ↓
Qual controle pode reduzir o risco?
        ↓
O controle realmente funcionou?
```

Também ficou evidente a importância da correlação entre diferentes fontes de informação.

Neste laboratório foram utilizadas evidências provenientes de:

- sockets;
- processos;
- file descriptors;
- `/proc`;
- Nmap;
- logs da aplicação.

A utilização de múltiplas evidências ajuda a fortalecer uma hipótese antes de transformá-la em uma conclusão.

---

Pra tudo tem seu começo, independente do que possam falar, então apenas comece! 
