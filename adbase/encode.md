# Encode 相关

### 加密相关

加密实现了 sha1 算法和 hmacSha1 算法，使用方法如下：

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {
    const std::string input = "dGhlIHNhbXBsZSBub25jZQ==258EAFA5-E914-47DA-95CA-C5AB0DC85B11";
    adbase::Sha1 checksum;
    checksum.update(input);
    char hash[20] = {0};
    checksum.final(hash);
    LOG_INFO << "The SHA-1 of \"" << input << "\" is: " << adbase::base64Encode(hash, 20);

    std::string key = "86ae80bd15656981816e5ea3bda86c3e";
    char out[20] = {0};
    adbase::Sha1::hmacSha1(key.c_str(), key.size(), input.c_str(), input.size(), out);
    LOG_INFO << adbase::base64Encode(out, 20);
    return 0;
}
```

### 编码相关

###### Base64

Adbase 提供 base64 转化算法工具 `adbase::base64Encode` 和 `adbase::base64Decode`

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {

    /// base64 encode/decode example
    std::string base64 = "test base64";
    std::string base64encode = adbase::base64Encode(base64.c_str(), base64.size());
    LOG_INFO << base64encode;
    char base64decode[base64encode.size()];
    memset(base64decode, 0, base64encode.size());
    int length = 0;
    adbase::base64Decode(base64decode, &length, base64encode);
    LOG_INFO << base64decode;
    return 0;
}
```

###### Base62

Adbase 提供 base62 转化算法工具 `adbase::base62Encode` 和 `adbase::base62Decode`

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {
    adbase::Sequence seq;
    uint64_t seqId = seq.getSeqId(3, 3);
    std::string seqIdStr = adbase::base62Encode(seqId);
    LOG_INFO << seqId;
    LOG_INFO << seqIdStr;
    LOG_INFO << adbase::base62Decode(seqIdStr);
    return 0;
}
```

###### UrlEncode

Adbase 提供 url 编码转化算法工具 `adbase::urlEncode` 和 `adbase::urlDecode`

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {
    /// url encode/decode example
    std::string url = "http://www.weibo.com?test=1&test??????";
    std::string encode = adbase::urlEncode(url);
    LOG_INFO << encode;
    LOG_INFO << adbase::urlDecode(encode);
    return 0;
}
```

### 进制转化

adbase 提供了十六进制字符串与bytes 相互转化的方法： `adbase::bytesToHexString`、`adbase::hexStringToBytes`

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main() {
    std::string hexStr = "efabca";
    char buffer[3] = {0};
    adbase::hexStringToBytes(hexStr, buffer);

    std::string result = adbase::bytesToHexString(buffer, 3);
    LOG_INFO << result;
    return 0;
}
```
