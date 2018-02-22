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
      Images can be found here:
      http://www.osboxes.org/centos/
      http://www.osboxes.org/ubuntu/
      
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
    
  3.4 Testing kafka with python
      
    Install kafka module in python:
     
      # pip install kafka
        
    Start python and setup consumer:
        
      # python
      >>> from kafka import KafkaConsumer
      >>> consumer = KafkaConsumer('mytopic', bootstrap_servers='kafka:9092')
      >>> for message in consumer:
      ...   print message
      ...
      ConsumerRecord(topic=u'mytopic', partition=0, offset=1, timestamp=1513594024823, timestamp_type=0, key=None, value='message 4', checksum=-179384026, serialized_key_size=-1, serialized_value_size=9)
      ConsumerRecord(topic=u'mytopic', partition=1, offset=2, timestamp=1513594021786, timestamp_type=0, key=None, value='message 5', checksum=-1033357726, serialized_key_size=-1, serialized_value_size=9)
        
    Open new terminal window and start producer:
      # python
      >>> from kafka import KafkaProducer
      >>> producer = KafkaProducer(bootstrap_servers='kafka:9092')
      >>> producer.send('mytopic', value=b'message 4')
      >>> producer.send('mytopic', value=b'message 5')
  
    How to get this working in windows:

      Add kafka hostname to localhost ip in windows hosts file (C:\Windows\System32\drivers\etc\hosts)

        127.0.0.1 kafka.kafka.svc.cluster.local kafka
  
      Configure port forwarding to minikube virtual machine from Oracle VirtualBox
      
        Open minikube virtual machine settings... --> Network --> Adapter 1 --> advanced -->  Port Forwaring 
     
        Add new row and set following values:
        
        Name = kafka (some name)
        Host IP= 127.0.0.1 (localhost IP what is added to hosts file>)
        Host Port=9092 (Kafkas real port) 
        Guest IP=192.168.99.100 (minikube ip can get with command "minikube ip")
        Guest Port=31901 (external kafka port, for example from "minikube dashboard")
