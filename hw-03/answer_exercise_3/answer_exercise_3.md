# hw-03-exercise-03

[Horizontal Pod Autoscaler] Crea un objeto de kubernetes HPA, que escale a partir de las métricas CPU o memoria (a vuestra elección). Establece el umbral al 50% de CPU/memoria utilizada, cuando pase el umbral, automáticamente se deberá escalar al doble de replicas.

Podéis realizar una prueba de estrés realizando un número de peticiones masivas mediante la siguiente instrucción:
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never --/bin/sh -c "while sleep 0.01; do wget -q -O- http://&lt;svc_name&gt; done"

## Answer

Tenemos un deployment(nginx-deploy.yaml) y un servicio(nginx-svc.yaml). 

![image](./images/screenshot_1.png)

Desplegamos los pods y el servicio, y con el server corriendo ejecutamos:
~~~
kubectl autoscale deployment nginx-deploy --cpu-percent=50 --min=3 --max=10
~~~
- Se crea un Horizontal Pod Autoscaler. En el comando especificamos que se mantendrán entre 1 y 10 réplicas, escalando y desescalando para mantener un uso medio del 50% de CPU entre todas.

    ![image](./images/screenshot_2.png)

- Comprobamos el status del autoscaler (_kubectl get hpa_). El consumo de CPU debería ser 0 porque no estamos realizando peticiones. Check why: UNKNOWN TARGET.

Incrementamos la carga para ver cómo trabaja el autoscaler:
~~~
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never  -- sh -c "while sleep 0.01; do wget -q -O- http://nginx-svc; done"
~~~
- Dando un margen de tiempo, podemos ver el incremento en uso de CPU:

    ![image](./images/screenshot_3.png)

- Vemos que cuando el consumo pasa del umbral, se han escalado las réplicas. En este caso, vemos que al 177% de CPU pasamos de 3 a 5 réplicas (no estamos controlando la cantidad de carga; lleva un tiempo estabilizar el número de réplicas), pero al cabo de poco ya lo tenemos escalado al máximo de 10. 

- Acabamos con la carga que hemos forzado con un simple ctrl+C y, dando un margen de tiempo, olvemos a consultar el estado del deployment. 

    ![image](./images/screenshot_4.png)

**TODO**: CHECK ERROR en metrics server?
- metrics server enabled (minikube addons enable metrics-server)
- kubectl -n kube-system rollout status deployment metrics-server
- kubectl top node OK
- kubectl top pod
    - W0127 09:46:27.013695  138078 top_pod.go:265] Metrics not available for pod default/nginx-deploy-7fc894f9f8-58wq2, age: 3m55.01368723s
    - error: Metrics not available for pod default/nginx-deploy-7fc894f9f8-58wq2, age: 3m55.01368723s
