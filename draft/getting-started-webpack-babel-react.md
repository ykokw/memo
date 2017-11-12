# webpack / babel / react のセットアップ

- [元記事](https://stanko.github.io/webpack-babel-react-revisited/)
- yarnでいけるか試す

## Webpack

- moduleのビルド用
  - transpile / bundle js , compile sass / postCss

```
yarn add -D webpack
```

- `src/js/app.js` を作成してwebpack を使う
  - `dis/app.bundle.js` が作成される

```
./node_modules/webpack/bin/webpack.js ./src/js/app.js --output-filename ./dist/app.bundle.js
```

- 上のパラメータはwebpack.config.js で設定できる
  - コマンドもwebpack.js の実行だけでよくなる
  - entry（元ネタ）と output（出力先

```js
const path = require('path');

const paths = {
  DIST: path.resolve(__dirname, 'dist'),
  JS: path.resolve(__dirname, 'src/js'),
};

module.exports = {
  entry: path.join(paths.JS, 'app.js'),
  output: {
    path: paths.DIST,
    filename: 'app.bundle.js'
  },
};
```

- package.json にbuild scriptを追加
  - `yarn build` だけでよくなる
    - node_modules内のバイナリファイルはyarn (npm) が探してくれる

```
  "scripts": {
    "build": "webpack"
  }
```

## Webpack dev server

- ブラウザでアプリケーションを開いて動作確認するにはサーバーが必要
  - webpack dev server で開発中のアプリケーションをブラウザで動作確認できる

- install

```
yarn add webpack-dev-server
```

- package.json にdevコマンド追加

```json
{
  "name": "webpack-babel-react-revisited",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "devDependencies": {
    "webpack": "^3.8.1",
    "webpack-dev-server": "^2.9.4"
  },
  "scripts": {
    "dev": "webpack-dev-server",
    "build": "webpack"
  }
}

```

- `yarn dev` のあと、 `http://localhost:8080` にアクセスできるようになる
- ベースになるHTMLを作成 ( `src/index.html` )

```html

<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width">

    <title>Webpack Babel React revisited</title>
  </head>
  <body>
  </body>
</html>
```

- webpack.config.js にcontent baseを設定

```js
const path = require('path');

const paths = {
  DIST: path.resolve(__dirname, 'dist'),
  SRC: path.resolve(__dirname, 'src'), // content base用のpathを追加
  JS: path.resolve(__dirname, 'src/js'),
};

module.exports = {
  entry: path.join(paths.JS, 'app.js'),
  output: {
    path: paths.DIST,
    filename: 'app.bundle.js'
  },
  devServer: {
    contentBase: paths.SRC, // dev server のcontent base設定
  },
};
```

- この状態で `yarn dev` すると、jsを読み込んでないので空ページが表示される

- html-webpack-plugin をインストール

```
yarn add -D html-webpack-plugin
```

- webpack.config.js にプラグインの設定を追加

```js
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(paths.SRC, 'index.html'),
    }),
  ],
```

- 全体

```js
const path = require('path');

// プラグインをrequire
const HtmlWebpackPlugin = require('html-webpack-plugin');

const paths = {
  DIST: path.resolve(__dirname, 'dist'),
  SRC: path.resolve(__dirname, 'src'),
  JS: path.resolve(__dirname, 'src/js'),
};

module.exports = {
  entry: path.join(paths.JS, 'app.js'),
  output: {
    path: paths.DIST,
    filename: 'app.bundle.js'
  },
  devServer: {
    contentBase: paths.SRC,
  },
  // プラグインの設定
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(paths.SRC, 'index.html'),
    }),
  ],
};
```

- `yarn dev` でconsoleにメッセージが出力される
  - ファイルを変更したら自動で再読込してくれる

## Babel

- ES2015以上、モダンなJSでコードを書いたらtranspilerが必要
  - モダンなJSを、ブラウザで実行できるJSに変換してくれる
- Reactがbabelと深く結びついてる
- 良いコードを書くためにも

- 必要なパッケージが4つ
  - Babel core package
  - Babel webpack loader
  - Babel env preset
  - Babel React preset
- まとめてinstall

```
yarn add -D babel-core babel-loader babel-preset-env babel-preset-react
```

- babel用の設定ファイル `.babelrc` をrootに作成
  - envとreact用のpresetを使用する

```json
{
  "presets": ["env", "react"]
}
```

- webpack.config.jsの更新
  - 拡張子が `js` or `jsx` のファイルを読み込んでbabel-loaderを使う設定

```js
const path = require('path');

const HtmlWebpackPlugin = require('html-webpack-plugin');

const paths = {
  DIST: path.resolve(__dirname, 'dist'),
  SRC: path.resolve(__dirname, 'src'),
  JS: path.resolve(__dirname, 'src/js'),
};

module.exports = {
  entry: path.join(paths.JS, 'app.js'),
  output: {
    path: paths.DIST,
    filename: 'app.bundle.js'
  },
  devServer: {
    contentBase: paths.SRC,
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(paths.SRC, 'index.html'),
    }),
  ],
  // js or jsxのファイルでbabel-loaderを使う設定
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: [
          'babel-loader',
        ],
      },
    ],
  },
  // 拡張子を指定しなくてもimportできるようになる設定
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

## React

- react / react-dom をインストール

```
yarn add react react-dom
```

- index.html にdiv要素追加

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width">

    <title>Webpack Babel React revisited</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

- app.js を編集

```js
import React, { Component } from 'react';
import { render } from 'react-dom';

export default class Hello extends Component {
  render() {
    return (
      <div>
        Hello from react
      </div>
    );
  }
}

render(<Hello />, document.getElementById('app'));
```

- `yarn dev` すると、ブラウザにメッセージが表示される
