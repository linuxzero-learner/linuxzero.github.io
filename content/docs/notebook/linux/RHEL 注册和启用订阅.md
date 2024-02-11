## 注册

```bash
subscription-manager register
subscription-manager register --auto-attach
```

## 查询订阅

```bash
subscription-manager list --available
```

## 启用订阅

```bash
subscription-manager attach --pool <poolid>
subscription-manager attach --auto
```

## 取消订阅

```bash
subscription-manager remove --pool <poolid>
subscription-manager remove --all
```

## 取消注册

```bash
subscription-manager unregister
```

## 删除订阅数据

```bash
subscription-manager clean
```

