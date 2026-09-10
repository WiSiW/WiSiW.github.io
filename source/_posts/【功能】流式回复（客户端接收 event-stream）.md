---
title: 【功能】流式回复（客户端接收 text/event-stream
date: 2023-11-01 15:00:37
tags:
  - 流式回复
  - event-stream
---

1.从流式接口中获取

```javascript
fetch(
  "",
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({}) // params
  }
).then((res) => {
  const reader = res.body.getReader();
  const decoder = new TextDecoder();

  return reader.read().then(function processResult(result) {
    // result { done, value }
    console.log(result);
    // end
    if (result.done) {
      // reader.cancel()
      return;
    }

    const chunk = decoder.decode(result.value, {
      stream: true
    });

    // do something
    console.log(chunk);

    return reader.read().then(processResult);
  });
})
.catch(error => {
  console.error('Error occurred while fetching event stream:', error);
});
```


# 2.后端Java
```java
@PostMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter eventStream(@RequestBody Object obj) {
    SseEmitter emitter = new SseEmitter();
    System.out.println(obj.toString());
    ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
    executorService.scheduleAtFixedRate(() -> {
        try {
            // 模拟产生一条事件数据
            String eventData = "Event data: " + System.currentTimeMillis();
            emitter.send(SseEmitter.event().data(eventData));
        } catch (IOException e) {
            emitter.complete();
        }
    }, 0, 1, TimeUnit.SECONDS);
    return emitter;
}
```