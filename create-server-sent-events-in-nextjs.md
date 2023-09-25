# Create Server-Sent Events in Nextjs

由于Nextjs新的 App Router 采用的是 Mozilla 标准 WebAPI 的 [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response), 所以写法和普通的 Node SSE 略有不同。\
以下是代码示例：

**服务端 **_**app/api/stream/route.ts**_

```typescript
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
    const sseData = `:ok\n\nevent: message\ndata: Initial message\n\n`
    // 将 SSE 数据编码为 Uint8Array
    const encoder = new TextEncoder()
    const sseUint8Array = encoder.encode(sseData)

    // 创建 TransformStream
    const transformStream = new TransformStream({
        transform(chunk, controller) {
            controller.enqueue(chunk)
        },
    })

    // 创建 SSE 响应
    let response = new Response(transformStream.readable)

    // 设置响应头，指定使用 SSE
    response.headers.set('Content-Type', 'text/event-stream')
    response.headers.set('Cache-Control', 'no-cache')
    response.headers.set('Connection', 'keep-alive')
    response.headers.set('Transfer-Encoding', 'chunked')

    const writer = transformStream.writable.getWriter()
    writer.write(sseUint8Array)

    // 定义一个计数器
    let counter = 0

    // 每秒发送一个消息
    const interval = setInterval(() => {
        counter++

        if (counter > 10) {
            clearInterval(interval)
            return
        }

        const message = `event: message\ndata: Message ${counter}\n\n`
        const messageUint8Array = encoder.encode(message)
        writer.write(messageUint8Array)
    }, 1000)

    return response
}
```

\


**客户端**

```typescript
const eventSource = new EventSource('/sse/api/stream')
// 监听 SSE 事件的消息
eventSource.onmessage = function (event) {
    console.log('Received message:', event.data)
}
```

\


但是常规的 SSE 只可以通过GET来访问，这就导致请求方传递参数会有一些不那么优雅。如果需要传递大量文本时无法满足。

我们可以使用 Azure 的 [fetch-event-source](https://github.com/Azure/fetch-event-source)

在 Nextjs 中，服务端仅需将 `GET` 方法名改为 `POST`

客户端的修改如下：

```typescript
import { fetchEventSource } from '@microsoft/fetch-event-source';
const eventSourcePost = fetchEventSource('/sse/api/stream', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        foo: 'bar'
    }),
    onmessage: function (event) {
        console.log('Received message:', event.data)
    }
});
```

\
