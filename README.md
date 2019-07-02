
# TERRAFORM
# Instalando terraform
  1-. Descargar terraform para linux:
  
    * wget https://releases.hashicorp.com/terraform/0.xx.x/terraform_0.xx.x_linux_amd64.zip
    
  2-. Instalar unzip
  
    * sudo apt-get install unzip
    
  3-. unzip y agregar path
    
    * unzip terraform_0.11.1_linux_amd64.zip 
      sudo mv terraform /usr/local/bin/
      
 4-. Verificar instalacion
    
    *   terraform -version

# Despliegue de Imagenes
  1-. Obtener public key
    
      * ssh-keygen -y -f archivo.pem
      
  2-. Crear archivo con extensión tf y agregar el sig. texto
  
    * nano terraform.tf
    
    ##
      #variable "access_key" {}
      #variable "secret_key" {}


      provider "aws" {
        access_key = "AKIAYQJDUM5BM4QL3X6S"
        secret_key = "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"
        region     = "us-east-2"
      }

      #Lo comente para que no vuelva a crear el mismo recurso
      #resource "aws_key_pair" "seminario" {
      #  key_name   = "seminario"
      # public_key = "<PUBLIC-KEY> root@docker-seminario1"
      #}

      resource "aws_security_group" "<NOMBRE-RECURSO-SEGURIDAD>" {
      name        = "rancher_fw_rules3"
      description = "Rancher firewall rules"
      #  vpc_id      = "${aws_vpc.main.id}"
      #  vpc_id     = "${module.vpc.vpc_id}"

    ingress {
          from_port   = 22
          to_port     = 22
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
     }
    ingress {
          from_port   = 80
          to_port     = 80
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
     }
     ingress {
          from_port   = 443
          to_port     = 443
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
      }
          ingress {
          from_port   = 2376
          to_port     = 2376
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
      }
      ingress {
          from_port   = 2379
          to_port     = 2380
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
     }
        ingress {
        from_port   = 3306
        to_port     = 3306
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
     }
      ingress {
        from_port   = 6379
        to_port     = 6379
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
      ingress {
        from_port   = 6443
        to_port     = 6443
        protocol    = "udp"
        cidr_blocks = ["0.0.0.0/0"]
    }
      ingress {
        from_port   = 8472
        to_port     = 8472
        protocol    = "udp"
        cidr_blocks = ["0.0.0.0/0"]
     }
     ingress {
        from_port   = 9099
        to_port     = 9099
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
      ingress {
        from_port   = 10250
        to_port     = 10250
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
        ingress {
        from_port   = 10254
        to_port     = 10254
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
    ingress {
        from_port   = 30000
        to_port     = 32767
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
    ingress {
        from_port   = 30000
        to_port     = 32767
        protocol    = "udp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    egress {
        from_port       = 0
        to_port         = 65535
        protocol        = "tcp"
        cidr_blocks     = ["0.0.0.0/0"]
    #    prefix_list_ids = ["pl-12c4e678"]
      }
      egress {
        from_port       = 0
        to_port         = 65535
        protocol        = "udp"
        cidr_blocks     = ["0.0.0.0/0"]
    #    prefix_list_ids = ["pl-12c4e678"]
      }


      tags {
        Name = "rancher_fw_rules"
      }

    }



    resource "aws_instance" "ubuntu-f2" {
    #east-2
      ami           = "ami-0c55b159cbfafe1f0" 
      instance_type = "t2.micro"
      key_name = "<NOMBRE-DEL-ARCHIV.PEM>"
      root_block_device = {
        volume_type           = "gp2"
        volume_size           = 50
        delete_on_termination = true
      }

      provisioner "remote-exec" {
        inline = [
          "export PATH=$PATH:/usr/bin:/bin/sbin",
          "sudo echo 'Waiting for the network'",
          "sudo apt-get update",
          "sudo echo 'Instalacion de Docker'",
          "sudo apt-get -y install docker.io",
          "sudo echo 'Clonacion de images de DockerHub.com'",
          "sudo docker pull tanyarecinos/semi1bd:last ",
          "sudo docker pull tanyarecinos/server:last ",
          "sudo docker pull tanyarecinos/api:last ",
          "sudo echo 'cambio de nombres'",
          "sudo docker tag tanyarecinos/semi1bd:last semi1bd:latest",
          "sudo docker tag tanyarecinos/paginasirve:last paginasirve:latest",
          "sudo docker tag tanyarecinos/api:last api:latest",
          "sudo echo 'Eliminacion de tags innecesarios'",
          "sudo docker rmi tanyarecinos/semi1bd:last ",
          "sudo docker rmi tanyarecinos/paginasirve:last ",
          "sudo docker rmi tanyarecinos/api:last ",
          "sudo echo ''",
          "sudo docker run -it --name api-node api",
          "exit",
          "sudo docker run -it --name server-node -p 80:80 server",
          "exit",
          #"sudo docker run -it --name database database",
          #"exit",
        ]

        connection {
          type     = "ssh"
          private_key = "${file("id_rsa")}"
          user     = "ubuntu"
          timeout  = "5m"
        }
      }

      depends_on = ["aws_security_group.<NOMBRE-RECURSO-SEGURIDAD>"]

      vpc_security_group_ids = ["${aws_security_group.<NOMBRE-RECURSO-SEGURIDAD>.id}"]

      }

      output "ubuntu_ip" {
      #  value = "${aws_eip.ip2.public_ip}"
        value = "${aws_instance.ubuntu-f2.public_ip}"
      }
      #
      
  4-.Copiamos el archivo .pem y lo renombramos como id_rsa
  
  5-. Iniciamos el ambito de terraform en la carpeta donde creamos nuestro archivo .tf

    * terraform init

  6-. Aplicamos el plan

    * terraform plan

  7-. Lo ejecutamos para que cree la instancia

    * terraform apply

  
