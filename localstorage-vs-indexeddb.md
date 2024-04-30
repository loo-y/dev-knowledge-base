# localStorage VS IndexedDB

### 基本概念

在Web开发中，本地存储技术是实现离线应用和提高用户体验的关键。`localStorage`和`IndexedDB`是两种常用的浏览器本地存储技术，它们各自有不同的特点和适用场景。

* **localStorage**：是一种简单的键值存储系统，允许你以字符串的形式存储数据。它的主要用途是缓存数据，以便在用户再次访问网页时能够快速加载。`localStorage`适用于小型数据存储，如用户偏好设置、会话信息等。
* **IndexedDB**：是一个更为复杂的NoSQL数据库，它提供了一个非关系型数据库，可以在客户端存储大量结构化数据。`IndexedDB`支持多种数据类型，并且可以通过索引快速查询数据。它适用于需要存储大量数据和复杂数据结构的应用场景，如游戏、地图应用等。

### **TL;DR:**

* **localStorage** 是一个简单的键值存储，适合小型数据存储，如用户设置。并且有最大容量限制 (5M)。
* **IndexedDB** 是一个更复杂的NoSQL数据库，适合存储大量结构化数据，支持异步操作，不会阻塞主线程。
* **Dexie.js** 是一个封装了 IndexedDB 的库，简化了 API，使得操作更加容易和高效。使用 Dexie，可以轻松地进行数据的增删改查操作，并且它支持 Promise 和 async/await。
* 对于需要高效管理和操作大量数据的Web应用，尤其是在离线环境下，使用 IndexedDB 和 Dexie.js 是一个优秀的选择。



### 特性分析

#### 兼容性

localStorage 兼容性

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

IndexedDB 兼容性

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

我们可以看到，在现代浏览器中，主流内核基本已经支持 IndexedDB。

#### 存储容量

* `localStorage`的存储容量受限于浏览器和设备的限制，通常在5MB左右。
* `IndexedDB`的存储容量则大得多，理论上可以存储数GB的数据，具体取决于用户的硬盘空间。

#### 数据类型支持

* `localStorage`仅支持字符串类型的数据。
* `IndexedDB`支持多种数据类型，包括字符串、数字、布尔值、日期等，还可以存储二进制数据和对象。

#### API易用性

* `localStorage` 的 API 简单明了，易于上手。一般只需要使用 getItem、setItem、removeItem 等方法即可。而 IndexedDB 的 API 更加复杂，但也更为强大，它支持数据的增删改查，事务处理，索引创建和查找等操作。
* `IndexedDB`的API较为复杂，但也更为强大。支持数据的增删改查，事务处理，索引创建和查找，对象仓库的操作等，学习曲线较陡。

#### IndexedDB的异步优势

`IndexedDB`的操作是基于异步的，这意味着它不会阻塞主线程，从而不会影响用户界面的响应性。在执行数据库读写操作时，`IndexedDB`会在后台创建一个新的事务线程，这样可以避免长时间的I/O操作阻塞UI线程，确保Web应用的流畅运行。

这种异步性对于Web应用来说非常重要，尤其是对于需要处理大量数据或者复杂计算的应用。例如，一个需要在浏览器端处理大量数据的复杂 Web 应用，如图片编辑器或者大型游戏等，可能需要加载和处理大量的资源和用户数据，这些操作同步进行，可能会导致应用卡顿甚至崩溃，此时就应该选择 IndexedDB 作为你的数据存储方案。同样的，如果正在开发一个 PWA（Progressive Web App） ，它需要在离线状态下也可以访问大量数据，IndexedDB 也会是一个好的选择。因为它既能支持大量的数据存储，又能通过异步操作保证应用的流畅运行。

由于 IndexedDB 操作是异步的，并且默认情况下不会阻塞主线程，因此它通常不会直接影响 TBT。

#### IndexedDB 核心概念

`IndexedDB` 的核心组成部分包括数据库（Database）、对象仓库（Object Store）、索引（Index）和事务（Transaction）。&#x20;

对象仓库是 `IndexedDB` 中存储数据的地方，你可以把它想象成关系数据库中的表。&#x20;

索引是与对象仓库相关联的，它可以让你高效地查询数据。&#x20;

事务用来处理数据库的读写操作，以保证数据库的完整性和一致性。

以下是一个简单的 CRUD 的操作示例

```javascript
// 打开或创建一个 IndexedDB 数据库
let request = window.indexedDB.open("myDatabase", 1);

// 添加数据记录
request.onsuccess = function(event) {
  let db = event.target.result;
  let transaction = db.transaction(["myObjectStore"], "readwrite");
  let objectStore = transaction.objectStore("myObjectStore");
  objectStore.add({ id: 1, name: "my name" });
};

// 读取数据记录
transaction = db.transaction(["myObjectStore"]);
objectStore = transaction.objectStore("myObjectStore");
let request = objectStore.get(1);
request.onsuccess = function(event) {
  console.log("Name: ", request.result.name);
};

// 更新数据记录
transaction = db.transaction(["myObjectStore"], "readwrite");
objectStore = transaction.objectStore("myObjectStore");
request = objectStore.get(1);
request.onsuccess = function(event) {
  request.result.name = "new name";
  objectStore.put(request.result);
};

// 删除数据记录
transaction = db.transaction(["myObjectStore"], "readwrite");
objectStore = transaction.objectStore("myObjectStore");
objectStore.delete(1);

// 使用索引查询数据
transaction = db.transaction(["myObjectStore"]);
objectStore = transaction.objectStore("myObjectStore");
index = objectStore.index("name");
request = index.get("new name");
request.onsuccess = function(event) {
  console.log("ID: ", request.result.id);
};
```



### Dexie: 简便地使用 IndexedDB

即使`IndexedDB` 相比 `localStorage` 有诸多优点，但正是由于其复杂的 API，导致在一般项目中很难看到其身影。这里推荐使用[Dexie](https://dexie.org/)。Dexie.js 是一个处理 IndexedDB 的 JavaScript 库。

```javascript
import Dexie from 'dexie';

const db = new Dexie('MyDatabase');
db.version(1).stores({
    friends: 'name, age' // 定义对象仓库和索引
});

// 添加数据
db.friends.add({
    name: 'John',
    age: 25
});

// 使用索引查询
db.friends.where('age').below(30).each(friend => {
    console.log(friend.name);
});
```

可以看到使用 Dexie.js 让代码更加简洁清晰，更易于理解和维护。

#### Dexie.js 的核心特性

* **易于使用的 API**：Dexie.js 提供了一个更易于使用的 API，隐藏了 IndexedDB 的复杂性，它提供了一个类似于 SQL 数据库的 API，包括表的创建、数据的增删改查等操作。
* **Promise 和 async/await 支持**：Dexie.js 的所有操作都是基于 Promise 的，并且可以与 async/await 配合使用，使得异步代码更加简洁。
* **事务管理**：Dexie.js 提供了对 IndexedDB 事务的封装，使得你可以轻松地管理数据的一致性和完整性。
* **索引和查询**：Dexie.js 支持创建多种类型的索引，包括唯一索引、多值索引和复合索引，以及执行复杂的查询操作。
* **错误处理**：Dexie.js 提供了更加清晰的错误处理机制，使得调试和错误追踪更加容易。
* **性能优化**：Dexie.js 利用了 IndexedDB 的一些高级特性来优化性能，例如批量操作和避免不必要的事件监听。

#### 使用示例

假设我们现在要在一个 web 应用中使用 Dexie 和 IndexedDB。我们可用新建一个 db.ts

{% code title="shared/db.ts" %}
```typescript
import Dexie, { Table } from 'dexie'

interface DBChatItem {
    id?: number
    chatId: string
    role: string
    text: string
    timestamp: number
}

class IndexedDexie extends Dexie {
    chats!: Table<DBChatItem>
    
    constructor(dbname: string) {
        super(dbname)
        this.version(1.0).stores({
            chats: '++id, chatId, role, text, timestamp'
        })
    }
}

export const chatDb = new IndexedDexie('chatDb')
```
{% endcode %}

现在我们有了一个版本号为 1.0，名称为 chatDb 的 IndexedDB

当我们需要使用这个 chatDb 的时候，我们只需要在其他文件中：

```typescript
import { chatDb } from '../shared/db'

// 新增对话记录
function addNewChat (text) {
    chatDb.chats.add({
        chatId: 1,
        role: "user",
        text: text,
        timestamp: Date.now(),
    })
}

// 更新某一条对话记录
async function updateChat (chatId, text) {
    const theChatInDB = await chatDb.chat.get({ chatId })
    if (theChatInDB?.id) {
        await chatDb.chat.update(theChatInDB.id, { text })
    }
}
```



无论是直接使用 IndexedDB 还是通过 Dexie 这样的库来操作，都能够显著提升 Web 应用的性能和用户体验。它们使得 Web 应用能够离线工作，处理大量数据，并提供快速的响应能力。对于那些寻求更高效、更易于管理的数据库操作的开发需求，使用 Dexie 来操作 IndexedDB 是一个合适的方案。

