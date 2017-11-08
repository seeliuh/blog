# http文件服务 实现文件服务端

项目上需要在http server实现文件服务，接收到文件保存，并返回url。我使用crow实现http server，crow中并没有对接收文件有现成的解析，只能自己实现。

rfc1867规定了http上传文件的规范。引用资料中也有对格式的描述。格式如下：

```

--$boundary
some date
--$boundary
some data
--$boundary-- (结束多了--)

```

下面是自己对于解析方法的描述。

在header中查找Content-Type。格式为： ```Content-Type: multipart/form-data; boundary=${bound} ```boundary是数据块的分隔符，解析出来备用。

在body中，通过boundary将各数据块分割出来。

在各数据块中，查找filename，有filename的数据块才包含文件数据

在有filename的数据块中，\r\n\r\n后面的就是二进制文件数据的开始。 下一个boundary的位置减5（略过"\r\n--"）就是二进制文件数据的结束位置。

这种方式，可以更好的控制客户端上传的文件。受益匪浅。

以下是核心代码

```

class FormDataBlock {
  public:
    FormDataBlock() {
        pFileDataPos = 0;
        nFileDataLen = 0;
    }
    ~FormDataBlock() {}
    bool Parse(char *pBegin, char *pEnd) {
        //parse filename
        char *pFileNameBegin = mystrstr(pBegin, "filename", pEnd);
        if(pFileNameBegin) {
            pFileNameBegin = mystrstr(pFileNameBegin, "\"", pEnd);
            if(pFileNameBegin) {
                pFileNameBegin++;
                char *pFileNameEnd = mystrstr(pFileNameBegin, "\"", pEnd);
                if(pFileNameEnd) {
                    pFileNameEnd--;
                    sFileName.insert(0, pFileNameBegin, (pFileNameEnd - pFileNameBegin) + 1);
                } else {
                    return false;
                }
            } else {
                return false;
            }
        } else {
            return false;
        }
        //parse filedata
        pFileDataPos = mystrstr(pBegin, "\r\n\r\n", pEnd);
        if(pFileDataPos) {
            pFileDataPos += 4;
            nFileDataLen = pEnd - pFileDataPos;
        } else {
            return false;
        }
        return true;
    }
    bool SaveFile(const std::string &sBaseFilePath) {
        std::string sFullName = sBaseFilePath + "/" + sFileName;
        FILE *fp = fopen(sFullName.c_str(), "wb");
        if(!fp) {
            return false;
        }
        int nWrite = 0;
        while(1) {
            int nTmpWrite = fwrite(pFileDataPos + nWrite, 1, nFileDataLen - nWrite, fp);
            if(nTmpWrite >= nFileDataLen) {
                break;
            } else {
                nWrite += nTmpWrite;
            }
        }
        fclose(fp);
        return true;
    }
    std::string sFileName;
    char *pFileDataPos;
    int nFileDataLen;
};
class FormData {
  public:
    FormData() {}
    ~FormData() {}
    bool Parse(const std::string &body, const std::string &sBoundary) {
        bool ret  = false;
        const char *pBoundary = sBoundary.c_str();
        char *pBytes = (char *)body.data();
        int nBytesLen = body.size();
        char *pEndFlag = pBytes + (nBytesLen - 1);
        char *pBegin = pBytes;
        while(1) {
            pBegin = mystrstr(pBegin, pBoundary, pEndFlag);
            if(pBegin) {
                pBegin = mystrstr(pBegin, "\r\n", pEndFlag);
                if(pBegin) {
                    pBegin += 2;
                    char *pEnd = mystrstr(pBegin, pBoundary, pEndFlag);
                    if(pEnd) {
                        pEnd -= 5;
                        FormDataBlock block;
                        if(block.Parse(pBegin, pEnd)) {
                            lstFormBlock.push_back(block);
                            ret = true;
                        }
                    }
                } else {
                    break;
                }
            } else {
                break;
            }
        }
        return ret;
    }
    std::list<FormDataBlock > lstFormBlock;
};

```

## c++ string做buffer的注意事项

在crow中，包含二进制文件内容的body保存在string中。crow使用std::string::insert语句将tcp层接收到的网络数据赋值到body中。insert方法避免了二进制中\0的干扰，完整的将二进制数据存储在string中。我们在处理body时，需要格外注意，不能用字符串相关的函数，比如find一套，substr等等。

我使用string::data()函数，得到buffer指针，然后通过指针和定制化的strstr函数解析二进制+文本混合在一起的数据。

## 定制化strstr函数

``` 

char *mystrstr(char *s1, const char *s2, char *end) {
    int n;
    if (*s2) {
        while(s1 < end) {
            for (n = 0; * (s1 + n) == *(s2 + n); n++) {
                if (!*(s2 + n + 1)) {
                    return (char *)s1;
                }
            }
            s1++;
        }
        return NULL;
    } else {
        return (char *)s1;
    }
}
```

比官方strstr函数多了参数end，作为buffer的结束符。官方strstr会将\0作为结束符。用这个函数，可以在二进制+文本混合在一起的数据中搜索指定字符串。

这个函数在解析时非常有用

## 后话

之前就很好奇http传输文件是什么方式，因为我的认知http只能传输文本，传文件的话可能需要base64编码，担心用http的方式传输文件效率会很低。现在看来，效率不会差太多。接下来会对效率进行一个测试，与tcp层传二进制文件进行效率对比。

## 引用文档 参考资料
[HTTP协议之multipart/form-data请求分析](http://blog.csdn.net/five3/article/details/7181521)
