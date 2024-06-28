# Terraformfuncionesavanzada

Sección 1: Creación de Recursos en AWS utilizando Funciones Avanzadas
En esta práctica, vamos a crear una serie de recursos en AWS utilizando Terraform, y aplicaremos varias funciones avanzadas de Terraform para manipular datos y configurar nuestros recursos. 
 
Las funciones que utilizaremos son:
* Funciones numéricas (min)
* Funciones de cadena (join)
* Funciones de fecha y hora (formatdate)
* Funciones de red IP (cidrsubnet)
 
Paso 1: Configuración Inicial
 
1.1. Instalación de Terraform
Asegúrate de tener Terraform instalado. Puedes seguir las instrucciones oficiales en la documentación de Terraform.
 
 
Paso 2: Creación del Proyecto Terraform
2.1. Estructura del Proyecto
Crea un directorio para tu proyecto de Terraform:
 
mkdir terraform-aws-practice
cd terraform-aws-practice
 
Dentro de este directorio, crea un archivo main.tf donde definiremos nuestros recursos.
 
Paso 3: Definición de Variables y Backend
3.1. Definir Variables
Crea un archivo variables.tf para definir las variables que utilizaremos:
 
 provider "aws" {
  region = "us-east-1"  # Cambia esto a tu región preferida
}
variable "prefix" {
  description = "El prefijo para los nombres de los recursos"
  type        = string
  default     = "devops"
}
 
variable "vpc_cidr" {
  description = "El rango CIDR para la VPC"
  type        = string
  default     = "10.0.0.0/16"
}
 
Paso 4: Crear Recursos Utilizando Funciones Avanzadas
4.1. Función Numérica - min
Vamos a crear una VPC y subnets utilizando la función min para determinar el número mínimo de subnets a crear:
 
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags = {
    Name = "${var.prefix}-vpc"
  }
}
locals {
  subnet_count = min(2, 3, 4)
}
resource "aws_subnet" "subnets" {
    count = local.subnet_count
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = element(data.aws_availability_zones.available.names, count.index)
  tags = {
    Name = "${var.prefix}-subnet-${count.index}"
  }
}
 
4.2. Función de Cadena - join
Utilizaremos la función join para crear un nombre concatenado para nuestros recursos:
 
output "vpc_id" {
  value = aws_vpc.main.id
}
 
output "subnet_ids" {
  value = join(", ", aws_subnet.subnets.*.id)
}
 
4.3. Función de Fecha y Hora - formatdate
Vamos a utilizar formatdate para crear una etiqueta con la fecha y hora actual en un formato específico:
 
resource "aws_instance" "web" {
  ami           = "ami-01b799c439fd5516a"
  instance_type = "t2.micro"
 
  tags = {
    Name = "${var.prefix}-instance"
    CreatedAt = formatdate("YYYY-MM-DD hh:mm:ss", timestamp())
  }
}
 
4.4. Función de Red IP - cidrsubnet
Utilizaremos cidrsubnet para calcular subnets adicionales dentro de nuestra VPC:
 
resource "aws_subnet" "additional_subnets" {
  count = 2
  vpc_id = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, count.index + 3)
  availability_zone = element(data.aws_availability_zones.available.names, count.index + 3)
  tags = {
    Name = "${var.prefix}-additional-subnet-${count.index}"
  }
}
 
Paso 5: Implementación y Verificación
5.1. Inicializar el Proyecto
Inicializa tu proyecto de Terraform:
 
terraform init
 
5.2. Planificar la Infraestructura
Crea un plan para tu infraestructura:
 
terraform plan
 
5.3. Aplicar la Configuración
Aplica la configuración para crear los recursos en AWS:
 
terraform apply

