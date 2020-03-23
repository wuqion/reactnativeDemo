# 添加react-native-web控件

本人ReactNative小白，简单备注。
ps:不整理App.js文件会有各种未知问题官方也没有说明，比较头疼

ReactNative 0.61.5

倒入插件
```
yarn add react-dom react-native-web

yarn add --dev babel-loader url-loader webpack webpack-cli webpack-dev-server

yarn add --dev babel-plugin-react-native-web

yarn add @babel/preset-env

这个不要用yarn，不然版本下不对
npm i babel-preset-react-native@5
```

新建文件 web/webpack.config.js
```
// web/webpack.config.js

const path = require('path');
const webpack = require('webpack');

const appDirectory = path.resolve(__dirname, '../');

// This is needed for webpack to compile JavaScript.
// Many OSS React Native packages are not compiled to ES5 before being
// published. If you depend on uncompiled packages they may cause webpack build
// errors. To fix this webpack can be configured to compile to the necessary
// `node_module`.
const babelLoaderConfiguration = {
  test: /\.js$/,
  // Add every directory that needs to be compiled by Babel during the build.
  include: [
    path.resolve(appDirectory, 'index.web.js'),
    path.resolve(appDirectory, 'src'),
    path.resolve(appDirectory, 'node_modules/react-native-uncompiled')
  ],
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true,
      // The 'react-native' preset is recommended to match React Native's packager
      presets: ['react-native','@babel/preset-env'],
      // Re-write paths to import only the modules needed by the app
      plugins: ['react-native-web']
    }
  }
};

// This is needed for webpack to import static images in JavaScript files.
const imageLoaderConfiguration = {
  test: /\.(gif|jpe?g|png|svg)$/,
  use: {
    loader: 'url-loader',
    options: {
      name: '[name].[ext]'
    }
  }
};

module.exports = {
  entry: [
    // load any web API polyfills
    // path.resolve(appDirectory, 'polyfills-web.js'),
    // your web-specific entry file
    path.resolve(appDirectory, 'index.web.js')
  ],

  // configures where the build ends up
  output: {
    filename: 'bundle.web.js',
    path: path.resolve(appDirectory, 'dist')
  },

  // ...the rest of your config

  module: {
    rules: [
      babelLoaderConfiguration,
      imageLoaderConfiguration
    ]
  },

  resolve: {
    // This will only alias the exact import "react-native"
    alias: {
      'react-native$': 'react-native-web'
    },
    // If you're working on a multi-platform React Native app, web-specific
    // module implementations should be written in files using the extension
    // `.web.js`.
    extensions: [ '.web.js', '.js' ]
  }
}

```
建立文件建src
将App.js移动到src
修改index.js里面的路径
```
import App from './App’;
改为
import App from './src/App’;
```
新建index.web.js
```
/**
 * @format
 */

import {AppRegistry} from 'react-native';
import App from './src/App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);

AppRegistry.runApplication(appName,{
    rootTag: document.getElementById('root')
})
```
修正App.js文件
```

import React from 'react';
import {
  View,
  Text,
} from 'react-native';

const App: () => React$Node = () => {
  return (
    <View>
      <Text>ddd</Text>
    </View>
  );
};


export default App;
```
新建文件index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
    <script src="bundle.web.js"></script>
</body>
</html>
```
执行命令测试
```
./node_modules/.bin/webpack-dev-server -d --config ./web/webpack.config.js --inline --hot --colors

```
执行命令打包
```
./node_modules/.bin/webpack -p --config ./web/webpack.config.js
```

建立好的package.json大概这样
```
{
  "name": "demo",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "start": "react-native start",
    "test": "jest",
    "lint": "eslint ."
  },
  "dependencies": {
    "babel-preset-react-native": "^5.0.2",
    "react": "16.9.0",
    "react-dom": "^16.13.1",
    "react-native": "0.61.5",
    "react-native-web": "^0.12.2"
  },
  "devDependencies": {
    "@babel/core": "^7.6.2",
    "@babel/runtime": "^7.6.2",
    "@react-native-community/eslint-config": "^0.0.5",
    "babel-jest": "^24.9.0",
    "babel-loader": "^8.1.0",
    "babel-plugin-react-native-web": "^0.12.2",
    "eslint": "^6.5.1",
    "jest": "^24.9.0",
    "metro-react-native-babel-preset": "^0.56.0",
    "react-test-renderer": "16.9.0",
    "url-loader": "^4.0.0",
    "webpack": "^4.42.0",
    "webpack-cli": "^3.3.11",
    "webpack-dev-server": "^3.10.3"
  },
  "jest": {
    "preset": "react-native"
  }
}
```
react-native-web 官方地址
http://necolas.github.io/react-native-web/docs/?path=/docs/guides-multi-platform--page

注意要在webpack.config.js加入@babel/preset-env 如下面这样，官方没有说明
const babelLoaderConfiguration = {
  test: /\.js$/,
  // Add every directory that needs to be compiled by Babel during the build.
  include: [
    path.resolve(appDirectory, 'index.web.js'),
    path.resolve(appDirectory, 'src'),
    path.resolve(appDirectory, 'node_modules/react-native-uncompiled')
  ],
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true,
      // The 'react-native' preset is recommended to match React Native's packager
      presets: ['react-native','@babel/preset-env'],
      // Re-write paths to import only the modules needed by the app
      plugins: ['react-native-web']
    }
  }
};v
