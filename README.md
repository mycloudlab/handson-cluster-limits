
# 1.1 Definindo um pod com limites

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-seunome
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "220Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150Mi", "--vm-hang", "1"]
```

# 1.2 Definindo um pod com mais memoria do que o limite 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-seunome
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "150Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200Mi", "--vm-hang", "1"]
```


# 1.3 Definindo um pod com limite de CPU

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-seunome
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "300Mi"
        cpu: "1"
      requests:
        memory: "300Mi"
        cpu: "0.5"
    command: ["stress"]
    args: [ "--cpu", "2"]
```

após alguns minutos execute:

```bash
oc project seu-namespace
oc adm top pod
```
A saida deve ser algo como abaixo:

```bash
NAME                  CPU(cores)   MEMORY(bytes)   
memory-demo-seunome   1000m        2Mi      
```

# 2.1 definindo um limit range

Remova o limit range atual
```bash
oc delete limitrange limits-core-resource-limits
```

e crie o limit range abaixo:

```yaml
kind: LimitRange
apiVersion: v1
metadata:
 name: limits-core
spec:
 limits:
   - type: Container
     max:
       cpu: '500m'
       memory: 256Mi
     default:
       cpu: 250m
       memory: 256Mi
     defaultRequest:
       cpu: 200m
       memory: 100Mi
   - type: Pod
     max:
       cpu: '1'
       memory: 1Gi
```

Neste exemplo são permitidos a criação de pods que a soma dos recursos não ultrapasse 1 core e 1Gi de uso.

Os containers quando não especificados terão limite de 250 mili cores e 256Mi, e request de 200 milicores e 100Mi de memoria request.

Tentar definir limites fora desse intervalo ocasionará em erro: 

tente criar o pod abaixo sem aplicar limites:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
    - name: web
      image: nginx
```

Depois visualize o YAML do pod, verá que foi definido um limite embora não tenha sido informado:
```yaml
...
...
containers:
    - name: web
      image: nginx
      resources:
        limits:
          cpu: 250m
          memory: 256Mi
        requests:
          cpu: 200m
          memory: 100Mi
      volumeMounts:
... 
...
```

Remova o container.

O limit range não impede que seja definido um limite desde que não atinja o tamanho máximo:

Tente criar o pod abaixo é veja que recebe um erro pois o pod ultrapassou o limite maximo para um container que é de 500Mi.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
    - name: web
      image: nginx
      resources:
        limits:
          memory: "1Gi"
          cpu: "1"
        requests:
          memory: "300Mi"
          cpu: "0.5"
```

# 3.1 Resource Quota

Crie o resource quota abaixo:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "3"
    requests.memory: 1Gi
    limits.cpu: "5"
    limits.memory: 2Gi
    pods: "5"
```

No exemplo acima podemos criar até 5 pods e a soma dos 5 pods não pode ultrapassar os limites de memória.

Crie abaixo os pods de 1 a 6 e veja que no sexto vai dar erro pois será imposto o quota.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-1-6
spec:
  containers:
    - name: web
      image: nginx
```

# 4.1 Exercício ferramenta hypercc

A ferramenta permite saber como quanto de capacidade o cluster suporta para uma especificação de pod.

Na linha de comando execute a ferramenta hypercc.

```bash 
hypercc cluster-capacity --kubeconfig ~/.kube/config --podspec pod-example.yaml --verbose
```

