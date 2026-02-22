##  CLI Commands
- kubectl describe pod my-pod - посмотреть описание объекта
- kubectl get <pods> / kubectl get <pod, node, rs, etc> - получить поды кластера
  - kubectl get pod -l app=my-app - получить все по лейблу
- kubectl create -f pod.yml - создать инстанс из описания ямла 
- kubectl apply -f pod.yml - создать/пересоздать инстанс из описания ямла
- kubectl delete -f pod.yaml - удалить под описанный в ямле 
- kubectl delete pod my-pod - удалить под по имени
- kubectl delete pod --all - понятно
- kubectl scale --replicas 3 replicaset my-replicaset - поскейлить RS
- kubectl set image replicaset my-replicaset nginx=quay.io/testing-farm/nginx:1.13 - обновить приложение. Я засетал другой образ и все это подхватится и перекатится
- kubectl explain <pod/pod.spec/etc> - дока за все филды ямла инстанса
- kubectl edit deploy - открыть на лету конфиг, и применить его
- kubectl rollout undo deployment my-deployment - откатиться
- kubectl patch deployment my-deployment --patch '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"cpu":"10"},"limits":{"cpu":"10"}}}]}}}}' - пропатчить че нам надо

## Верхнеуровневое устройство куба

![1.1 Kubernetes workflow.png](img/1.1%20Kubernetes%20workflow.png)
![1.2 Kubernetes workflow.png](img/1.2%20Kubernetes%20workflow.png)
![1.3 Kubernetes workflow.png](img/1.3%20Kubernetes%20workflow.png)

1. При помощи чего можно управлять кубом
    - UI - всякие веб-морды в т.ч. Lens
    - cli - кубКонтрол - в терминале
2. Передаем своим пожелания в api куба
3. Апи прихранивает конфигурацию в БД и передает в работу в "K8s Master" - это для упрощения, на самом деле тут нахрдится группа сервисов
4. "K8s Master" накатывает на ноды

## Первый запуск миниКуба

1. Поднимаю миникуб в докере
2. стартую
   ```bash
   minikube start
   ```
3. Чекаю
   ```bash
   minikube status  
   ```

## Pods
Под - минимальная единица в кубе. 

Внутри пода всегда есть как минимум 2 контейнера. Один конейнер будет приложением, а второй с именем 
POD_имя контейнера_хеши... Этот служебный контейнер создается при создании пода с приложением и несет 
внутри себя сетевой неймспес для контейнеров этого пода. 

Внутри пода может существовать несколько контейнеров - но это скорее исключение и этого стоит избегать.
Внутри такого пода контейнеры обращаются друг к другу через localhost
Когда это выглядит оправданным:
- Если из нужно запускать на одном хосте. Например приложение имеет свой кэш и с ним живет, т.е. кеш не распределенный. 
   Нас устраивает, что перезапуск приложения на поде будет рестиртить и кэш. Другие поды этого же приложения не видят не своего кэша
- Приложения масштабируются линейно. Вот у нас приложения с собственными кэшами. Скейлим еще одно приложение и 
   нам норм, что поднимется новая прила со своим кэшом
- Если компоненты имеют сильную связь. Например Прометеус стратует с какими-то настроейками. Если мы хотим что-то подтюнить 
   то Прометеус не умеет считывать изменения в конфигах. Тогда с этот под в кпрометеусу подсаживают контейнер с прилой 
   которая умеет пинать прометеус и подсовывать ему новые конфиги

#### Практика по теме
в файле описан создаваемый под
[pod.yaml](../practice/3.application-abstractions/1.pod/pod.yaml)

создаем под 
   ```bash
   kubectl create -f pod.yaml
   ```
И теперь в консоле, получая под мы видим
```bash
NAME     READY   STATUS    RESTARTS   AGE
my-pod   1/1     Running   0          6m11s
```
При попытке создать еще один под из этого же ямла, мы получим ошибку: т.к. в одном пространстве имен одна и та же сущность 
с одним и тем же именем существовать не может. Если мы ямле поменяем имя в метадате, то стартанет новый под

### kubectl describe
kubectl describe pod my-pod - позволяет полусить инфу по объекту

```bash
Name:             my-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sun, 22 Feb 2026 16:43:31 +0300
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  nginx:
    Container ID:   docker://1231cc03dfceee1e48f183b47fa8122b55f03be257e0bdcbf709a7823a54a695
    Image:          quay.io/testing-farm/nginx:1.12
    Image ID:       docker-pullable://quay.io/testing-farm/nginx@sha256:09e210fe1e7f54647344d278a8d0dee8a4f59f275b72280e8b5a7c18c560057f
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 22 Feb 2026 16:43:40 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-m6frd (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-m6frd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  15m   default-scheduler  Successfully assigned default/my-pod to minikube
  Normal  Pulling    15m   kubelet            Pulling image "quay.io/testing-farm/nginx:1.12"
  Normal  Pulled     15m   kubelet            Successfully pulled image "quay.io/testing-farm/nginx:1.12" in 8.837s (8.837s including waiting). Image size: 108382637 bytes.
  Normal  Created    15m   kubelet            Created container: nginx
  Normal  Started    15m   kubelet            Started container nginx
```

## ReplicaSet

replicaset - это набор реплик приложения. Внутри описания находится шаблон под, которые будут из него собираться 
РепликаСет создавая ноды будет проставлять на них лейблы, по которым он всегда сможет определить свои поды 
А у самого репликаСета есть свойство "селектор", которое точно такое же, как и леблы для под
Еще среди свойств сета есть свойство, которое говорит, сколько реплик нужно создать

RS следит за подами. Мы создали 2 и если сейчас руками один убить или добавить, rs все равно вернет к заданному
в конфиге состоянию

#### Практика по теме
в файле описан создаваемый инстантс
[replicaset.yaml](../practice/3.application-abstractions/2.replicaset/replicaset.yaml)

получим rs
```bash
NAME            DESIRED   CURRENT   READY   AGE
my-replicaset   2         2         2       69s
```
      DESIRED - запрошено 2 реплики
      CURRENT - создано 2
      READY - готово к использованию 2

получим поды
```bash
NAME                  READY   STATUS    RESTARTS   AGE
my-replicaset-7fcl2   1/1     Running   0          5m34s
my-replicaset-zdb5j   1/1     Running   0          5m34s
```
Видим, что проблему с неймингами нам решил rs

Как можно поскейлить прилу
- поправить конфиг rs и сделать apply
- kubectl scale --replicas 3 replicaset my-replicaset 

Попытаемся обновить приложение. У нас nginx 12, а мы подсунем 13
- kubectl set image replicaset my-replicaset nginx=quay.io/testing-farm/nginx:1.13

Теперь если мы получим describe по RS то мы увидим, что образ обновлен
```bash
Pod Template:
  Labels:  app=my-app
  Containers:
   nginx:
    Image:         quay.io/testing-farm/nginx:1.13
```
а если мы получим сейчас поды, то мы увидим, что он давно живут и они явно не обновились. Все дело в том, 
что мы лишь заменили образ в темплейте, но никто поды не пересоздавал. Но если мы сейчас грохнем 1 под, то RS поднимет 
нам недостающий под и он будет на новом образе.

RS не решает проблему обновления приложения7 Он следит за количеством под и поддерживает это

## Deployment
в файле описан создаваемый инстантс
[deployment.yaml](../practice/3.application-abstractions/3.deployment/deployment.yaml)

kubectl edit deploy - обратились к апи кубера, попросили описание деплоймента. Отредачим его, сохраним и выйдем, 
это изменение попадет не в мой локльный файл, а полетит в куб и применится. Это хороши способ на ходу что-то менять 
и дебажить, но плохой способ это делать на всех, т.к. непонятно, кто это накатил и зачем

открылся vim
- i - insert
- правим то что надо, как в обычном редакторе
- :wq - сохранить и выйти

Что произошло: У нас создался новый репликаСет и накатился вместе с новыми подами. А если мы сейчас получим rs,
то увидим две RS
```bash
my-deployment-84cdc578bb   0         0         0       25m
my-deployment-d9ffb897f    2         2         2       4m17s
```
Это произошло потому: что куб убивал один старый под и поднимал один новый. и так до тех пор, пока новых не стало
2 заявленных. Старый оставил потому, что мы может откатиться на него
```bash
kubectl rollout undo deployment my-deployment
```
Глубину отката можно регулировать в спеке деплоймента. По умолчанию revisionHistoryLimit=10 

### Стратегия обновления

Можно реплики распределять как по штукам, так и по процентам

```bash
replicas: 2
#  стратегия обновления подов.
#  maxSurge - на сколько больше нормы подов мы можем поднять во время обновления
#  maxUnavailable - на сколько можем снизить. В данном случае будет так
strategy:
rollingUpdate:
#    можно сделать на 1 под больше запрошенного, во время rollingUpdate
      maxSurge: 1
#      нельзя опускать ни на один под во ремя обновления
      maxUnavailable: 0
```

При таком раскладе, у нас создастся +1 реплика и она будет новой - из станет три всего. 
Затем прибхется 1 старая. Накатится снова +1 новая и убъется последняя старая

```bash
replicas: 2
strategy:
rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```
При таком раскладе, у нас убъется 1 старая, создастся 2 новые и убъется последняя старая

```bash
replicas: 1
strategy:
rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```
При таком раскладе, мы не получим даунТайм тогда, когдка конфиг будет таким. 
Создасться +1 и затем убъется старая реплика

## Resources

[deployment-with-resources.yaml](../practice/3.application-abstractions/4.resources/deployment-with-resources.yaml)

Некоторые ресурсы могут быть для отдельного неймспейса, а некоторые для всего кластера. Ресусы бывают:
- Limits (настоящие ресурсы, типа cpu/memory)
  - количество ресурсов, которые под может использовать
  - Верхняя граница
- Requests (числа в ямле. Для того, что бы кубер понимал, на какую ноду это можно вообще приземлить. 
Рекваесты не имеют никакого отношения к реальному потреблению ресурсов подами)
  - количество ресурсов, которые резервируются для подв нв ноде
  - Не делятся с другими подами на ноде

#### QoS Class
Это свойство, которое можно посмотреть в описании к поду и оно означает:

- BestEffort - Если Limits и Requests не указаны в ямле, значит что прила может потреблять все ресурсы ноды. 
    Если ноде становится плохо, такие поде переносятся с ноды в первую очередь. 
- Burstable - Если Limits больше чем Requests - такие во вторую очередь переносятся с нод.
- Guaranteed - Если Requests и Limits равнф, кубер до последнего такие поды не трогает. 

Если задрать ресурсы: которые некуда пристроить, то мы получим статус пода в Pending. 
Тогда идем в describe, там есть events и там читать, что случилось

