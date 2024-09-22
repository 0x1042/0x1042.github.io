# terminal 打开报错 

```
SecCodeCheckValidity: Error Domain=NSOSStatusErrorDomain Code=-67062 "(null)" (-67062)
```

## 修复 

```
xattr -cr  /Applications/Visual\ Studio\ Code.app
sudo codesign --force --deep --sign - /Applications/Visual\ Studio\ Code.app
``` 
