# montagem do ambiente

o build é feito via openshift no namespace openshift 


Comando para executar o build da imagem de base:

```bash
cat Dockerfile | oc new-build -D - --name handson
```

Comando para remover em caso de erro:

```bash
oc delete all -l "build=handson"
```

Adiciona todos como cluster-admin:
```bash
for i in {1..20}
do
   oc adm policy add-cluster-role-to-user cluster-admin user$i
done
```

Cria a workspace do projeto:
```bash
for i in {1..20}
do
   cat lab.yaml | sed -e "s/HANDSON/user$i/g" | oc create -f -
done
```


