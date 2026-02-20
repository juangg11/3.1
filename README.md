# Práctica: Habilitación de HTTPS con Let's Encrypt, Docker y Docker Compose

## Conceptos Teóricos

### 1.1 HTTPS y Let's Encrypt

**HTTPS** (Hypertext Transfer Protocol Secure) es la versión segura del protocolo HTTP, que utiliza cifrado TLS/SSL para proteger la transmisión de datos. Los certificados utilizados en esta práctica son emitidos por **Let's Encrypt**, una autoridad de certificación gratuita y automatizada.

### 1.2 Protocolo ACME y HTTPS-PORTAL

Para la obtención del certificado se utiliza el **protocolo ACME**. El servicio **HTTPS-PORTAL** actúa como un proxy inverso basado en Nginx que automatiza la solicitud, instalación y renovación de los certificados de Let's Encrypt de forma transparente.

---

## Configuración del Entorno

### 2.1 Requisitos de la Instancia EC2

| Recurso | Especificación |
|---------|----------------|
| Tipo de instancia | t2.small |
| Almacenamiento | 20 GB |
| Sistema Operativo | Ubuntu Server |

### 2.2 Configuración de Red (Security Groups)

Se han habilitado los siguientes puertos en el firewall de AWS:

| Puerto | Protocolo | Servicio |
|--------|-----------|----------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |

### 2.3 Configuración DNS

Se ha registrado el dominio `walter2.servebeer.com` configurando registros de tipo **A** que apuntan a la dirección IP pública de la instancia EC2.

---

## Implementación Técnica

### 3.1 Instalación de Docker y Docker Compose

Se ha utilizado el siguiente script de Bash para la preparación del nodo:

```bash
#!/bin/bash
set -x

apt update
apt install -y ca-certificates curl gnupg lsb-release

mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

usermod -aG docker $USER

systemctl enable docker
systemctl start docker
```

### 3.2 Archivo de Configuración Docker Compose

El archivo `docker-compose.yml` integra los servicios de MySQL, PrestaShop y HTTPS-PORTAL:

```yaml
version: '3.4'

services:
  mysql:
    image: mysql:9.1
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend-network
    restart: always

  prestashop:
    image: prestashop/prestashop:8
    environment:
      - DB_SERVER=mysql
    volumes:
      - prestashop_data:/var/www/html
    networks:
      - backend-network
      - frontend-network
    restart: always
    depends_on:
      - mysql

  https-portal:
    image: steveltn/https-portal:1
    ports:
      - 80:80
      - 443:443
    restart: always
    environment:
      DOMAINS: "${DOMAIN} -> http://prestashop:80"
      STAGE: 'production'
    networks:
      - frontend-network

volumes:
  mysql_data:
  prestashop_data:

networks:
  backend-network:
  frontend-network:
```

### 3.3 Archivo de Variables de Entorno

Crear un archivo `.env` en el mismo directorio:

```env
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=prestashop
MYSQL_USER=ps_user
MYSQL_PASSWORD=ps_password
DOMAIN=walter2.servebeer.com
```

> **Nota de Seguridad:** No subas el archivo `.env` a Git. Asegúrate de añadirlo al `.gitignore`

---

## Puesta en Producción y Verificación

### 4.1 Despliegue de los Servicios

Se despliegan los servicios mediante el comando:

```bash
docker compose up -d
```

### 4.2 Verificación de Contenedores

Se verifica la correcta creación de los contenedores con:

```bash
docker compose ps
```

Salida esperada:

```
NAME                    IMAGE                       STATUS
prestashop-mysql-1      mysql:9.1                   Up
prestashop-prestashop-1 prestashop/prestashop:8     Up
prestashop-https-1      steveltn/https-portal:1     Up
```

### 4.3 Acceso al Sitio Web

Se accede a la URL `https://walter2.servebeer.com` para comprobar la asignación del certificado SSL.

---

## Estructura del Proyecto

```
.
├── docker-compose.yml
├── .env
├── .env.example
├── .gitignore
├── install_docker.sh
└── README.md
```
<img width="1030" height="519" alt="image" src="https://github.com/user-attachments/assets/4c381bde-56ec-475d-9fcb-472b8ed5cb97" />
<img width="1017" height="537" alt="image" src="https://github.com/user-attachments/assets/c1bdd523-9d85-4cb1-b88c-42b8713d6a99" />
<img width="1047" height="154" alt="image" src="https://github.com/user-attachments/assets/36b493a9-6cf4-4440-9d8d-9a6311a052fa" />
<img width="1043" height="253" alt="image" src="https://github.com/user-attachments/assets/5bb4c862-4807-4d6b-9f30-142d01b970ba" />
<img width="701" height="173" alt="image" src="https://github.com/user-attachments/assets/90268fca-103f-4c4f-9eb6-28d3d9041c37" />
<img width="897" height="507" alt="image" src="https://github.com/user-attachments/assets/c2232900-bee3-4f2e-bc95-35ede3649a27" />
<img width="893" height="493" alt="image" src="https://github.com/user-attachments/assets/16c2fe23-d328-45aa-8df6-842009ce8783" />






