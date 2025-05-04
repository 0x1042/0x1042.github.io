# vscode 

## terminal 打开报错 

```
SecCodeCheckValidity: Error Domain=NSOSStatusErrorDomain Code=-67062 "(null)" (-67062)
```

## 修复 

```
xattr -cr  /Applications/Visual\ Studio\ Code.app
sudo codesign --force --deep --sign - /Applications/Visual\ Studio\ Code.app
``` 

# apt 

## install warning

```
Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

## 修复

[askubuntu.com](https://askubuntu.com/questions/1398344/apt-key-deprecation-warning-when-updating-system-key-is-stored-in-legacy-trust)

```
cd /etc/apt
sudo cp trusted.gpg trusted.gpg.d
```