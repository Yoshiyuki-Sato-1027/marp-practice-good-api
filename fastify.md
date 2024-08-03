---
marp: true
---

# Fastify のエラーハンドリング

## 現状の運用と問題点

### 現状のエラーハンドリング

- `errorCode`を何番目の関数か＋ステータスコードで返している

```javascript
const getOrderListService = dbMiddleware(getOrderList);
return pipe(
  start(extractParamsForOrdersList(request)),
  bypass(getOrderListService),
  async (result) =>
    match(await result)
      .with({ error: { errorCode: 1400 } }, () => {
        return reply.code(400).send({ error: "Bad Request" });
      })
      .with({ error: { errorCode: 2500 } }, () => {
        return reply.code(500).send({ error: "Internal Sever Error!" });
      })
      .with({ success: true }, ({ data }) => {
        return reply.code(200).send(data);
      })
      .exhaustive()
)();
```
