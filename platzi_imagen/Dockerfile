FROM ubuntu
# todos los docker file deben iniciar con un from
# que declara la imagen base
# desde la cual creamos nuestra imagen custom

# se le dice que en tiempo de build ejecute este comando en el container de la imagen
RUN touch /usr/src/hola-platzi

# a pesar de que el resultado final no posee el archivo
# docker igual genera el layer que posee el archivo
# pues capa no es mas que una diferencia con la anterior
RUN rm /usr/src/hola-platzi