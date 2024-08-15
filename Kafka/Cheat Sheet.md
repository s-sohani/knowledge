
./[kafka-topics.sh](http://kafka-topics.sh/) --bootstrap-server 172.17.0.141:9092 --list  
  
./[kafka-topics.sh](http://kafka-topics.sh/) --bootstrap-server 172.17.0.141:9092 --topics-with-overrides --describe | grep graph-googleclone-keys  
  
/opt/kafka/bin/kafka-topics.sh --bootstrap-server 172.17.0.141:9092 --topic filter-anchor --alter --partitions 44  
  
/opt/kafka/bin/kafka-configs.sh --bootstrap-server 172.17.0.141:9092 --alter --topic filter-anchor --add-config retention.ms=259200000  
  
/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server 172.17.0.141:9092 --group graph-page-quality --describe | awk {'print $3" "$4'} | sort  
  
./[kafka-topics.sh](http://kafka-topics.sh/) --bootstrap-server kaf4:9092 --topic to-download --describe  
  
  
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.17.0.141:9092 --topic filter-anchor --partition 9 --offset 60614766 --max-messages 1  
  
/opt/kafka/bin/kafka-topics.sh --bootstrap-server 172.17.0.141:9092 --topic test --replication-factor=1 --partitions 20 --create  
  
/opt/kafka/bin/kafka-topics.sh --bootstrap-server 172.17.0.141:9092 --topic test --delete  
  
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic filter-anchor | grep -P '^\{"url":( )?"http(s)?://(www.)?ketabettelaat.com'  
  
./[kafka-console-consumer.sh](http://kafka-console-consumer.sh/) --bootstrap-server kaf3:9092 --topic test --from-beginning  
  
./[kafka-console-producer.sh](http://kafka-console-producer.sh/) --broker-list kaf2:9092 --topic test