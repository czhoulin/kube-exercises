# hw-02-exercise-05

Diseña una estrategia de despliegue que se base en ”Blue Green”. Podéis utilizar la imagen del ejercicio 1.

Adjunta todos los ficheros para crear esta prueba de concepto.

## Answer

Estrategia:
- La versión v1.0 está en producción.
- Levantamos la versión v2.0.
- Esperamos a que v2.0 esté lista.
- Redirigimos el tráfico de v1.0 a v2.0
- Bajamos la versión v1.0

Pasos a seguir:
1. Creamos el objeto deployment Blue
    ~~~
    kubectl apply -f deployment-blue.yaml
    ~~~
2. Creamos un servicio Blue
    ~~~
    kubectl apply -f service-blue.yaml
    ~~~
    - _kubectl get service_: vemos que el service está expuesto en el 80 y en nuestra máquina minikube se le ha asignado el puerto 30338.
    - _kubectl get endpoints blue-service_: vemos los pods a los que se realizan las peticiones.
    - _export MINIKUBE_IP=$(minikube ip)_: extraemos a variable de entorno la IP de minikube
    - _echo $MINIKUBE_IP_: comprobamos el valor de la variable
    - _kubectl describe service blue-service_ / _kubectl get service blue-service -o json_: vemos detalles del servicio
    - _export PORT_BLUE=$(kubectl get service blue-service -o 'jsonpath={.spec.ports[0].nodePort}')_: extraemos el puerto a variable de entorno
    - echo $PORT_BLUE: comprobamos el valor de la variable
    - _while true; do curl ${MINIKUBE_IP}:${PORT_BLUE}/version; echo ‘\n’; sleep .5; done_: vemos que van cargando las versiones 1.18
    - _kubectl get service_: en otra ventana vemos el service corriendo
3. Creamos el objeto deployment Green
    ~~~
    kubectl apply -f deployment-green.yaml
    ~~~
4. Creamos un servicio Green
    ~~~
    kubectl apply -f service-green.yaml
    ~~~
    - _export PORT_GREEN=$(kubectl get service green-service -o 'jsonpath={.spec.ports[0].nodePort}')_: extraemos el puerto a variable de entorno
    - _while true; do curl ${MINIKUBE_IP}:${PORT_GREEN}/version; echo ‘\n’; sleep .5; done_: vemos que van cargando las versiones 1.19.6
6. Abrimos otra ventana para ver los cambios:
    ~~~
    kubectl get pods -w
    ~~~
7. Hacemos que ambos servicios apunten a la versión nueva:
    ~~~
    kubectl patch service blue-service -p '{"spec":{"selector":{"version": "1.19.6"}}}'
    ~~~

    No hace falta crear dos servicios