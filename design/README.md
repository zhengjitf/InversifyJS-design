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
├── container # 容器，我们需要在容器中声明 bingding（创建绑定关系），获取实例
│   ├── container.ts
│   ├── container_module.ts
│   ├── container_snapshot.ts
│   ├── lookup.ts
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
├── syntax
│   ├── binding_in_syntax.ts
│   ├── binding_in_when_on_syntax.ts
│   ├── binding_on_syntax.ts
│   ├── binding_to_syntax.ts
│   ├── binding_when_on_syntax.ts
│   ├── binding_when_syntax.ts
│   └── constraint_helpers.ts
```