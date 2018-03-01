# Jenkins
@Prerequisitos
- VM CentOS 7

# Leccion 1
## Instalacion de Jenkins en CentOS 7
<<<<<<< HEAD

```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum update
yum install jenkins java-1.8.0-openjdk -y
```

## Habilitamos Jenkins

```
systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins
```
## Comprobamos que Jenkins está escuchando en el 8080:

```
ss -putan | grep 8080
```
@@@
Si desde fuera no accedemos al 8080 por estar filtrado por un FW, podemos instalar NGINX delante y hacer que sirva nuestro nuevo Jenkins.
@@@

## NGINX
### Instalacion de Nginx

```
yum install nginx -y
```

### Configuracion de NGINX

```
vim /etc/nginx/nginx.conf
```
- En:
```
location /{
}
```
- Añadimos el proxy_pass:
```
location /{
    proxy_pass http://127.0.0.1:8080;
}
```

### Habilitar y arrancar nginx:

```
systemctl enable nginx
systemctl start nginx
systemctl status nginx
```

### SELinux: de enforcing a permissive

```
getenforce
setnforce 0
getenforce
```

### Testeo con elinks

```
yum install elinks
elinks http://localhost:8080
elinks http://localhost
```

### Permanencia del redirect al 8080

```
yum install setroubleshoot-server selinux-policy-devel -y
sepolicy network -t http_port_t
semanage port -m -t http_port_t -p tcp 8080
sepolicy network -t http_port_t
setenforce 1
getenforce
systemctl restart nginx jenkins
```

## Configurar jenkins

- Accedemos via http:

```
arkalira1.mylabserver.com
```

- Unlock Jenkins

```
cat /var/lib/jenkins/secrets/initialAdminPassword
4fc9fce31fdf4c8695a50ac9e0308e28
```

- Instalacion: "Install with suggested plugins".
- Seguir el resto de pasos.
- Ir a Manage Jenkins y veremos un error del proxy reverso.

## Solucionar problema con proxy reverso detras de @NGINX:

- https://wiki.jenkins.io/display/JENKINS/Running+Jenkins+behind+Nginx

```
vim /etc/nginx/nginx.conf
```

- En el location / de Jenkins añadimos debajo del proxy_pass:

```
proxy_redirect     default;
proxy_http_version 1.1;

proxy_set_header   Host             $host;
proxy_set_header   X-Real-IP        $remote_addr;
proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
proxy_max_temp_file_size 0;

#this is the maximum upload size
client_max_body_size       10m;
client_body_buffer_size    128k;

proxy_connect_timeout      90;
proxy_send_timeout         90;
proxy_read_timeout         90;

proxy_buffer_size          4k;
proxy_buffers              4 32k;
proxy_busy_buffers_size    64k;
proxy_temp_file_write_size 64k;

```

## Ejemplo de location completo para Jenkins usando NGINX:

```
location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect     default;
        proxy_http_version 1.1;

        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_max_temp_file_size 0;

        #this is the maximum upload size
        client_max_body_size       10m;
        client_body_buffer_size    128k;

        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;

        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
}
```

# Leccion 2
## Preparar un entorno y cuentas para lanzar builds

```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum update
```
