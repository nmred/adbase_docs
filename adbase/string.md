# 字符串相关

### Trim

`trim` 系列操作方法默认情况下会将 `空格\f\n\r\t\v` 等字符去掉，也支持定制化 trim 某些字符, 通过第二个参数传递

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    std::string str = "\n\t\vtest\n   ";

    LOG_INFO << "string len:" << str.size();
    LOG_INFO << "left trim after len: " << adbase::leftTrim(str).size();
    LOG_INFO << "right trim after len: " << adbase::rightTrim(str).size();
    LOG_INFO << "all trim after len: " << adbase::trim(str).size();

    std::string trimChar = "\n\v\t t";
    LOG_INFO << "custom left trim after len: " << adbase::leftTrim(str, trimChar.c_str()).size();
    LOG_INFO << "custom right trim after len: " << adbase::rightTrim(str, trimChar.c_str()).size();
    LOG_INFO << "custom all trim after len: " << adbase::trim(str, trimChar.c_str()).size();
    return 0;
}
```

### Replace

字符串替换

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    std::string subject = "aabbccaabbccbbccaa";
    int count = 0;

    LOG_INFO << adbase::replace("aa", "AA", subject, count);
    LOG_INFO << "replace count:" << count;
    return 0;
}
```

### Explode

第一个参数是要分隔的字符串，第二个参数是分隔的字符，第三个参数是是否要 `trim` 分隔出来的结果

```cpp
#include <adbase/Utility.hpp>
#include <adbase/Logging.hpp>

int main(void) {
    std::string subject = "a ab a  ba";

    std::vector<std::string> result = adbase::explode(subject, 'a', false);
    LOG_INFO << "result count:" << result.size();
    for(auto &t : result) {
        LOG_INFO << "char:" << t << " |size|:" << t.size();
    }

    std::vector<std::string> trimResult = adbase::explode(subject, 'a', true);
    LOG_INFO << "trim result count:" << trimResult.size();
    for(auto &t : trimResult) {
        LOG_INFO << "trim char:" << t << " |size|:" << t.size();
    }
    return 0;
}
```
