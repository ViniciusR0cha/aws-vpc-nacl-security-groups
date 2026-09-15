# aws-vpc-nacl-security-groups
Laboratório prático da AWS Skill Builder sobre configuração e extensão de regras em Network ACLs e Security Groups na VPC.

# AWS Networking: Security Groups vs Network ACLs (Hands-on Lab)

Este repositório contém a documentação prática do laboratório **"Configure Network Access Control Lists and Security Groups in Amazon VPC"** da AWS Skill Builder.

O objetivo deste projeto foi entender na prática a diferença entre a camada de segurança no nível de sub-rede (**NACLs**) e no nível de instância (**Security Groups**), além de realizar o diagnóstico e correção de falhas de conectividade (troubleshooting).

---

## 📌 Arquitetura & Conceitos Chave

| Recurso | Nível de Atuação | Estado (*State*) | Regras Suportadas |
| :--- | :--- | :--- | :--- |
| **Network ACL (NACL)** | Sub-rede (*Subnet*) | **Stateless** (exige regras explícitas de entrada e saída) | ALLOW e DENY (processadas em ordem numérica) |
| **Security Group (SG)** | Instância / ENI | **Stateful** (se a entrada for permitida, o retorno é automático) | Apenas ALLOW (negação implícita no final) |

---

## 🛠️ Etapas do Projeto

### 1. Configuração de Network ACLs
* **Net ACL 1** (Associada à Subnet Pública 1 / Instância A):
  * Configurada para permitir tráfego de Entrada (*Inbound*) e Saída (*Outbound*) para `0.0.0.0/0`.
* **Net ACL 2** (Associada à Subnet Pública 2 / Instância B):
  * Inicialmente configurada apenas com regra de Saída (*Outbound*), **sem nenhuma regra de Entrada (*Inbound*)** explicitada além do `DENY` padrão.

### 2. Configuração de Security Groups
* Edição de **SG A** e **SG B** para liberar os seguintes protocolos de entrada:
  * **HTTP** (Porta 80)
  * **HTTPS** (Porta 443)
  * **All ICMP - IPv4** (Para testes de `ping`)

---

## 🔍 Troubleshooting & Diagnóstico de Conectividade

### Cenário 1: Teste Inicial (Com Falha)
A partir do **AWS Systems Manager Session Manager** na Instância A:
1. **Ping para Internet (`www.aws.amazon.com`):** **Sucesso** (0% packet loss). Demonstrando que a Net ACL 1 e o SG A permitiam tráfego ICMP.
2. **Ping para a Instância B:** **Falha** (`100% packet loss`).

![Falha de Conectividade](testando%20conectividade%20loss.png)

* **Diagnóstico:** Como o Security Group B já possuía a regra de entrada ICMP liberada, a causa do bloqueio estava na **Net ACL 2**, que não tinha regra de entrada (*Inbound*) ativa para permitir a chegada dos pacotes ICMP na Subnet Pública 2.

---

### Cenário 2: Correção e Validação (Sucesso)
1. Foi adicionada a Regra 10 na **Net ACL 2** permitindo **Todo o tráfego** vindo de `0.0.0.0/0`.
2. Repetição do comando `ping -c 3 <IP-da-Instancia-B>` na Instância A.

![Conectividade Restabelecida](lab%20ok.png)

* **Resultado:** 0% de perda de pacotes e tempo de resposta ~0.3ms, confirmando que a fluxo entre sub-redes foi liberado.

---

## 📸 Imagens do Painel da AWS

* **Associação das Network ACLs:**
  ![Net ACLs](ACLs%201%20e%202.png)

* **Regras de Entrada dos Security Groups:**
  ![Regras SG](regras%20de%20entrada%20SGB%20B.png)

---

## 🎯 Aprendizados Práticos

* **Comportamento Stateless:** Demonstrado na prática que a falta de uma regra de entrada em uma NACL bloqueia conexões mesmo que o Security Group interno permita a entrada e a NACL permita a saída.
* **Ferramentas de Acesso:** Uso do **AWS Session Manager** via console para gerenciamento seguro e acesso a instâncias EC2 sem depender de chaves SSH expostas na internet.
