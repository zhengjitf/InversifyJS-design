## Reflect Metadata

### Reflect.defineMetadata()
在对象或属性上定义 metadata

```js
Reflect.defineMetadata(metadataKey, metadataValue, target);
Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey);
```

### Reflect.hasMetadata()
检查对象或属性的原型链上是否存在指定元数据 key

```js
let result = Reflect.hasMetadata(metadataKey, target);
let result = Reflect.hasMetadata(metadataKey, target, propertyKey);
```

### Reflect.hasOwnMetadata()
检查对象或属性自身是否存在指定元数据 key

```js
let result = Reflect.hasOwnMetadata(metadataKey, target);
let result = Reflect.hasOwnMetadata(metadataKey, target, propertyKey);
```

### Reflect.getMetadata()
获取对象或属性的原型链上的指定元数据 key 的元数据值

```js
let result = Reflect.getMetadata(metadataKey, target);
let result = Reflect.getMetadata(metadataKey, target, propertyKey);
```

注：TS 中设置 `emitDecoratorMetadata` 为 `true` 会自动注入这些元数据，类似效果如下：

```js
// Design-time type annotations
function Type(type) { return Reflect.metadata("design:type", type); }
function ParamTypes(...types) { return Reflect.metadata("design:paramtypes", types); }
function ReturnType(type) { return Reflect.metadata("design:returntype", type); }

// Decorator application
@ParamTypes(String, Number)
class C {
  constructor(text, i) {
  }

  @Type(String)
  get name() { return "text"; }

  @Type(Function)
  @ParamTypes(Number, Number)
  @ReturnType(Number)
  add(x, y) {
    return x + y;
  }
}
```

#### 内置 metadataKey

```js
// 获取属性类型
Reflect.getMetadata("design:type", type)

// 获取函数参数类型
Reflect.getMetadata("design:paramtypes", target, key)

// 获取返回值类型
Reflect.getMetadata("design:returntype", target, key)
```

### Reflect.getOwnMetadata()
获取对象或属性自身的指定元数据 key 的元数据值

```js
let result = Reflect.getOwnMetadata(metadataKey, target);
let result = Reflect.getOwnMetadata(metadataKey, target, propertyKey);
```

### Reflect.getMetadataKeys()
获取对象或属性的原型链上的所有元数据 key

```js
let result = Reflect.getMetadataKeys(target);
let result = Reflect.getMetadataKeys(target, propertyKey);
```

### Reflect.getOwnMetadataKeys()
获取对象或属性自身的所有元数据 key

```js
let result = Reflect.getOwnMetadataKeys(target);
let result = Reflect.getOwnMetadataKeys(target, propertyKey);
```

### Reflect.deleteMetadata()
删除对象或属性指定元数据 key

```js
let result = Reflect.deleteMetadata(metadataKey, target);
let result = Reflect.deleteMetadata(metadataKey, target, propertyKey);
```

### @Reflect.metadata

```js
// apply metadata via a decorator to a constructor
@Reflect.metadata(metadataKey, metadataValue)
class C {
  // apply metadata via a decorator to a method (property)
  @Reflect.metadata(metadataKey, metadataValue)
  method() {
  }
}
```

---
Ref:
- [Metadata Proposal - ECMAScript](https://rbuckton.github.io/reflect-metadata/#introduction)
