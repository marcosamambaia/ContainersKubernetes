# Usa a imagem oficial do Nginx como base
FROM nginx:latest

# Remove os arquivos padrão do Nginx
RUN rm -rf /usr/share/nginx/html/*

# Copia os HTMLs personalizados da pasta local para dentro do container
COPY html/ /usr/share/nginx/html/

# Expõe a porta padrão do Nginx
EXPOSE 80