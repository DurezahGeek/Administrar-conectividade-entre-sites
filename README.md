# Administrar conectividade entre sites

## 🧠 Emparelhamento de VNets com Arquitetura Hub-and-Spoke no Azure

## 🔷 Conceito Geral

[imagem1](https://github.com/DurezahGeek/Administrar-conectividade-entre-sites/blob/main/srcACS/1.png)

No Azure, o **VNet Peering** pode ser comparado a uma infraestrutura *on-premises* com **DMZ (Zona Desmilitarizada)**, onde serviços expostos (como servidores web, firewalls e proxies) são isolados por segurança.

---

## 🏗️ Arquitetura Hub-and-Spoke

### 🔹 Hub

Centraliza serviços compartilhados como:

- Azure Firewall / NVAs  
- Gateways (VPN, ExpressRoute)  
- DNS personalizado  
- Servidores de inspeção/controle de tráfego  
- Logs centralizados  

> 📌 Funciona como a DMZ no modelo tradicional.  
> 📌 Controla e roteia o tráfego de entrada/saída.

### 🔹 Spokes

VNets isoladas que representam ambientes ou workloads (Dev, Teste, Produção).

- Cada **Spoke**:
  - Se conecta **apenas ao Hub** via peering.
  - **Não se comunica com outros Spokes**, a menos que configurado.

---

## 📌 Vantagens do Modelo Hub-and-Spoke

- 🔐 **Mais segurança** e controle sobre tráfego leste-oeste.
- 🚫 Reduz a propagação lateral de ataques.

---

## 🔐 Segurança no Emparelhamento

Se todas as VNets estiverem emparelhadas entre si, um invasor pode se mover lateralmente.

✅ **Recomendação**: Use o modelo **Hub-and-Spoke**, onde todo tráfego passa por **firewalls e regras de segurança**.

---

## 🔁 Comunicação: Unidirecional vs Bidirecional

- O emparelhamento pode ser:
  - ➡️ **Unidirecional**: VNet A → VNet B.
  - 🔄 **Bidirecional**: VNet A ↔ VNet B (necessário emparelhamento em ambas).
  - ➡️ Exemplo:
  - Se VNet2 é emparelhada com VNet3, apenas VNet2 pode enviar dados para VNet3.
  - Para comunicação de volta, é necessário emparelhamento também em VNet3.

⚠️ Sem emparelhamento nos dois sentidos, respostas (ex: ping, RDP, API) não funcionarão, mesmo que o tráfego de ida chegue.

### ✅ Dicas:
- **Bidirecional**: Para comunicações cliente-servidor, APIs, banco de dados.
- **Unidirecional**: Para envio de logs, replicações.

---

## 🌍 Global VNet Peering

- Permite emparelhar VNets em regiões diferentes (ex: *Brazil South ↔ East US*).
- Tipos:
  - 🌐 **Regional Peering**: Mesma região.
  - 🌐 **Global Peering**: Regiões diferentes.

### 🛡️ Características:

- Emparelha VNets entre **assinaturas** ou até **tenants diferentes**.
- Utiliza o **backbone privado** da Microsoft.
- Tráfego é **privado, seguro e de alta performance**.

---

## 🔁 Trânsito de Gateway (Gateway Transit) com Hub-and-Spoke
[imagem2](https://github.com/DurezahGeek/Administrar-conectividade-entre-sites/blob/main/srcACS/2.png)

### 🏗️ 1. VNet Hub

- Centraliza a conectividade com on-premises via **VPN Gateway**.
- Configurações:
  - Emparelhada com Spokes.
  - Marcar: `Allow Gateway Transit`.

### 🌐 2. VNets Spoke

- Sem gateway próprio.
- Marcar: `Use Remote Gateway` para usar o gateway da Hub.

### 🏢 3. Rede On-Premises
- Data center físico da empresa.
- Conectada via VPN Gateway da Hub.

### 🔀 Fluxo de tráfego
- Exemplo:
   - Um servidor na Spoke 1 precisa acessar o ambiente local:

```txt
Spoke 1 → Hub → VPN Gateway → Rede Local
```
✅ A Spoke usa o gateway da Hub para alcançar a rede local.

## ✅ Por que usar Gateway Transit?

| Benefício       | Descrição                                                           |
|------------------|----------------------------------------------------------------------|
| 💰 Economia       | Um único gateway para várias VNets.                                   |
| 🔐 Centralização  | Inspeção e controle centralizados.                                    |
| 📉 Simplicidade   | Menos recursos para gerenciar.                                        |
| 🔁 Flexibilidade  | VNets em diferentes regiões compartilham o gateway da Hub.            |

---

## 🔄 Emparelhamento de VNets (VNet Peering)

Conexão privada e direta entre redes virtuais.

Permite:

- ✅ Acesso direto entre Vnets
- ✅ Encaminhamento de tráfego
- ✅ Uso de Gateway Transit

> 🔍 O status deve aparecer como **"Conectado"** no portal.

---

## 🔁 Encadeamento de Serviços (Service Chaining)
[imagem3](https://github.com/DurezahGeek/Administrar-conectividade-entre-sites/blob/main/srcACS/3.png)

Permite redirecionar tráfego de uma VNet para:

- 🔸 NVA (Network Virtual Appliance)
- 🔸 Firewall
- 🔸 Gateway VPN

### 📌 Usos comuns:

- 🔍 Inspeção de tráfego
- 📍 Roteamento personalizado
- 🔐 Políticas centralizadas

---

## 🔸 Rotas no Azure

### 📦 Rotas de Sistema (System Routes)
[imagem4](https://github.com/DurezahGeek/Administrar-conectividade-entre-sites/blob/main/srcACS/4.png)

- Criadas automaticamente pelo Azure.
- Garantem comunicação entre:
  - VMs e sub-redes
  - Acesso à Internet
  - VPNs e ExpressRoute
- ⚠️ Não podem ser removidas

---

### 🔧 UDRs (User Defined Routes)

- Substituem rotas de sistema.
- Permitem:
  - 🔁 Forçar tráfego por firewall ou NVA
  - 🚫 Bloquear saída para Internet
  - 📏 Aplicar políticas de segurança

> 🧪 **Importante:** Sempre testar e validar o tráfego após aplicar UDRs!

---

## 🔌 Pontos de Extremidade

[imagem5](https://github.com/DurezahGeek/Administrar-conectividade-entre-sites/blob/main/srcACS/5.png)

### 🔸 Service Endpoints

- Conecta VNets com serviços **PaaS** de forma privada.
- Tráfego não sai da rede da Microsoft.

#### ✅ Vantagens:

- 🔐 Mais segurança
- 🛑 Sem necessidade de IP público
- ⚙️ Aplicação rápida

---

### 🔒 Azure Private Link
[imagem6](https://github.com/DurezahGeek/Administrar-conectividade-entre-sites/blob/main/srcACS/6.png)

- Fornece acesso **privado via IP da VNet**.
- Mais seguro do que Service Endpoints.

#### ✅ Benefícios:

- 🔐 Tráfego **nunca sai** da Microsoft
- 🔒 Evita exposição pública
- 🌐 Pode ser acessado por VNets emparelhadas ou redes locais

---
## 📥 Tabelas de Rotas no Azure

### 🛠️ Como criar:
- Vá até "Route Tables" no portal Azure.
    - Clique em Create.
    - Defina nome, região, grupo de recursos.
    - Associe a tabela à sub-rede desejada.
    - Adicione rotas personalizadas conforme a necessidade.



## 🔁 Peering de VNets e Roteamento

### 🔸 Para habilitar comunicação entre VNets:
Criar o Peering entre as redes.

- Marcar:
- ✅ "Permitir tráfego entre redes virtuais".
- ✅ "Permitir encaminhamento de tráfego" se quiser que uma VNet atue como roteadora (Service Chaining, por exemplo).

## 🧠 Reforço com Perguntas-Chave

| Pergunta | Resposta |
|---------|----------|
| 🔹 Qual tipo de emparelhamento é usado entre VNets na mesma região? | Emparelhamento Regional |
| 🔹 E entre regiões diferentes? | Emparelhamento Global |
| 🔹 Vantagem do peering entre VNets? | Comunicação segura e privada |
| 🔹 O que acontece com peering unidirecional? | Comunicação sem resposta |
| 🔹 Para que serve uma UDR? | Controlar rota do tráfego |
| 🔹 Como usar gateway de outra VNet? | "Allow Gateway Transit" + "Use Remote Gateway" |
| 🔹 Por que usar Hub-and-Spoke? | Segmentação, segurança, controle |
| 🔹 Função das rotas de sistema? | Comunicação padrão entre recursos |
| 🔹 UDR substitui o quê? | Rotas padrão |
| 🔹 Quando usar Service Endpoint? | Para acesso privado a PaaS via sub-rede |
| 🔹 Diferença entre Endpoint e Private Link? | Endpoint = sub-rede; Private Link = IP privado |
| 🔹 Inspeção entre VNets? | UDR → NVA no Hub |
| 🔹 Diagnóstico de rede no Azure? | Network Watcher |
| 🔹 Configuração para roteamento entre VNets? | "Permitir encaminhamento de tráfego" |

---

## 🔗 Links Úteis para Estudo

- 📘 [Peering de Redes Virtuais no Azure](https://learn.microsoft.com/pt-br/azure/virtual-network/virtual-network-peering-overview)  
- 📘 [Gerenciar Peering](https://learn.microsoft.com/pt-br/azure/virtual-network/virtual-network-manage-peering?tabs=peering-portal#requirements-and-constraints)  
- 📘 [FAQ sobre VNets](https://learn.microsoft.com/pt-br/azure/virtual-network/virtual-networks-faq)  
- 📘 [Rotas Personalizadas (UDR)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)  
- 📘 [Azure Private Link](https://docs.microsoft.com/azure/private-link/)  
- 🧪 [Lab AZ-700 - Global VNet Peering](https://mslabs.cloudguides.com/guides/AZ-700%20Lab%20Simulation%20-%20Connect%20two%20Azure%20virtual%20networks%20using%20global%20virtual%20network%20peering)  
- 📗 [Guia AZ-104 - Exercício 9](https://mslabs.cloudguides.com/en-us/guides/AZ-104%20Exam%20Guide%20-%20Microsoft%20Azure%20Administrator%20Exercise%209)

---

> 🧩 Este conteúdo é parte do estudo para certificações **AZ-104**, com foco em redes e conectividade no Azure.


