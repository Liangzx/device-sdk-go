# tkeel device sdk for go

## Description

### Organization --TODO

- __mqtt__ - [paho.mqtt]()
- __examples__ - some example use this sdk
- __tkeel__ - ues mqtt to 
- __util__ - some useful util

### Quick start -- TODO:


```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "math/rand"
    "time"

    adpt "github.com/tkeel-io/device-sdk-go/adapter/mqtt"
    "github.com/tkeel-io/device-sdk-go/tkeel"
    "github.com/tkeel-io/device-sdk-go/util/wait"
)

func main() {
    log.SetFlags(log.Lshortfile)

    // 创建连接平台的 Adaptor
    conn, err := adpt.NewAdaptor(brokerAddr, "",
        adpt.WithAutoReconnect(true),
        adpt.WithCleanSession(false),
        adpt.WithUserName(username),
        adpt.WithPassword(pwd),
    )
    if err != nil {
        panic(err)
    }

    tk, err := tkeel.New("teek", conn)

    if err != nil {
        log.Fatal(err)
    }

    // 订阅设备关心的 topic
    vv := map[tkeel.Topic]tkeel.TopicHandle{
        tkeel.RawTopic:       rawTopicHandler,
        tkeel.CommandTopic:   commandsTopicHandler,
        tkeel.AttributeTopic: attributesTopicHandler,
    }

    for t, f := range vv {
        tk.On(t, f)
    }

    tm := time.Second * 1

    wait.EveryWithContext(context.Background(), func(ctx context.Context) {
        v, _ := deviceValue()
        // 推送遥测数据
        tk.Telemetry(v)
    }, tm)
}

func attributesTopicHandler(message adpt.Message) {
    fmt.Printf("attributes=%s\n", string(message.Payload()))
}

//
func commandsTopicHandler(message adpt.Message) {
    fmt.Printf("commands=%s\n", string(message.Payload()))
}

//
func rawTopicHandler(message adpt.Message) {
    fmt.Printf("rawTopic=%s\n", string(message.Payload()))
}

func deviceValue() ([]byte, error) {
    mv := map[string]interface{}{
        "val1": rand.Intn(20),
    }
    return json.Marshal(mv)
}
```



## Usage
> Assuming you already have [installed](https://golang.org/doc/install) Go

