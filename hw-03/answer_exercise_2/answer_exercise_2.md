# hw-03-exercise-02

[StatefulSet] Crear un StatefulSet con 3 instancias de MongoDB. Se pide:
- Habilitar el clúster de MongoDB
- Realizar una operación en una de las instancias a nivel de configuración y verificar que el cambio se ha aplicado al resto de instancias
- Diferencias que existiría si el montaje se hubiera realizado con el objeto de ReplicaSet

## Answer

Creamos las réplicas
~~~
kubectl get pods -w (en otra ventana) → vemos que se crearán de forma ordenada
kubebectl create -f mongodb-statefulset.yaml
~~~
Creamos el servicio
~~~
kubectl create -f mongodb-svc.yaml
kubect get svc
kubectl get endpoint mongodb-svc
~~~
Ahora tenemos 3 réplicas que no están conectadas entre sí. Queremos un cluster que tenga HA.

- Entramos dentro de un pod 
    ~~~
    kubect exec -it mongo-0 sh
    ~~~

- Accedemos al cliente de mongo 
    ~~~
    mongo
    ~~~

- Estamos conectados dentro de la primera réplica. Vemos que no está configurada la replicaSet con la mongoDB. Creamos el cluster con nuestras 3 réplicas:
    ~~~
    rs.status() 
    rs.initiate…. → iniciar un cluster de mongoDB
    ~~~

- Vemos el cluster con las reps inicializadas
    ~~~
    rs.conf() → v
    ~~~

    ![image](./images/screenshot_1.png)

Si operamos en una de las réplicas veremos que los cambios se aplican en las otras.

Levantamos express y añadimos una base de datos:
~~~
kubectl get pods -w
kubectl create -f mongoexpress-deploy.yaml
kubectl port-forward podName 8088:8081
~~~
Creamos una colección + un documento:
![image](./images/screenshot_2.png)

Vemos este cambio en cualquiera de las instancias
~~~
kubectl exec -it mongo-2 sh → mongo
rs.secondaryOk()
show dbs
use MYDATABASENAME
show tables
db.MYCOLLECTION.find() → vemos mi cambio
exit
~~~

Igual que en el momento de creación, si desescalamos a dos el número de réplicas veremos que se hace de forma ordenada, comenzando por la última intancia (mongo-2)
~~~
kubectl scale --replicas=2 statefulset mongo
~~~
Con ReplicaSet hubiese sido aleatorio.
