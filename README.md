# NEO-hackathon-vol1-Team-John_Doe
Team John Doe


This repository contains instructions and configuration files to create stateless Kafka with Kubernetes Minikube.

Installation:

1. Setup Kubernetes Minikube:

https://github.com/TampereTC/NEO-hackathon-vol1-Team-John_Doe/blob/master/Minikube%20installation

2. Setup Kafka:

https://github.com/TampereTC/NEO-hackathon-vol1-Team-John_Doe/tree/master/kafka

kubectl create namespace kafka
kubectl --namespace kafka create -f zookeeper.yaml
kubectl --namespace kafka create -f zookeeper-services.yaml
kubectl --namespace kafka create -f kafka-controller.yaml
kubectl --namespace kafka create -f kafka-service-external.yaml

3. Testing

3.1 Local

  List pods:
    kubectl get pods --namespace kafka
    NAME                           READY     STATUS    RESTARTS   AGE
    kafka-controller-t2dq4         1/1       Running   0          2m
    zookeeper-controller-1-mjm7t   1/1       Running   0          3m
    zookeeper-controller-2-98rsv   1/1       Running   0          3m
    zookeeper-controller-3-f2hb9   1/1       Running   0          3m
 
  Login to kafka pod console:
    kubectl exec -it kafka-controller-t2dq4 /bin/bash --namespace kafka 
      # cd /opt/kafka
      Sent message to topic:
      # ./bin/kafka-console-producer.sh --broker-list kafka:9092 --topic mytopic
      > message 1
      Ctrl-C
      Consume message from topic:
      # ./bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic mytopic --from-beginning
      message 1
      Ctrl-C
      Exit from console:
      # exit
      
  3.2 Kubernetes internal
  
    Create kafka consumer and producer pods:
      kubectl --namespace kafka create -f kafka-client-producer.yaml
      kubectl --namespace kafka create -f kafka-client-consumer.yaml
      
    Login to producer pod console and send message to kafka:
      kubectl exec -it testclient-producer /bin/bash --namespace kafka
        # ./bin/kafka-console-producer.sh --broker-list kafka:9092 --topic mytopic
        > message 2
        Ctrl-C
    Open second CMD window and login to consumer pod console:
      kubectl exec -it testclient-consumer /bin/bash --namespace kafka
        # ./bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic mytopic --from-beginning
        message 1
        message 2
        Ctrl-C
    
  3.3 External connection to kafka
  
    In VirtualBox instantiate new CentOS or Ubuntu VM

    Download and unpack Kafka:
    https://www.apache.org/dyn/closer.cgi?path=/kafka/1.0.0/kafka_2.11-1.0.0.tgz

    Add kafka controller FQDN to /etc/hosts:
    <minikube IP> kafka.kafka.svc.cluster.local kafka 
    
    Find out kafka external port in kuberentes:
    kubectl get services --namespace kafka
    NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
    kafka     NodePort    10.0.0.36    <none>        9092:<kafka service external port>/TCP               1m
  
    Add reverse NAT in Linux host:
     iptables -t nat -A OUTPUT -p tcp -d <minikube IP> --dport 9092 -j DNAT --to-destination <minikube IP>:<kafka service external port>
    
    Change to kafka directory and run producer:
      # cd kafka_2.11-1.0.0
      # ./bin/kafka-console-producer.sh --broker-list kafka:9092 --topic mytopic
        > message 3
        Ctrl-C
      # ./bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic mytopic --from-beginning
        message 1
        message 2
        message 3
        Ctrl-C
    
  
