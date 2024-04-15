---
title: go-micro-broker-kafka-demo
date: 2020-06-10 08:13:33
tags:
 - go-micro
 - kafka
---

# 困惑
有点小心动要研究一下 go-micro 的 broker，正好在研究 kafka，想找个 demo 尝试一下，但是真让人头大，并没有找到很容易上手的案例，于是自己码一篇来帮助有兴趣研究的朋友。

# V2 版本
## 代码
**在 https://github.com/micro/examples v2 版本下面的 broker/main.go 中增加了一行代码**
```go
_ "github.com/micro/go-plugins/broker/kafka/v2"
```

<!--more-->
具体代码如下：

```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/micro/go-micro/v2/broker"
	"github.com/micro/go-micro/v2/config/cmd"
	_ "github.com/micro/go-plugins/broker/kafka/v2"
)

var (
	topic = "go.micro.topic.foo"
)

func pub() {
	tick := time.NewTicker(time.Second)
	i := 0
	for _ = range tick.C {
		msg := &broker.Message{
			Header: map[string]string{
				"id": fmt.Sprintf("%d", i),
			},
			Body: []byte(fmt.Sprintf("%d: %s", i, time.Now().String())),
		}
		if err := broker.Publish(topic, msg); err != nil {
			log.Printf("[pub] failed: %v", err)
		} else {
			fmt.Println("[pub] pubbed message:", string(msg.Body))
		}
		i++
	}
}

func sub() {
	_, err := broker.Subscribe(topic, func(p broker.Event) error {
		fmt.Println("[sub] received message:", string(p.Message().Body), "header", p.Message().Header)
		return nil
	})
	if err != nil {
		fmt.Println(err)
	}
}

func main() {
	cmd.Init()

	if err := broker.Init(); err != nil {
		log.Fatalf("Broker Init error: %v", err)
	}
	if err := broker.Connect(); err != nil {
		log.Fatalf("Broker Connect error: %v", err)
	}

	go pub()
	go sub()

	<-time.After(time.Second * 10)
}
```

# V1 版本
## 代码
**在 https://github.com/micro/examples v1 版本目录下面的 broker/main.go 中增加了一行代码**
```go
_ "github.com/micro/go-plugins/broker/kafka"
```

具体代码如下：

```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/micro/go-micro/broker"
	"github.com/micro/go-micro/config/cmd"
	_ "github.com/micro/go-plugins/broker/kafka"
)

var (
	topic = "go.micro.topic.foo"
)

func pub() {
	tick := time.NewTicker(time.Second)
	i := 0
	for _ = range tick.C {
		msg := &broker.Message{
			Header: map[string]string{
				"id": fmt.Sprintf("%d", i),
			},
			Body: []byte(fmt.Sprintf("%d: %s", i, time.Now().String())),
		}
		if err := broker.Publish(topic, msg); err != nil {
			log.Printf("[pub] failed: %v", err)
		} else {
			fmt.Println("[pub] pubbed message:", string(msg.Body))
		}
		i++
	}
}

func sub() {
	_, err := broker.Subscribe(topic, func(p broker.Event) error {
		fmt.Println("[sub] received message:", string(p.Message().Body), "header", p.Message().Header)
		return nil
	})
	if err != nil {
		fmt.Println(err)
	}
}

func main() {
	cmd.Init()

	if err := broker.Init(); err != nil {
		log.Fatalf("Broker Init error: %v", err)
	}
	if err := broker.Connect(); err != nil {
		log.Fatalf("Broker Connect error: %v", err)
	}

	go pub()
	go sub()

	<-time.After(time.Second * 10)
}
```
# 运行
**请使用你的 kafka 的地址**
```
go run main.go --broker=kafka --broker_address=192.168.0.111:9092
```
# 结果
```
[pub] pubbed message: 0: 2020-06-10 06:44:50.824034186 +0800 CST m=+1.011746054
[sub] received message: 0: 2020-06-10 06:44:50.824034186 +0800 CST m=+1.011746054 header map[id:0]
[pub] pubbed message: 1: 2020-06-10 06:44:51.824509526 +0800 CST m=+2.012221507
[sub] received message: 1: 2020-06-10 06:44:51.824509526 +0800 CST m=+2.012221507 header map[id:1]
[pub] pubbed message: 2: 2020-06-10 06:44:52.824126208 +0800 CST m=+3.011838083
[sub] received message: 2: 2020-06-10 06:44:52.824126208 +0800 CST m=+3.011838083 header map[id:2]
[pub] pubbed message: 3: 2020-06-10 06:44:53.824808121 +0800 CST m=+4.012520018
[sub] received message: 3: 2020-06-10 06:44:53.824808121 +0800 CST m=+4.012520018 header map[id:3]
[pub] pubbed message: 4: 2020-06-10 06:44:54.824376704 +0800 CST m=+5.012088806
[sub] received message: 4: 2020-06-10 06:44:54.824376704 +0800 CST m=+5.012088806 header map[id:4]
[pub] pubbed message: 5: 2020-06-10 06:44:55.824832097 +0800 CST m=+6.012544119
[sub] received message: 5: 2020-06-10 06:44:55.824832097 +0800 CST m=+6.012544119 header map[id:5]
[pub] pubbed message: 6: 2020-06-10 06:44:56.824383524 +0800 CST m=+7.012095620
[sub] received message: 6: 2020-06-10 06:44:56.824383524 +0800 CST m=+7.012095620 header map[id:6]
[pub] pubbed message: 7: 2020-06-10 06:44:57.824511069 +0800 CST m=+8.012222970
[sub] received message: 7: 2020-06-10 06:44:57.824511069 +0800 CST m=+8.012222970 header map[id:7]
[sub] received message: 8: 2020-06-10 06:44:58.824249742 +0800 CST m=+9.011961623 header map[id:8]
[pub] pubbed message: 8: 2020-06-10 06:44:58.824249742 +0800 CST m=+9.011961623
```
# kafka 结果查看
### topic
```
$ ./kafka-topics.sh  --zookeeper localhost:2181 --list
__consumer_offsets
go.micro.topic.foo
```
### content
```
$ ./kafka-console-consumer.sh --bootstrap-server 192.168.0.111:9092 --topic go.micro.topic.foo --from-beginning
{"Header":{"id":"0"},"Body":"MDogMjAyMC0wNi0xMCAwNjozODowOS41MDM1MTE1MjggKzA4MDAgQ1NUIG09KzEuMDE0MDAzOTY3"}
{"Header":{"id":"1"},"Body":"MTogMjAyMC0wNi0xMCAwNjozODoxMC41MDMxODM5ODUgKzA4MDAgQ1NUIG09KzIuMDEzNjc2Mzgw"}
{"Header":{"id":"2"},"Body":"MjogMjAyMC0wNi0xMCAwNjozODoxMS41MDMyNTQgKzA4MDAgQ1NUIG09KzMuMDEzNzQ2NDM4"}
{"Header":{"id":"3"},"Body":"MzogMjAyMC0wNi0xMCAwNjozODoxMi41MDM3Nzc4OTUgKzA4MDAgQ1NUIG09KzQuMDE0MjcwNDYy"}
{"Header":{"id":"4"},"Body":"NDogMjAyMC0wNi0xMCAwNjozODoxMy41MDMyMjc2MjcgKzA4MDAgQ1NUIG09KzUuMDEzNzIwMDg2"}
{"Header":{"id":"5"},"Body":"NTogMjAyMC0wNi0xMCAwNjozODoxNC41MDI4OTI0ODEgKzA4MDAgQ1NUIG09KzYuMDEzMzg0ODY4"}
{"Header":{"id":"6"},"Body":"NjogMjAyMC0wNi0xMCAwNjozODoxNS41MDMyMzYyICswODAwIENTVCBtPSs3LjAxMzcyODc2OA=="}
{"Header":{"id":"7"},"Body":"NzogMjAyMC0wNi0xMCAwNjozODoxNi41MDMzMzkyMTggKzA4MDAgQ1NUIG09KzguMDEzODMxNjM0"}
{"Header":{"id":"8"},"Body":"ODogMjAyMC0wNi0xMCAwNjozODoxNy41MDMwNTc3NzggKzA4MDAgQ1NUIG09KzkuMDEzNTUwMzEw"}
{"Header":{"id":"9"},"Body":"OTogMjAyMC0wNi0xMCAwNjozODoxOC41MDMyMjAxMzcgKzA4MDAgQ1NUIG09KzEwLjAxMzcxMjcwNQ=="}
```
# 问题
v1 版本由于 kafka 会引入 **github.com/DataDog/zstd** 会导致程序无法在 Windows 下面运行，以上代码在 **CentOS7** 上运行正常

