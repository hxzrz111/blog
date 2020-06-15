---
title: kafka学习-安装篇
date: 2019-06-25 23:52:50
tags: kakfa
categories: kafka
---

# docker 安装 zookeeper kafka

## 1. docker 下载 zookeeper and kafka

	//下载zookeeper
	docker pull wurstmeister/zookeeper

	//下载kafka
	docker pull wurstmeister/kafka

## 2. 启动镜像

	//启动zookeeper	
	docker run -d --name zookeeper --publish 2181:2181 --volume /etc/localtime:/etc/localtime wurstmeister/zookeeper

	//启动kafka
	docker run -d --name kafka --publish 9092:9092 \
	--link zookeeper \
	--env KAFKA_ZOOKEEPER_CONNECT=127.0.0.1:2181 \
	--env KAFKA_ADVERTISED_HOST_NAME=127.0.0.1 \
	--env KAFKA_ADVERTISED_PORT=9092  \
	--volume /etc/localtime:/etc/localtime \
	wurstmeister/kafka

	127.0.0.1 换成自己的ip

## 3. 进入容器

	docker exec -it kafka /bin/sh

	cd opt/kafka_2.12-2.2.1/bin

## 4. 测试kafka

	// 创建topic
	./kafka-topics.sh --create --zookeeper 47.104.178.113:2181 --replication-factor 1 --partitions 1 --topic mykafka
	
	// 查看topic
	./kafka-topics.sh --list --zookeeper 47.104.178.113:2181
	
	// 创建生产者
	./kafka-console-producer.sh --broker-list 47.104.178.113:9092 --topic mykafka 
	
	// 让我们用kafka发送一条消息
	{'hello','你好kafka'}

	// 创建消费者
	./kafka-console-consumer.sh --zookeeper 47.104.178.113:2181 --topic mykafka --from-beginning
