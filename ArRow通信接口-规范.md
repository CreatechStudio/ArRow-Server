> 此文档阐述了如何与 ArRow 服务器通信接口建立连接。
>

1. # 通信协议

   1. ## 流程

      所有通信均以向 `www.atatc.net` 发送 TCP 请求为基础，版本 v2.7.x 使用 2077 端口。

      建立连接后经过以下步骤初始化：

      *[] 内为明文，{} 内为密文，以下格式为「发送的消息 -> 返回的消息」*

      若加密：

      - [`y`] -> [公钥]
      - [公钥] + [`@`] + [签名] + [`@`] + {请求} -> {应答}

      若不加密：

      - [`n`] -> [`plain_text`]
      - [请求] -> [应答]

      请主动维持一问一答原则，若在得到回复前再次发送请求可能发生错误甚至导致 [IP 封禁](#ip_blocking)。

      **在所有消息末尾，必须增加三位反斜杠。**

   2. ## 公钥交换

      ArRow 完全使用 [ARSA](https://www.github.com/ATATC/ARSA/) 提供的加密加签方法。若其不支持你在使用的平台，请确保你的**公钥文本遵守 X509 标准，私钥文本遵守 PKCS8 标准**，请务必在使用前确定 ARSA 的 JAVA 版本能够导入你的公钥文本。

      在通信过程中，双方交换的公钥信息构成形如：[公钥文本] + [`#`] + [公钥长度]。
      例：

      ```
      MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJ3IXIKbiX8gqriiHUSDobohZmRD6GDSV//KOEpHz/cqA5/buR3cl2BB1t4sJPDWJQGF3fKmaIuH3lt0nygSlcMCAwEAAQ==#512
      ```

   3. ## 请求格式

      向 ArRow 服务器发起的请求通常包含 3 个元素：**命令、副词、参数**，其中副词为可选参数。它们所组成的 json 字符串即客户端向服务器发送的请求。

      1. ### 命令

         请在[《ArRow通信接口-命令》](ArRow通信接口-命令.md)相关文档中了解。

      2. ### 副词

         副词根据命令不同而不同。

      3. ### 参数

         参数是一个由许多参数组成的 json 列表，每一个参数应包含 `name` 和 `value`。
         例：

         ```json
         [{"name": "language", "value": "zh-cn"}, {"name": "...", "value": "..."}]
         ```

         支持的参数根据命令不同而不同。**不过请确保你的参数中包含"language"这一项，它将在版本 v2.8.0 后作为常规参数被所有命令支持**。

         **发送时使用 Base64 编码，以避免其内容与语法分隔符冲突**。
         例：

         ```
         W3sibmFtZSI6ICJsYW5ndWFnZSIsICJ6aC1jbiJ9LCB7Im5hbWUiOiAiLi4uIiwgInZhbHVlIjogIi4uLiJ9XQ==
         ```

      下例展示了一次完整的请求段：

      ```json
      {"command": "hello", "adv": "", "args": "W3sibmFtZSI6ICJsYW5ndWFnZSIsICJ6aC1jbiJ9LCB7Im5hbWUiOiAiLi4uIiwgInZhbHVlIjogIi4uLiJ9XQ=="}
      ```

   4. ## 应答格式

      应答格式根据命令的不同而不同。

   5. ## 例

      下例展示了一次完整的加密通信 (一问一答)：

      ```
      y
      ```

      ```
      MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzwdb53Lts5Tkc/3+7y09vZ7F5ipFXLPr4doDvA2iS4k/m/iiY1zqR8NZJQWYrl0zJLF6TOEvTdmIVqLPj4gs3VBghSbwiIRObvI/fGhM6KxwlNFcPYVmS92XrZAGYp9WDdZzRhFbTlfSPMXWTyEeOvFfJhex4yllad3FV/J9wTrMJwdhhLbXY52sv9gawoAo/hlmYvP+d77+lCcFP0kUh1feYFv0ye/AKE8Uk8WFw2nQxkwS7taEzymJhBU30HEtH0nHGXIu0B25PgHVx8U/2+2I60T7g471daj7uacEKjfDK+wHuVOhCPrjN3kZILPWRuvwLTnpl92Kq1H6225AIwIDAQAB#2048
      ```

      ```
      MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtjSoHdN3Pz7bLT427d+Cofr0tIVvxpUFY5wmUiCxcABqw3GtZSoEvDzUtLjVjSASA1Yy7w6hmKfaeni6LTTGxNrBSr5ETXRNkG/5KEwgnDrqq/5Q0tpMOyFZNdIT4BmwLtUosJsO33/f1cLhmSqza0dvEcblXfTJww3kPt1RcHGJMUHBT+P+wpAZIIX0/kW5DkdgmOAIWbFzJICJ8sc6//BUr7VI3C+1JrI/6iLK5ReMcOi0lH85lALZ0Thn8VTqmaKpgTHlo0GBdJIuvkuK41ucPXWQgnu6iEzq2kxbtZ3iXQrQ1L529/VthMUjwFtJ9JOUHw86Lt3Rir4enbVeTwIDAQAB#2048@bB5HUfZcWI4M8SK7hZgfi3f20ZruGydnKDe/yAD9dlwCRKy8ZF6dnqaT6z/N1n0Lg2QhfAdtyc5TX8Dox21z0arOS2JlLmPds5ejpnNewZnBXii4bzt0u7Ld51thfZdvJrNjiT2/gzZ5oycwx61eeOuL007KwmkMJRDidC1svtUf9DMN7t0lf7cRDsNP2OGYsyAnv2TroNEQcooBL/TjEePC48gjI55tkmbJw4glnsPzVcn7mKWQt9irPB8XS31dhSwpAWDU1Vv3LmHwsAh1ISste82G1w/tKqvggYCpxp+lLSoNtKVcb86mbEIPYE04o9aBxfqhJsqzvae+AJdVig==@tcuv+TvGMmvh1OWefGSWfaG5/yVvjVR7HYM20nWEPE0WCjZGgSgW+SKe6phX0BA3riHC/5TWw3BaZoYPobl7ddELVrPmEUpNYt7oEd62Sciow4Ua0OTJ506w65X1xwW2jXhsYeiygK3kFvtW62iVjBcYsxuxmkWPI+pELHBJphs6Yyn9clU8J4pLzfxri7cceIZrEbhwmOOFmJdrWAnkMCEo5mASAPAN3AVHoR6SddWfvRXIctxK/4zzGfvvGOKItsKItheKJ+ok2MpV1zIDEYdmyFUWeYcFa3Sjyk4GtgeBg6RqErPgD2GI1ExyLfHh6jizgPp/JgeM7tok5x2v4A==
      ```

      ```
      zXiWr+oAvJcj59VhY9WHfKosL6b0daDqrydzCo7Q47cJ9r34En+aBrYNvSp/2Umb56NsN+q74hKnsv1W7bCh/G/I++SypEUUA3jNLO44uDELK5NfyHv3WhIEYe2AXXyJgIZ0HZJwi8z6EIIHkbn7scCE6FvK167b5NThiOn56GVxMu4yOlKGcPef7vpXmhySH40jHxKazrzgKB54pyKEKbJdLg+AXhTEkpbEKhrTNIMJrYxnkF7zSC2khzgG6A9kpQGRYNK+fu1Yd6T8mjwTQiyqs6iWAnF0UOhVx05MijpG8SUHPymVTUDwlN+k3JNH3VcHQhmCdxoizFJwKLSubQ==@ZWwrQdY0uxkLLoCzq5oq3Ias3yDJoV85jY093uLHZjON4skuYTAeIiAgU/WvYBYRPHganuXE6EUNtoKzoVnZzqEGdxrMs8L9M4dTvlrhaPN6/l/4dVRG35vJOMp+TEe4YsclFbufNGMrcGPknxgvtS/cvIDOUshqOGXJbJE7pMXfWJXWHqQs59qjWrXToYov1WoWSekD4eU7v1ThhakO0b7NeaR3ZXPqo30RbwL3gRsDw5sz6Y5eBunCuEZoSWVKSdE3yWN2CJ2dVFWEeh2B39LXTnrDMExvZrL4SLxGXUhMDaFPl2tQKwwVS2VIxJAOe0xTDEjSnMvwiC0VWYP6FA==
      ```

      下例展示了一次完整的明文通信 (一问一答)：

      ```
      n
      ```

      ```
      plain_text
      ```

      ```json
      {"command": "hello", "adv": "", "args": "W3sibmFtZSI6ICJsYW5ndWFnZSIsICJ6aC1jbiJ9LCB7Im5hbWUiOiAiLi4uIiwgInZhbHVlIjogIi4uLiJ9XQ=="}
      ```

      ```
      hello
      ```

2. # IP 封禁

   为防止爬虫占用资源或模拟客户端恶意攻击，重复以下行为**可能**引起 IP 封禁：

   - 流程完成前结束连接。
   - 发送空消息。
   - 不遵守流程。
   - 公钥无法解析。
   - 请求格式错误。

   被封禁后，你发起的通信将被直接关闭，无提前报备**无法恢复**。
