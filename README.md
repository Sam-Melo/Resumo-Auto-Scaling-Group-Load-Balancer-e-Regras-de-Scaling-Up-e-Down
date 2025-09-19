# Resumo - Auto Scaling Group, Load Balancer e Regras de Scaling Up e Down

---

## Auto Scaling Group  

O **AWS Auto Scaling** é um serviço da Amazon que ajusta automaticamente a quantidade de recursos usados por uma aplicação.  
Ele serve para aumentar ou diminuir a capacidade conforme a demanda, evitando desperdício e garantindo que a aplicação não fique lenta quando tiver muito acesso.  

### O que aprendi  
- **Capacidade mínima, desejada e máxima:** No começo eu não entendia direito a diferença, acabei confundindo o valor desejado com o mínimo. Depois percebi que o “desejado” é o número de instâncias que o ASG tenta manter sempre, já o “mínimo” garante que nunca caia abaixo disso, e o “máximo” limita para não subir demais.  
- **Regras de Scaling:** Vi que é essencial configurar bem as métricas do CloudWatch, porque em uma das tentativas eu deixei CPU em 30%, e o ASG começou a criar várias instâncias sem necessidade. Isso mostra que tem que ajustar com calma, senão o custo aumenta sem necessidade.  
- **Integração com Load Balancer:** O ASG sozinho funciona, mas quando conecta com o ALB a disponibilidade fica bem melhor. Se uma instância cai, o Load Balancer para de mandar tráfego pra ela, e o ASG sobe outra automaticamente. Isso faz o sistema ficar mais estavel, mesmo com falhas.  

### Como criar  
1. No console AWS vá em **EC2** → **Auto Scaling Groups** → **Criar Grupo de Auto Scaling**.  

   ![print1](https://github.com/user-attachments/assets/28707df6-2da7-4fef-b5af-59babc4266a8)  

2. Configure seu **Auto Scaling Group**:  

   ![print2](https://github.com/user-attachments/assets/f8b3bd31-3bbe-4007-946d-ea4a94199bb0)  

3. Criar uma **Launch Template**:  
   - Vá em **EC2 → Launch Templates**.  
   - Clique em **Create launch template**.  
   - Nome: `meu-template-asg`.  
   - Escolha AMI e tipo de instância (ex.: `t2.micro`).  
   - Configure **Key Pair**, **Security Group** e rede (VPC e Subnet).  
   - Salve.  

4. Criar o **Auto Scaling Group**:  
   - Vá em **EC2 → Auto Scaling Groups**.  
   - Clique em **Create Auto Scaling group**.  
   - Nome: `meu-asg-test`.  
   - Selecione o **Launch Template** criado antes.  
   - Escolha a VPC e as Subnets (sempre pelo menos em 2 AZs).  
   - (Opcional) Associe a um **Load Balancer**.  
   - Configure a **Health Check** (padrão EC2, mas pode ser ELB).  
   - Defina capacidade mínima, desejada e máxima (ex.: min 1, desired 2, max 4).  
   - Configure políticas de scaling:  
     - Exemplo: aumentar se CPU > 70% por 5 min.  
     - Exemplo: diminuir se CPU < 30% por 10 min.  
   - Clique em **Create Auto Scaling group**.  

---

## Load Balancer  

O **Elastic Load Balancer (ELB)** é um serviço da AWS que distribui automaticamente o tráfego de entrada entre vários destinos, como instâncias EC2, containers e IPs.  
Ele também faz checagem de saúde dos alvos registrados, e só manda requisições para os que estão funcionando bem.  

### Tipos  

- **Application Load Balancer (ALB):**  
  - Para HTTP/HTTPS.  
  - Nome: `meu-alb`.  
  - Esquema: `Internet-facing`.  
  - VPC: escolha a sua.  
  - Sub-redes: duas públicas (em AZs diferentes).  
  - SG: liberar tráfego HTTP (porta 80) de `0.0.0.0/0`.  
  - Listener: HTTP : 80.  
  - Target Group: instâncias, protocolo HTTP, porta 80, health check em `/`.  

- **Network Load Balancer (NLB):**  
  - Para conexões **TCP, UDP, TLS**.  
  - Nome: `meu-nlb`.  
  - Esquema: Internet-facing ou Internal.  
  - Listener: protocolo desejado (ex.: TCP:443).  
  - Target Group: do mesmo protocolo/porta do serviço.  

- **Gateway Load Balancer (GWLB):**  
  - Para appliances de terceiros (firewalls, IDS, etc).  
  - Nome: `meu-gwlb`.  
  - Sub-redes: onde o tráfego vai passar.  
  - Target Group: compatível com GENEVE.  

### O que aprendi  
- **Sub-redes corretas:** Aprendi que quando o Load Balancer é Internet-facing ele tem que estar em sub-redes públicas, se colocar em privadas não funciona, e o acesso da internet dá timeout.  
- **Security Groups:** O LB tem o próprio SG, que precisa liberar porta 80 (ou 443). Sem isso o navegador não carrega a aplicação.  
- **Health Checks:** O ALB só envia tráfego pra instâncias saudáveis. Já recebi erro 502 porque todas estavam unhealthy.  
- **Escalabilidade automática:** O LB acompanha o tráfego, não precisa mexer manual toda vez, isso ajuda muito quando o acesso varia bastante.  

### Como criar  
1. No console AWS vá em **EC2 → Balanceadores de Carga → Criar Load Balancer**.  

   ![print3](https://github.com/user-attachments/assets/a87a8391-c1e7-4200-a52b-8f12d4289153)  

2. Criando o **Application Load Balancer (ALB)**:  

   ![print4](https://github.com/user-attachments/assets/7a2011f5-238a-41af-9f15-50ecaaf53811)  

---

## Regras de Scaling Up e Down  

O **Scaling Up e Scaling Down** são as regras dentro do Grupo de Auto Scaling que controlam quando o sistema deve aumentar ou diminuir a quantidade de instâncias.  
- **Scaling Up:** adiciona instâncias quando a demanda sobe (ex.: CPU > 70%).  
- **Scaling Down:** reduz instâncias quando a demanda cai (ex.: CPU < 30%).  
Isso mantém equilíbrio entre custo e desempenho, sem precisar mexer manualmente.  

### O que aprendi  
- **Importância da métrica certa:** Configurei CPU em 30% e o grupo criou instâncias demais sem necessidade. Aprendi que tem que ajustar com calma.  
- **Equilíbrio entre limites:** Se configurar mal, as instâncias sobem e descem toda hora. Melhor usar thresholds realistas e tempo de avaliação maior.  
- **Integração com o Balanceador:** Quando o ALB marca uma instância como não saudável, o grupo cria outra automaticamente.  
- **Cooldown/Tempo de inicialização:** Sem configurar, o grupo conta a instância como pronta antes dela estar, atrapalhando o balanceamento.  

### Como criar  
1. Vá em **EC2 → Grupos de Auto Scaling**.  
2. Clique no seu grupo.  
3. Vá até a aba **Escalonamento automático**.  
4. Clique em **Adicionar política de escalonamento**.  
5. Escolha o tipo de política:  
   - **Acompanhamento de destino (Target Tracking):** define uma métrica alvo (ex.: CPU em 50%).  
   - **Escalonamento por etapas (Step Scaling):** regras detalhadas, ex.: CPU > 70% = +1 instância, CPU > 85% = +2.  
6. Configure os **Alarmes do CloudWatch**:  
   - Scale Up: CPU > 70% por 5 min → adicionar 1 instância.  
   - Scale Down: CPU < 30% por 10 min → remover 1 instância.  
7. Ajuste o **Tempo de inicialização da instância** (ex.: 300s).  
8. Salve e teste gerando carga para ver o scaling acontecendo.  

---
