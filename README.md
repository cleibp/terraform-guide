<h1 align="center"> 🚀 Terraform </h1>

## 📑 Sumário

- [O que é o Terraform](#-o-que-é-o-terraform)
- [Como funciona](#-como-funciona)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Primeiros passos](#-primeiros-passos)
- [Fluxo de trabalho](#-fluxo-de-trabalho)
- [Linguagem HCL](#-linguagem-hcl)
- [Providers](#-providers)
- [Resources](#-resources)
- [Variáveis](#-variáveis)
- [Outputs](#-outputs)
- [Locals](#-locals)
- [Data sources](#-data-sources)
- [State](#-state)
- [Módulos](#-módulos)
- [Workspaces](#-workspaces)
- [Meta-arguments](#-meta-arguments)
- [Provisioners](#-provisioners)
- [Comandos essenciais](#-comandos-essenciais)
- [Exemplo completo](#-exemplo-completo)
- [Solução de problemas](#-solução-de-problemas)
- [Boas práticas](#-boas-práticas)
- [Referências](#-referências)

---

## 📖 O que é o Terraform

O [Terraform](https://www.terraform.io/) é uma ferramenta de **Infraestrutura como Código (IaC)** da HashiCorp. Com ele você descreve, em arquivos de texto, a infraestrutura desejada (servidores, redes, bancos de dados, DNS, buckets, clusters...) e o Terraform cria, altera e remove esses recursos de forma previsível.

Ele funciona com diversos provedores: AWS, Azure, Google Cloud, DigitalOcean, Kubernetes, GitHub, Cloudflare e centenas de outros.

**Vantagens:**

- ✅ **Declarativo**: você descreve *o que* quer, não *como* fazer
- ✅ **Multi-cloud**: mesma linguagem para vários provedores
- ✅ **Plano antes de aplicar**: veja exatamente o que vai mudar (`terraform plan`)
- ✅ **Versionável**: a infraestrutura vive no Git, com revisão e histórico
- ✅ **Reprodutível**: crie ambientes idênticos (dev, homologação, produção)
- ✅ **Grafo de dependências**: ordena e paraleliza a criação dos recursos automaticamente

> 💡 Existe também o [OpenTofu](https://opentofu.org/), um fork open source do Terraform mantido pela Linux Foundation. Os conceitos e a sintaxe deste manual valem para ambos (troque `terraform` por `tofu` nos comandos).

---

## ⚙️ Como funciona

```
Arquivos .tf  →  Terraform Core  →  Providers (APIs)  →  Infraestrutura real
                      │
                      └── State (terraform.tfstate): registro do que foi criado
```

| Conceito | Descrição |
|---|---|
| **Provider** | Plugin que conversa com a API de um serviço (AWS, Azure, etc.) |
| **Resource** | Um componente de infraestrutura gerenciado (VM, bucket, rede...) |
| **Data source** | Leitura de informações de algo que já existe |
| **Variable** | Parâmetro de entrada da configuração |
| **Output** | Valor exposto ao final da execução |
| **Module** | Conjunto reutilizável de recursos |
| **State** | Arquivo que mapeia o código para os recursos reais |
| **Backend** | Onde o state é armazenado (local, S3, Azure Blob, etc.) |
| **Plan** | Prévia das mudanças que serão feitas |
| **Workspace** | Instâncias separadas de state para a mesma configuração |

---

## 📋 Pré-requisitos

- Terminal (Linux, macOS ou Windows)
- Credenciais de acesso ao provedor que você vai usar (ex.: AWS, Azure, GCP)
- Editor de texto (recomendado: VS Code com a extensão **HashiCorp Terraform**)
- Git, para versionar o projeto

---

## 💾 Instalação

### Windows

```bash
choco install terraform
# ou
winget install Hashicorp.Terraform
```

### macOS

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### Linux (Debian/Ubuntu)

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

### Verificando a instalação

```bash
terraform version
```

### Autocompletar no terminal (opcional)

```bash
terraform -install-autocomplete
```

---

## 🏁 Primeiros passos

Vamos criar um recurso simples e local, que não precisa de conta em nenhuma nuvem:

```bash
mkdir meu-terraform && cd meu-terraform
```

`main.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

resource "local_file" "ola" {
  filename = "${path.module}/ola.txt"
  content  = "Olá, Terraform! 🚀"
}
```

Executando:

```bash
terraform init      # baixa os providers
terraform plan      # mostra o que será criado
terraform apply     # cria o recurso (confirme com "yes")
terraform destroy   # remove tudo
```

---

## 🔄 Fluxo de trabalho

```
Escrever  →  init  →  validate  →  plan  →  apply  →  (destroy)
```

| Etapa | Comando | O que acontece |
|---|---|---|
| 1. Escrever | — | Você cria ou altera os arquivos `.tf` |
| 2. Inicializar | `terraform init` | Baixa providers e módulos, configura o backend |
| 3. Formatar | `terraform fmt` | Padroniza o estilo do código |
| 4. Validar | `terraform validate` | Verifica a sintaxe e a consistência |
| 5. Planejar | `terraform plan` | Mostra o que será criado, alterado ou destruído |
| 6. Aplicar | `terraform apply` | Executa as mudanças |
| 7. Destruir | `terraform destroy` | Remove a infraestrutura gerenciada |

Símbolos do plano:

| Símbolo | Significado |
|---|---|
| `+` | Será criado |
| `~` | Será alterado no local |
| `-` | Será destruído |
| `-/+` | Será destruído e recriado |

---

## 🧾 Linguagem HCL

O Terraform usa a **HCL (HashiCorp Configuration Language)**, uma linguagem declarativa e fácil de ler.

### Estrutura básica

```hcl
<tipo_de_bloco> "<rótulo>" "<nome>" {
  argumento = valor
}
```

### Tipos de dados

```hcl
nome      = "servidor"                  # string
instancias = 3                          # number
ativo     = true                        # bool
zonas     = ["a", "b", "c"]             # list
tags      = { env = "dev", app = "web" } # map
```

### Comentários

```hcl
# comentário de uma linha
// também é um comentário de uma linha
/* comentário
   de várias linhas */
```

### Interpolação e expressões

```hcl
nome = "app-${var.ambiente}"
total = length(var.zonas)
upper = upper(var.nome)
```

### Condicional

```hcl
tipo = var.ambiente == "prod" ? "t3.large" : "t3.micro"
```

### Funções úteis

| Função | Exemplo | Resultado |
|---|---|---|
| `length()` | `length(["a","b"])` | `2` |
| `join()` | `join("-", ["a","b"])` | `"a-b"` |
| `lower()` / `upper()` | `upper("abc")` | `"ABC"` |
| `format()` | `format("app-%s", "x")` | `"app-x"` |
| `merge()` | `merge(map1, map2)` | mapa combinado |
| `lookup()` | `lookup(var.mapa, "chave", "padrao")` | valor ou padrão |
| `file()` | `file("script.sh")` | conteúdo do arquivo |
| `element()` | `element(var.lista, 0)` | primeiro item |
| `toset()` / `tolist()` | `toset(["a","a"])` | `["a"]` |

> 💡 Teste expressões no console interativo: `terraform console`

---

## 🔌 Providers

Providers são plugins que traduzem o código em chamadas de API.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "sa-east-1"
}
```

Operadores de versão:

| Operador | Significado |
|---|---|
| `= 5.0.0` | Exatamente essa versão |
| `>= 5.0` | Essa versão ou superior |
| `~> 5.0` | Qualquer `5.x` (não passa para a `6.0`) |
| `>= 5.0, < 6.0` | Intervalo |

### Múltiplas configurações do mesmo provider (alias)

```hcl
provider "aws" {
  region = "sa-east-1"
}

provider "aws" {
  alias  = "virginia"
  region = "us-east-1"
}

resource "aws_s3_bucket" "replica" {
  provider = aws.virginia
  bucket   = "meu-bucket-replica"
}
```

> ⚠️ **Nunca** coloque credenciais no código. Use variáveis de ambiente (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`), perfis, SSO ou papéis (roles) do provedor.

---

## 🧱 Resources

Resources são os recursos de infraestrutura que o Terraform gerencia.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t3.micro"

  tags = {
    Name = "servidor-web"
  }
}
```

- `aws_instance` é o **tipo** (definido pelo provider)
- `web` é o **nome local** (usado para referenciar no código)
- Referência a um atributo: `aws_instance.web.id`

### Dependências

O Terraform deduz as dependências pelas referências:

```hcl
resource "aws_eip" "ip" {
  instance = aws_instance.web.id   # dependência implícita
}
```

Dependência explícita, quando não há referência direta:

```hcl
resource "aws_instance" "app" {
  # ...
  depends_on = [aws_db_instance.banco]
}
```

---

## 🔤 Variáveis

### Declarando (`variables.tf`)

```hcl
variable "ambiente" {
  description = "Nome do ambiente"
  type        = string
  default     = "dev"
}

variable "instancias" {
  description = "Quantidade de instâncias"
  type        = number
  default     = 1

  validation {
    condition     = var.instancias > 0 && var.instancias <= 10
    error_message = "Use um valor entre 1 e 10."
  }
}

variable "senha_db" {
  description = "Senha do banco"
  type        = string
  sensitive   = true
}
```

### Usando

```hcl
resource "aws_instance" "web" {
  count         = var.instancias
  instance_type = "t3.micro"
  # ...
}
```

### Passando valores

```bash
# 1. Pela linha de comando
terraform apply -var="ambiente=prod"

# 2. Por arquivo
terraform apply -var-file="prod.tfvars"

# 3. Por variável de ambiente (prefixo TF_VAR_)
export TF_VAR_ambiente=prod
```

`prod.tfvars`:

```hcl
ambiente   = "prod"
instancias = 3
```

Precedência (da menor para a maior): valor `default` → variáveis de ambiente → `terraform.tfvars` → `*.auto.tfvars` → `-var` e `-var-file`.

---

## 📤 Outputs

Expõem informações ao final da execução ou para outros módulos.

```hcl
output "ip_publico" {
  description = "IP público do servidor"
  value       = aws_instance.web.public_ip
}

output "senha" {
  value     = var.senha_db
  sensitive = true
}
```

```bash
terraform output
terraform output ip_publico
terraform output -json
```

---

## 📍 Locals

Valores calculados reutilizáveis dentro do módulo:

```hcl
locals {
  prefixo = "${var.projeto}-${var.ambiente}"

  tags_comuns = {
    Projeto  = var.projeto
    Ambiente = var.ambiente
    Gerido   = "terraform"
  }
}

resource "aws_s3_bucket" "dados" {
  bucket = "${local.prefixo}-dados"
  tags   = local.tags_comuns
}
```

---

## 🔍 Data sources

Leem informações de recursos que já existem (ou de dados do provedor), sem gerenciá-los:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

---

## 🗃️ State

O **state** (`terraform.tfstate`) é o "mapa" entre o seu código e os recursos reais. Sem ele, o Terraform não sabe o que já criou.

> ⚠️ O state pode conter **dados sensíveis**. Nunca o versione no Git.

### Backend remoto (recomendado para times)

Exemplo com AWS S3:

```hcl
terraform {
  backend "s3" {
    bucket         = "meu-terraform-state"
    key            = "projeto/terraform.tfstate"
    region         = "sa-east-1"
    dynamodb_table = "terraform-locks"   # bloqueio (lock) do state
    encrypt        = true
  }
}
```

Benefícios do backend remoto: trabalho em equipe, **bloqueio** contra execuções simultâneas, criptografia e backup.

### Comandos de state

```bash
terraform state list                       # lista os recursos
terraform state show aws_instance.web      # detalhes de um recurso
terraform state mv <origem> <destino>      # renomeia/move no state
terraform state rm <recurso>               # remove do state (sem destruir)
terraform import aws_instance.web i-0abc   # importa um recurso existente
terraform refresh                          # atualiza o state com a realidade
```

### Bloco `moved` e `import`

```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.servidor_web
}

import {
  to = aws_instance.web
  id = "i-0123456789abcdef0"
}
```

---

## 📦 Módulos

Módulos agrupam recursos em partes reutilizáveis.

Estrutura recomendada:

```
projeto/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── modules/
    └── rede/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

Usando um módulo local:

```hcl
module "rede" {
  source = "./modules/rede"

  cidr     = "10.0.0.0/16"
  ambiente = var.ambiente
}

resource "aws_instance" "web" {
  subnet_id = module.rede.subnet_id
  # ...
}
```

Usando um módulo do [Terraform Registry](https://registry.terraform.io/):

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "minha-vpc"
  cidr = "10.0.0.0/16"
}
```

> 💡 Após adicionar ou alterar módulos, rode `terraform init` novamente.

---

## 🗂️ Workspaces

Permitem manter **states separados** para a mesma configuração (ex.: dev e prod).

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select dev
terraform workspace show
terraform workspace delete dev
```

Uso no código:

```hcl
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
}
```

> 💡 Para ambientes muito diferentes, muitos times preferem **pastas/states separados** por ambiente em vez de workspaces.

---

## 🎛️ Meta-arguments

Funcionam em qualquer resource ou module.

### `count`

```hcl
resource "aws_instance" "web" {
  count         = 3
  instance_type = "t3.micro"
  tags          = { Name = "web-${count.index}" }
}
```

### `for_each`

```hcl
resource "aws_iam_user" "usuarios" {
  for_each = toset(["ana", "joao", "maria"])
  name     = each.key
}
```

### `depends_on`

```hcl
depends_on = [aws_db_instance.banco]
```

### `lifecycle`

```hcl
resource "aws_instance" "web" {
  # ...
  lifecycle {
    create_before_destroy = true   # cria o novo antes de destruir o antigo
    prevent_destroy       = true   # impede a destruição acidental
    ignore_changes        = [tags] # ignora mudanças externas em atributos
  }
}
```

### `dynamic` blocks

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = [80, 443]
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

---

## 🛠️ Provisioners

Executam scripts durante a criação ou destruição de um recurso. A própria HashiCorp recomenda usá-los **como último recurso**; prefira `user_data`, imagens prontas ou ferramentas como o Ansible.

```hcl
resource "aws_instance" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]
  }
}
```

---

## 🧰 Comandos essenciais

| Comando | O que faz |
|---|---|
| `terraform init` | Inicializa o diretório (providers, módulos, backend) |
| `terraform init -upgrade` | Atualiza providers e módulos |
| `terraform fmt` | Formata o código |
| `terraform fmt -recursive` | Formata incluindo subpastas |
| `terraform validate` | Valida a configuração |
| `terraform plan` | Mostra o plano de execução |
| `terraform plan -out=plano.tfplan` | Salva o plano em arquivo |
| `terraform apply` | Aplica as mudanças |
| `terraform apply plano.tfplan` | Aplica um plano salvo |
| `terraform apply -auto-approve` | Aplica sem pedir confirmação |
| `terraform apply -target=<recurso>` | Aplica somente um recurso |
| `terraform destroy` | Destrói a infraestrutura |
| `terraform output` | Exibe os outputs |
| `terraform show` | Exibe o state ou um plano |
| `terraform graph` | Gera o grafo de dependências (DOT) |
| `terraform console` | Console interativo de expressões |
| `terraform state ...` | Gerencia o state |
| `terraform import` | Importa um recurso existente |
| `terraform taint` / `apply -replace=<recurso>` | Força a recriação de um recurso |
| `terraform workspace ...` | Gerencia workspaces |
| `terraform providers` | Lista os providers usados |
| `terraform version` | Mostra a versão |

> 💡 Use `-auto-approve` com cuidado, principalmente em produção.

---

## 🧪 Exemplo completo

Um exemplo na AWS com rede básica, security group e uma instância com Nginx.

`versions.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

`variables.tf`:

```hcl
variable "regiao" {
  type    = string
  default = "sa-east-1"
}

variable "ambiente" {
  type    = string
  default = "dev"
}

variable "tipo_instancia" {
  type    = string
  default = "t3.micro"
}
```

`main.tf`:

```hcl
provider "aws" {
  region = var.regiao
}

locals {
  tags = {
    Projeto  = "demo"
    Ambiente = var.ambiente
    Gerido   = "terraform"
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_security_group" "web" {
  name        = "web-${var.ambiente}"
  description = "Permite HTTP"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = local.tags
}

resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.tipo_instancia
  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl enable --now nginx
  EOF

  tags = merge(local.tags, { Name = "web-${var.ambiente}" })
}
```

`outputs.tf`:

```hcl
output "ip_publico" {
  description = "IP público da instância"
  value       = aws_instance.web.public_ip
}
```

Execução:

```bash
export AWS_PROFILE=meu-perfil

terraform init
terraform fmt
terraform validate
terraform plan -out=plano.tfplan
terraform apply plano.tfplan

terraform output ip_publico

# ao terminar, para evitar custos:
terraform destroy
```

> 💰 Recursos em nuvem podem gerar custos. Sempre rode `terraform destroy` em ambientes de teste.

`.gitignore` recomendado:

```gitignore
# Terraform
.terraform/
*.tfstate
*.tfstate.*
crash.log
*.tfplan
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Variáveis com dados sensíveis
*.tfvars
*.tfvars.json
```

> 📌 Versione o arquivo `.terraform.lock.hcl`: ele garante as mesmas versões de providers para todo o time.

---

## 🩺 Solução de problemas

| Problema | Possível solução |
|---|---|
| `Error: No valid credential sources found` | Configure as credenciais do provedor (variáveis de ambiente, perfil ou SSO) |
| `Error acquiring the state lock` | Outra execução está em andamento; aguarde ou use `terraform force-unlock <ID>` com cuidado |
| `Provider produced inconsistent result` | Rode `terraform init -upgrade` e tente novamente; verifique a versão do provider |
| `Error: Unsupported argument` | Confira a documentação do provider e a versão usada |
| `Cycle: ...` (dependência cíclica) | Reveja as referências entre recursos para eliminar o ciclo |
| Recurso já existe | Use `terraform import` para trazê-lo ao state |
| Diferença contínua no plano (*drift*) | Alguém alterou o recurso fora do Terraform; use `terraform plan` para ver e `ignore_changes` se for esperado |
| `Backend configuration changed` | Rode `terraform init -reconfigure` (ou `-migrate-state` para migrar o state) |
| Versão incompatível | Ajuste `required_version` ou atualize o Terraform |
| State corrompido ou perdido | Restaure um backup do backend remoto (habilite o versionamento do bucket) |

Para depurar:

```bash
TF_LOG=DEBUG terraform plan
```

No Windows (PowerShell):

```powershell
$env:TF_LOG="DEBUG"; terraform plan
```

---

## ✅ Boas práticas

- 🔒 **Nunca versione** `terraform.tfstate`, `*.tfvars` com segredos ou credenciais
- ☁️ Use **backend remoto** com bloqueio (lock), criptografia e versionamento
- 📌 **Fixe as versões** do Terraform e dos providers
- 🧹 Rode `terraform fmt` e `terraform validate` antes de cada commit (ou em um hook de pré-commit)
- 🔍 Sempre revise o `terraform plan` antes do `apply`; em CI/CD, salve o plano e aplique-o exatamente
- 📦 Crie **módulos** para padrões repetidos, com variáveis e outputs bem documentados
- 🏷️ Padronize **nomes e tags** (projeto, ambiente, responsável, centro de custo)
- 🌍 Separe **ambientes** (dev, homologação, produção) em states distintos
- 🔑 Marque variáveis e outputs sensíveis com `sensitive = true` e use um gerenciador de segredos
- 🛡️ Aplique o princípio do **menor privilégio** nas credenciais usadas pelo Terraform
- 🧪 Use ferramentas como `tflint`, `checkov` ou `trivy` para análise de qualidade e segurança
- 📝 Documente o projeto (por exemplo, com o `terraform-docs`)
- 🗑️ Destrua ambientes de teste para evitar custos desnecessários

---

## 📚 Referências

- [Documentação oficial do Terraform](https://developer.hashicorp.com/terraform/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [Tutoriais da HashiCorp](https://developer.hashicorp.com/terraform/tutorials)
- [OpenTofu](https://opentofu.org/)
- [Referência da linguagem HCL](https://developer.hashicorp.com/terraform/language)

---

## Contatos

[![Github Badge](https://img.shields.io/badge/-Github-000?style=flat-square&logo=Github&logoColor=white&link=https://github.com/cleibp)](https://github.com/cleibp)
[![Linkedin Badge](https://img.shields.io/badge/-LinkedIn-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/cleitonpaiva/)](https://www.linkedin.com/in/cleitonpaiva/)
[![Whatsapp Badge](https://img.shields.io/badge/-Whatsapp-4CA143?style=flat-square&labelColor=4CA143&logo=whatsapp&logoColor=white&link=https://api.whatsapp.com/send?phone=5516988368457&text=Ola!)](https://api.whatsapp.com/send?phone=5516988368457&text=Ola!)
[![Gmail Badge](https://img.shields.io/badge/-Gmail-c14438?style=flat-square&logo=Gmail&logoColor=white&link=mailto:cleibp@gmail.com)](mailto:cleibp@gmail.com)

[Pague-me um ☕](https://www.buymeacoffee.com/cleibp)

Feito com muito ❤️☕👨🏻‍💻 por Cleiton Paiva
