# 通过Block Store实现账户无缝迁移

在用户迁移到新设备后自动同步各个应用的账户设置。

### 什么是Block Store

Block Store API可以让应用存储用户凭据，从而可在未来的新设备中取回凭据，并用于从新验证用户。当用户使用一台设备引导到另一台设备是，凭据数据就会在设备间传输。

### Block Store 的工作原理

- 当用户登录应用时（或是在此之后的任何时间），可以将用户的身份认证令牌保存至Block Store。
- 使用Block Store保存令牌之后，令牌会被加密并保存在设备的本地存储中。
- 当用户使用“设备到设备”的恢复流程时，数据会被传输到新设备上。
- 如果用户在“设备到设备”的灰度是选择同时恢复他们的数据，当用户在新设备上打开应用时，Block Store会为应用取回令牌。

### 如何使用

当用户登录您的应用时，您可以通过调用 `storeBytes() `将您为用户生成的身验认证令牌存储至 Block Store。这一操作会将用户的凭据存储到源设备。现在，令牌已被加密并保存到了设备的本地存储中。

```kotlin
val client = Blockstore.getClient(this)
val token = StoreBytesData.Builder()
              .setBytes(/* BYTE_ARRAY */)
              .build()

// storeBytes 应当在用户验证完成之后再调用
client.storeBytes(token)).addOnSuccessListener{ result -> 
  Log.d(TAG, “Stored: ${result} bytes”) 
}
```

当用户在新设备上完成 "设备到设备" 的恢复流程时，Block Store 会取回您的令牌。由于用户已经同意在恢复流程中恢复您应用的数据，所以此操作无需额外的许可。当用户打开您的应用时，您可以通过调用 `retrieveBytes()` 从 Block Store 请求您的令牌。取得的令牌可以用于在新设备上保持用户的登录状态。如果调用此接口的应用没有令牌，Block Store 依然会调用`` onSuccessListener()``，但结果会是空字节。如果您在同一个设备上先后调用 `storeBytes()` 和 `retrieveBytes()`，`retrieveBytes() `会返回先前调用的 storeBytes() 设置的字节。

```kotlin
val client = Blockstore.getClient(this)
client.retrieveBytes()
    .addOnSuccessListener { result ->
        Log.d(TAG, “Retrieved: ${String(result)}”)
    }

    .addOnFailureListener { e ->
      Log.e(TAG, “Failed to retrieve bytes”, e)
    }
```

