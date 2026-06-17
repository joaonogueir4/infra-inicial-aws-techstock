# Infraestrutura Inicial AWS - Projeto TechStock

Este repositório contém a documentação da arquitetura de nuvem inicial para o projeto TechStock, implementada inteiramente no ambiente Amazon Web Services (AWS) na região East-US. O foco do desenho é garantir o isolamento da camada de dados e backend em sub-redes totalmente privadas.

---

## 🗺️ Papel dos Serviços na Topologia Atual

Abaixo está descrita a função estratégica de cada componente dentro da rede, detalhando como o tráfego flui entre o cliente e a aplicação.

### 1. Camada de Borda (Internet Pública)
* **Amazon S3 (`s3-frontend-techstock-kelvin`):** Armazena e hospeda de forma estática os arquivos do frontend (HTML, CSS, JS) com os quais o cliente interage diretamente via navegador.
* **Internet Gateway (`igw-techstock-kelvin`):** Funciona como a única porta de comunicação da VPC com o mundo externo, permitindo que a aplicação frontend envie requisições de API para dentro da rede da AWS.

### 2. Rede Isolada (VPC - Bloco CIDR: 10.0.0.0/16)
* **SUB-REDE PÚBLICA (`10.0.1.0/24`):** Uma zona cinzenta com acesso à internet, usada exclusivamente para abrigar o gateway de saída privada.
  * **NAT Gateway (`nat-techstock-kelvin`):** Permite que os servidores da sub-rede privada façam conexões de saída com a internet (para baixar atualizações e patches de segurança), mas impede de forma absoluta que qualquer computador de fora inicie uma conexão direta com eles.
* **SUB-REDE PRIVADA (`10.0.2.0/24`):** O coração seguro do projeto, totalmente isolado do acesso público direto.
  * **Amazon EC2 Backend (`ec2-backend-techstock-kelvin`):** Instância virtual que processa a lógica de negócios da aplicação e recebe as requisições de API vindas da internet.
  * **Amazon RDS (`rds-database-techstock-kelvin`):** Banco de dados relacional gerenciado que armazena as informações do sistema de forma segura, comunicando-se estritamente com o backend.
  * **Amazon EC2 Monitoring (`ec2-monitoring-techstock-kelvin`):** Servidor dedicado à coleta de métricas e telemetria, monitorando a saúde e o desempenho do banco de dados e do backend de forma interna.

---

## 📈 Observações Técnicas e Sugestões de Melhoria

Avaliando a topologia atual e os dados gerados na calculadora de custos da AWS, foram identificados pontos críticos que podem ser otimizados para um ambiente real ou acadêmico:

### 1. Redução Drástica de Custos (FinOps)
* **Otimização do NAT Gateway:** O NAT Gateway configurado equivale a **44,6% do custo fixo total** da infraestrutura ($66,16 mensais de um total de $148,10). Como se trata de um ambiente de laboratório, esse recurso gerenciado pode ser substituído por uma **NAT Instance** (uma máquina virtual comum `t3.micro` configurada para roteamento), o que reduziria esse custo específico para menos de $5 mensais.
* **Ajuste de Escopo do Banco de Dados:** O Amazon RDS está provisionado no modelo Multi-AZ (Multi-Zona de Disponibilidade) custando $71,03. Para fins de teste e validação de desenvolvimento, a alteração para **Single-AZ** manteria o funcionamento idêntico e cortaria esse valor pela metade.

### 2. Evolução de Arquitetura (Próximos Passos)
* **Alta Disponibilidade do Backend:** O servidor backend (`ec2-backend-techstock-kelvin`) opera atualmente como uma instância única, gerando um ponto único de falha (SPOF). A arquitetura ideal futura deve contar com um Application Load Balancer (ALB) na sub-rede pública distribuindo a carga para duas ou mais instâncias de backend em zonas de disponibilidade diferentes.
* **Preparação para Escopo Multi-Cloud:** O bloco CIDR principal desta VPC foi fixado em `10.0.0.0/16`. Caso esta infraestrutura precise se conectar futuramente com outro provedor de nuvem (como a Microsoft Azure) via VPN, deve-se garantir que o ambiente parceiro utilize uma faixa de rede completamente diferente (ex: `192.168.0.0/16`) para evitar conflitos de roteamento e falhas no tráfego dos túneis.
