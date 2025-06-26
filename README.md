# 🚀 AWS Multi-AZ Web Application Deployment

## 📋 Descrição

Arquitetura AWS resiliente e escalável para aplicações web com alta disponibilidade, distribuída em duas Zonas de Disponibilidade. Inclui Application Load Balancer, Auto Scaling Group, RDS Multi-AZ, S3 Storage e monitoramento completo via CloudWatch/SNS.

**Recursos principais:**
- ✅ **Alta Disponibilidade**: Multi-AZ deployment com failover automático
- ✅ **Escalabilidade**: Auto Scaling baseado em CPU
- ✅ **Segurança**: Security Groups restritivos e subnets privadas
- ✅ **Monitoramento**: Alertas automáticos e métricas em tempo real
- ✅ **Backup**: RDS automated backups e S3 versioning

## 🏗️ Arquitetura

### Componentes Principais

- **VPC**: Rede privada virtual isolada (10.0.0.0/16)
- **Multi-AZ**: Distribuição em 2 Zonas de Disponibilidade (1a e 1b)
- **Load Balancer**: Application Load Balancer para distribuição de tráfego
- **Auto Scaling**: Grupo de Auto Scaling baseado em métricas de CPU
- **Database**: RDS com configuração Multi-AZ para alta disponibilidade
- **Storage**: S3 Bucket para armazenamento de objetos
- **Monitoring**: CloudWatch e SNS para monitoramento e alertas

## 🌐 Topologia de Rede

### Subnets Públicas
- **Public Subnet 1** (AZ-1a): Hosts ALB targets, EC2 instances e NAT Gateway
- **Public Subnet 2** (AZ-1b): Hosts ALB targets, EC2 instances e NAT Gateway

### Subnets Privadas
- **Private Subnet 1** (AZ-1a): RDS Primary Database
- **Private Subnet 2** (AZ-1b): RDS Standby Database (Multi-AZ)

### Conectividade
- **Internet Gateway**: Acesso à internet para recursos públicos
- **NAT Gateways**: Acesso seguro à internet para recursos privados
- **Route Tables**: Roteamento otimizado para tráfego público e privado

## 💻 Recursos de Computação

### EC2 Instances
- **Quantidade**: 2 instâncias (uma por AZ)
- **Distribuição**: Load balancing automático via ALB
- **Scaling**: Auto Scaling Group com políticas baseadas em CPU

### Auto Scaling Group
- **Launch Template**: Configuração padronizada para novas instâncias
- **Scaling Policy**: Baseado em métricas de CPU do CloudWatch
- **Health Checks**: Verificação automática de saúde das instâncias

## 🗄️ Banco de Dados

### RDS Configuration
- **Engine**: MySQL (porta 3306)
- **High Availability**: Multi-AZ deployment
- **Primary**: Localizado na AZ-1a (Private Subnet 1)
- **Standby**: Localizado na AZ-1b (Private Subnet 2)
- **Replication**: Replicação automática entre Primary e Standby

## 🪣 Armazenamento

### S3 Bucket
- **Propósito**: Armazenamento de arquivos estáticos e backups
- **Acesso**: Direto das instâncias EC2
- **Durabilidade**: 99.999999999% (11 9's)

## 🛡️ Segurança

### Security Groups

#### ALB Security Group
- **Inbound**: HTTP (80) e HTTPS (443) de qualquer origem
- **Outbound**: Tráfego para instâncias EC2

#### EC2 Security Group
- **Inbound**: HTTP (80) apenas do ALB Security Group
- **Outbound**: Acesso controlado conforme necessário

#### RDS Security Group
- **Inbound**: MySQL (3306) apenas do EC2 Security Group
- **Outbound**: Restrito

## 📊 Monitoramento e Alertas

### CloudWatch
- **Métricas**: CPU, memória, rede, disk I/O
- **Alarms**: Configurados para scaling e alertas
- **Logs**: Centralizados para análise

### SNS (Simple Notification Service)
- **Email Notifications**: Alertas automáticos para administradores
- **Integration**: Conectado ao CloudWatch para alertas críticos

## 🔧 Pré-requisitos

### Ferramentas Necessárias
- AWS CLI configurado
- Terraform ou CloudFormation
- Acesso à conta AWS com permissões adequadas
- Par de chaves EC2 para acesso SSH

### Permissões IAM Necessárias
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "rds:*",
        "elasticloadbalancing:*",
        "autoscaling:*",
        "s3:*",
        "cloudwatch:*",
        "sns:*",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
```

## 🚀 Implantação

### Passo 1: Preparação
```bash
# Clone o repositório
git clone <repository-url>
cd aws-multi-az-deployment

# Configure as credenciais AWS
aws configure
```

### Passo 2: Configuração de Variáveis
```bash
# Copie o arquivo de exemplo
cp terraform.tfvars.example terraform.tfvars

# Edite as variáveis conforme necessário
vim terraform.tfvars
```

### Passo 3: Deploy com Terraform
```bash
# Inicialize o Terraform
terraform init

# Planeje a implantação
terraform plan

# Aplique as mudanças
terraform apply
```

## ⚙️ Configuração

### Variáveis Principais
```hcl
# terraform.tfvars
vpc_cidr = "10.0.0.0/16"
availability_zones = ["us-west-2a", "us-west-2b"]
instance_type = "t3.medium"
min_size = 2
max_size = 10
desired_capacity = 2
db_instance_class = "db.t3.micro"
```

### Tags Recomendadas
```hcl
common_tags = {
  Environment = "production"
  Project     = "web-app"
  Owner       = "devops-team"
  Terraform   = "true"
}
```

## 📈 Monitoramento

### Métricas Principais
- **CPU Utilization**: Alerta quando > 80%
- **Memory Utilization**: Alerta quando > 85%
- **Database Connections**: Monitoramento contínuo
- **Load Balancer Response Time**: Alerta quando > 2s

### Dashboards CloudWatch
- Application Performance Dashboard
- Infrastructure Health Dashboard
- Database Performance Dashboard

## 🔧 Manutenção

### Backups Automáticos
- **RDS**: Backup automático diário com retenção de 7 dias
- **S3**: Versionamento habilitado
- **EC2**: Snapshots semanais dos volumes EBS

### Updates e Patches
- **AMI**: Atualizações mensais programadas
- **RDS**: Janela de manutenção configurada
- **Security Patches**: Aplicação automática

## 🚨 Troubleshooting

### Problemas Comuns

#### Instâncias não passam no Health Check
```bash
# Verificar logs da aplicação
aws logs tail /aws/ec2/application --follow

# Verificar status do ALB
aws elbv2 describe-target-health --target-group-arn <arn>
```

#### Database Connection Issues
```bash
# Verificar security groups
aws ec2 describe-security-groups --group-ids <sg-id>

# Verificar status do RDS
aws rds describe-db-instances --db-instance-identifier <db-id>
```

### Logs Importantes
- `/var/log/cloud-init.log`: Inicialização da instância
- `/var/log/httpd/`: Logs do servidor web
- CloudWatch Logs: Logs centralizados da aplicação

## 💰 Estimativa de Custos

### Componentes de Custo (us-west-2)
- **EC2 Instances** (2 x t3.medium): ~$60/mês
- **RDS Multi-AZ** (db.t3.micro): ~$25/mês
- **Application Load Balancer**: ~$20/mês
- **NAT Gateways** (2): ~$90/mês
- **S3 Storage**: Variável conforme uso
- **CloudWatch**: ~$5/mês

**Estimativa Total**: ~$200/mês (pode variar com uso)

## 🔐 Segurança

### Boas Práticas Implementadas
- ✅ Subnets privadas para databases
- ✅ Security Groups restritivos
- ✅ NACLs configuradas
- ✅ Encryption at rest e in transit
- ✅ Backup automático habilitado
- ✅ Monitoramento de segurança

### Recomendações Adicionais
- Implementar AWS WAF
- Configurar VPC Flow Logs
- Usar AWS Secrets Manager para credenciais
- Implementar AWS GuardDuty

## 📚 Recursos Adicionais

### Documentação
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/)
- [RDS Multi-AZ Deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

### Ferramentas Úteis
- [AWS Cost Calculator](https://calculator.aws/)
- [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)
- [CloudFormation Designer](https://console.aws.amazon.com/cloudformation/designer)

## 🤝 Contribuição

### Como Contribuir
1. Fork o repositório
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

### Reportar Issues
- Use as templates de issue fornecidas
- Inclua logs e detalhes do ambiente
- Especifique steps para reproduzir o problema

**Desenvolvido por:** [![Linkedin Badge](https://img.shields.io/badge/-Eduardo-blue?style=flat-square&logo=linkedin&logoColor=white&link=https://www.linkedin.com/in/seu-perfil)](https://www.linkedin.com/in/jos%C3%A9-eduardo-cardoso-webhaker) [![GitHub Badge](https://img.shields.io/badge/-EduCard-black?style=flat-square&logo=github&logoColor=white&link=https://github.com/seu-usuario)](https://github.com/J-Edu-Cardoso)

## 📞 Contato

Este é um projeto de estudos pessoal. Para dúvidas ou sugestões:

- **GitHub Issues**: Abra uma issue neste repositório
- [![Linkedin Badge](https://img.shields.io/badge/-Eduardo-blue?style=flat-square&logo=linkedin&logoColor=white&link=https://www.linkedin.com/in/seu-perfil)](https://www.linkedin.com/in/jos%C3%A9-eduardo-cardoso-webhaker)
- [![GitHub Badge](https://img.shields.io/badge/-EduCard-black?style=flat-square&logo=github&logoColor=white&link=https://github.com/seu-usuario)](https://github.com/J-Edu-Cardoso)

---

## 📄 Licença

Este projeto está licenciado sob a MIT License - veja o arquivo [LICENSE](LICENSE) para detalhes.

---

**Projeto desenvolvido como estudo de arquiteturas AWS por Eduardo Cardoso**  
**Última atualização**: Junho 2025  
**Versão**: 1.0.0
