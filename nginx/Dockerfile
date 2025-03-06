FROM nginx:latest

# Instalar pacotes adicionais
RUN apt-get update && apt-get install -y iputils-ping curl && apt-get clean

# Copiar configuração personalizada
COPY nginx.conf /etc/nginx/nginx.conf

# Expor a porta 8080
EXPOSE 8080

