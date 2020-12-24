# react-native-code-push

用于app端接入热更新功能；

源码地址：https://github.com/microsoft/react-native-code-push

**经测试目前可用的最新版本 react-native-code-push@6.4.1**

### 一、配置私有部署服务器：
iOS：
info.plist添加
```
<key>CodePushServerURL</key>
<string>https://yourcodepush.server.com</string>
```

android:
strings.xml添加
```
<string moduleConfig="true" name="CodePushServerURL">https://yourcodepush.server.com</string>
```

参考地址：https://github.com/microsoft/react-native-code-push/blob/master/docs/api-android.md#api-for-react-native-060-version-and-above

### 二、接入方法
见文档
https://github.com/microsoft/react-native-code-push

### 三、使用方式
见文档
https://github.com/microsoft/react-native-code-push