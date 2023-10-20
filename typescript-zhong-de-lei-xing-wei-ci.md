# Typescript中的类型谓词

这个概念是我在询问 gpt 如何使用 Set 时发生的，当时我的一个问题是：`如何判断一个变量是 Set 类型。`它给我举的例子:

```typescript
function isSet(variable: any): variable is Set<any> {
  return variable instanceof Set;
}
```

我很疑惑这里的 variable is Set 和 boolean 有什么区别。



实际上在这个场景下， boolean 能直接替换，也就是变成这样：

```typescript
function isSet(variable: any): boolean {
  return variable instanceof Set;
}
```



所以我没搞懂他们的区别。在和 gpt 一番纠缠之后，我明白了，实际上这个 `variable is Set` 是为了让 ts 编译器去识别当前所引入的变量是什么类型。

再举一个直观的例子：

```typescript
interface Cat {
  name: string;
  meow(): void;
}

interface Dog {
  name: string;
  bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return 'meow' in animal;
}

function performSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    // 在这个块中，TypeScript 知道 animal 是 Cat 类型
    animal.meow();
  } else {
    // 在这个块中，TypeScript 知道 animal 是 Dog 类型
    animal.bark();
  }
}
```

这里使用 `animal is Cat` 的写法，当在 performSound 方法中，走到第一个 if 内的时候，ts 就知道了当前的 animal 对象确确实实为 Cat 类型。

而如果我使用 boolean 来替换，编辑器就会报错：

<div align="left">

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

 

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

</div>

因为此时，即使走到了 if isCat 里面， ts 也无法得知当前的 animal 到底为什么子类型。

添上 gpt 的补充：

> 类型谓词主要用于以下情况：
>
> 1. **更具体的类型判断**: 类型谓词可以提供更具体的类型信息，以便在后续代码中使用该类型。例如，在示例 1 中，使用 `isArray` 类型谓词后，TypeScript 知道 `data` 是一个数组类型，可以安全地访问数组的方法和属性。
> 2. **类型守卫和类型推断**: 类型谓词可以用于类型守卫和类型推断，以帮助 TypeScript 推断变量的具体类型。例如，在示例 2 中，使用 `hasNameProperty` 类型谓词后，TypeScript 推断出 `person` 具有 `name` 属性，并且是 `Person` 类型。
>
> \
> 当涉及到联合类型和类型缩小时，类型谓词可以提供更精确的类型信息，而使用布尔类型无法替代。\
>
>
> 类型谓词的主要目的是为了在 TypeScript 中提供更精确的类型信息，以便 TypeScript 编译器能够在特定的代码块或语境中识别出变量的具体类型。
>
> 通过使用类型谓词，你告诉 TypeScript 编译器某个条件是类型的一种判断，从而缩小了变量的类型范围。这样，TypeScript 编译器就能够利用这些更具体的类型信息来进行类型推断、类型检查和类型守卫，以提供更准确的静态类型检查和更智能的代码补全。
>
> 类型谓词的使用可以使代码更可靠、更易于维护，并提供更好的开发体验。它允许你在代码中明确指定某个变量是某个特定类型，而不仅仅是布尔类型的真假判断。





