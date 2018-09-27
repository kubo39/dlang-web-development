# メモリ

## GCチューニング

[このページ](https://dlang.org/spec/garbage.html#gc_config) を参照

## 実行時

- `core.memory.GC.Stats` でGCヒープ内のメモリ使用量と空き容量を確認できる
- 下のようなハンドラを用意しておく
  - プロダクション環境では認証などしておきユーザに見えないように注意

```d
void getMemstat(HTTPServerRequest req, HTTPServerResponse res)
{
    import core.memory;
    size_t usedSize = GC.stats.usedSize;
    size_t freeSize = GC.stats.freeSize;
    Json[string] obj;
    obj["usedSize"] = usedSize;
    obj["freeSize"] = freeSize;
    res.writeJsonBody(obj);
}
```
