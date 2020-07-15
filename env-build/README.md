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

# Lab java
jdk < 8:191

java -XshowSettings:vm -version

jdk 8:191 >

java -XX:MaxRAM=400m -Xmx300m -XX:MaxRAMFraction=1 -XshowSettings:vm -version

java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XshowSettings:vm -version

java -XX:MaxRAMFraction=2 -XshowSettings:vm -version

+----------------+-------------------+
| MaxRAMFraction | % of RAM for heap |
|----------------+-------------------|
|              1 |              100% |
|              2 |               50% |
|              3 |               33% |
|              4 |               25% |
+----------------+-------------------+


import java.util.ArrayList;
import java.util.List;

public class MemEat {
    public static void main(String[] args) {
        List l = new ArrayList<>();
        while (true) {
            byte b[] = new byte[1048576];
            l.add(b);
            Runtime rt = Runtime.getRuntime();
            System.out.println( "free memory: " + rt.freeMemory() );
        }
    }
}

