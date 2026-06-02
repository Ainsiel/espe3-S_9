# Resumen Ejecutivo

## 1. Objetivo de la Actividad

El objetivo de esta actividad es realizar el despliegue automatizado de un sistema de reservas en un servidor Amazon EC2, utilizando contenedores Docker y una pipeline de GitHub Actions.

La idea principal es que el proyecto pueda publicarse de forma ordenada, repetible y sencilla. Cada vez que se actualice la rama `main` del repositorio `espe3-S_9`, la pipeline se encarga de ejecutar las validaciones necesarias y actualizar la aplicacion en el servidor EC2.

Con esto se evita realizar el despliegue manualmente cada vez que exista un cambio en el sistema.

## 2. Arquitectura del Sistema

## 2.1. Vision general

El proyecto `espe3-S_9` corresponde a un sistema de reservas llamado **EventPass**, orientado a la reserva de entradas para eventos.

El sistema esta compuesto por una interfaz web donde los usuarios pueden ver eventos disponibles, registrarse, iniciar sesion, realizar reservas y consultar sus reservas. Tambien cuenta con una parte encargada de procesar las solicitudes, guardar la informacion y responder a las acciones realizadas desde la pagina web.

Para facilitar su ejecucion, el proyecto fue preparado con contenedores Docker. Esto permite que el sistema se ejecute de la misma manera en distintos ambientes, como el computador local o un servidor en la nube.

La arquitectura general se puede entender de la siguiente forma:

- El usuario ingresa al sistema desde un navegador web.
- El servidor EC2 recibe la visita y muestra la pagina principal del sistema.
- La pagina se comunica internamente con el servicio encargado de manejar las reservas.
- La informacion del sistema se guarda en una base de datos SQLite dentro de un volumen Docker.
- Docker Compose coordina la ejecucion del frontend, backend y almacenamiento necesario.

En cuanto a infraestructura, el sistema queda alojado en una instancia EC2 de Amazon Web Services. Esta instancia funciona como servidor principal, donde se ejecutan los contenedores que mantienen disponible la aplicacion.

## 3. Ejecucion del Workflow: Caso Practico

La pipeline de CI/CD permite automatizar el proceso de despliegue del sistema.

En este caso, el flujo funciona de la siguiente manera:

1. Se realiza un cambio en el proyecto y se sube a la rama `main` del repositorio `espe3-S_9`.
2. GitHub Actions detecta automaticamente que hubo una actualizacion.
3. La pipeline ejecuta las pruebas del backend para comprobar que el sistema no tenga errores importantes.
4. Si las pruebas son exitosas, GitHub Actions se conecta al servidor EC2 mediante SSH.
5. Dentro del servidor, se actualiza el codigo del proyecto con la ultima version de la rama `main`.
6. Luego se reconstruyen y reinician los contenedores Docker usando Docker Compose.
7. Finalmente, la nueva version del sistema queda disponible en el servidor EC2.

Gracias a este flujo, no es necesario entrar manualmente al servidor para copiar archivos, reiniciar servicios o ejecutar comandos cada vez que se realice una actualizacion. El proceso queda centralizado en GitHub y se ejecuta de forma automatica.

## 4. Conclusion

Automatizar las pipelines de despliegue es importante porque permite que los cambios del sistema lleguen al servidor de manera mas rapida, controlada y confiable.

En un despliegue manual, es facil olvidar pasos, ejecutar comandos incorrectos o subir una version incompleta. En cambio, una pipeline automatizada sigue siempre el mismo proceso, ejecuta validaciones previas y reduce la posibilidad de errores humanos.

Ademas, esta forma de trabajo mejora la organizacion del proyecto, ya que cada cambio subido a la rama principal puede transformarse automaticamente en una nueva version desplegada. Esto es especialmente util cuando un sistema crece, cuando trabajan varias personas en el mismo repositorio o cuando se necesita mantener una aplicacion disponible en un servidor real.

En resumen, el uso de Docker, EC2 y GitHub Actions permite construir un proceso de despliegue moderno, repetible y mas seguro para el sistema EventPass.
