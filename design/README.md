## 模型

### 源码目录

```bash
├── annotation # 注解，即一些包装器函数
│   ├── decorator_utils.ts
│   ├── inject.ts
│   ├── inject_base.ts
│   ├── injectable.ts
│   ├── lazy_service_identifier.ts
│   ├── multi_inject.ts
│   ├── named.ts
│   ├── optional.ts
│   ├── post_construct.ts
│   ├── pre_destroy.ts
│   ├── property_event_decorator.ts
│   ├── tagged.ts
│   ├── target_name.ts
│   └── unmanaged.ts
├── bindings # binding：主要是将 serviceIdentifier （@inject 的参数）与 具体的类绑定
│   ├── binding.ts
│   └── binding_count.ts
├── constants
│   ├── error_msgs.ts
│   ├── literal_types.ts
│   └── metadata_keys.ts
├── container # 容器，我们需要在容器中声明 bingding（创建绑定关系），然后获取实例
│   ├── container.ts
│   ├── container_module.ts
│   ├── container_snapshot.ts
│   ├── lookup.ts # Lookup： 查询类
│   └── module_activation_store.ts
├── interfaces
│   └── interfaces.ts
├── inversify.ts
├── planning
│   ├── context.ts
│   ├── metadata.ts
│   ├── metadata_reader.ts
│   ├── plan.ts
│   ├── planner.ts
│   ├── queryable_string.ts
│   ├── reflection_utils.ts
│   ├── request.ts
│   └── target.ts
├── resolution
│   ├── instantiation.ts
│   └── resolver.ts
├── scope
│   └── scope.ts
├── syntax # 主要处理 binding 后的一些操作
│   ├── binding_in_syntax.ts
│   ├── binding_in_when_on_syntax.ts
│   ├── binding_on_syntax.ts
│   ├── binding_to_syntax.ts
│   ├── binding_when_on_syntax.ts
│   ├── binding_when_syntax.ts
│   └── constraint_helpers.ts
```

### 主线
#### 几个概念
- `serviceIdentifier`: 服务标识符，即 `@inject(TYPES.Weapon)` 中的 `TYPES.Weapon`，用于标识 Service 类

#### @injectable
可被注入的类需要使用 `@injectable` 注解

```ts
function injectable() {
  return function <T extends abstract new (...args: never) => unknown>(target: T) {

    if (Reflect.hasOwnMetadata(METADATA_KEY.PARAM_TYPES, target)) {
      throw new Error(ERRORS_MSGS.DUPLICATED_INJECTABLE_DECORATOR);
    }

    // 这里获取到的是通过开启 emitDecoratorMetadata 生成的构造函数参数元数据
    const types = Reflect.getMetadata(METADATA_KEY.DESIGN_PARAM_TYPES, target) || [];
    Reflect.defineMetadata(METADATA_KEY.PARAM_TYPES, types, target);

    return target;
  };
}
```
主要逻辑是将构造函数的参数元数据，以自定义元数据键（`METADATA_KEY.PARAM_TYPES`）保存

#### @inject
使用 `@inject` 进行依赖标注

`inject` 调用的是 `injectBase`，传入的 `metadataKey` 为 `METADATA_KEY.INJECT_TAG`，
最终的包装器其实是通过 `createTaggedDecorator` 生成的，有两种类型，一种是包装（方法）参数（`tagParameter`），另一种是包装属性（`tagProperty`）

```ts
function injectBase(metadataKey: string) {
  return <T = unknown>(serviceIdentifier: ServiceIdentifierOrFunc<T>) => {
    return (
      target: DecoratorTarget,
      targetKey?: string | symbol,
      indexOrPropertyDescriptor?: number | TypedPropertyDescriptor<any>,
    ) => {
      if (serviceIdentifier === undefined) {
        const className = typeof target === "function" ? target.name : target.constructor.name;

        throw new Error(UNDEFINED_INJECT_ANNOTATION(className));
      }
      return createTaggedDecorator(
        new Metadata(metadataKey, serviceIdentifier)
      )(target, targetKey, indexOrPropertyDescriptor);
    };
  }
}
```

`tagParameter` 和 `tagProperty` 最终调用的都是 `_tagParameterOrProperty`

```ts
function _tagParameterOrProperty(
  // tagParameter： METADATA_KEY.TAGGED； tagProperty：METADATA_KEY.TAGGED_PROP
  metadataKey: string, 
  // tagParameter：原型；tagProperty：构造函数 TODO: ??????
  annotationTarget: NewableFunction,
  // tagParameter：参数下标；tagProperty：属性名
  key: string | symbol,
  // { key: METADATA_KEY.INJECT_TAG, value: serviceIdentifier }
  metadata: interfaces.MetadataOrMetadataArray,
) {
  const metadatas: interfaces.Metadata[] = _ensureNoMetadataKeyDuplicates(metadata);

  let paramsOrPropertiesMetadata: Record<string | symbol, interfaces.Metadata[] | undefined> = {};
  // read metadata if available
  // 取 annotationTarget 对象上的指定键的元数据。(一个类可能存在多个相同 metadataKey 的注解)
  if (Reflect.hasOwnMetadata(metadataKey, annotationTarget)) {
    paramsOrPropertiesMetadata = Reflect.getMetadata(metadataKey, annotationTarget);
  }

  let paramOrPropertyMetadata: interfaces.Metadata[] | undefined = paramsOrPropertiesMetadata[key as string];

  if (paramOrPropertyMetadata === undefined) {
    paramOrPropertyMetadata = [];
  } else { // 表示有相同 metadataKey 相同 key（属性/参数下标）的注解
    // 不允许有相同 key 的 metadata TODO: 不是很清晰
    for (const m of paramOrPropertyMetadata) {
      if (metadatas.some(md => md.key === m.key)) {
        throw new Error(`${ERROR_MSGS.DUPLICATED_METADATA} ${m.key.toString()}`);
      }
    }
  }

  // set metadata
  paramOrPropertyMetadata.push(...metadatas);
  paramsOrPropertiesMetadata[key] = paramOrPropertyMetadata;
  // paramsOrPropertiesMetadata 保存的就是 { [key]: metadata[] }
  // key 就是参数下标或者属性名
  // metadata 对于 @inject 就是 { key: METADATA_KEY.INJECT_TAG, value: serviceIdentifier }
  Reflect.defineMetadata(metadataKey, paramsOrPropertiesMetadata, annotationTarget);

}
```

### Container
