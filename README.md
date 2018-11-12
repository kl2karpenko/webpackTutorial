# Импорт HTML и CSS

Для начала в нашей папке ```./dist``` создадим небольшой файл ```index.html```.

```html
<html>
  <head>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div>Hello, world!</div>
    <script src="main.js"></script>
  </body>
</html>
```

Как видите, здесь мы импортируем ```style.css```. Давайте настроим его! Как я уже упомянула, для Webpack у нас есть только одна точка входа. Так куда же мы поставим наш css?

Создаем ```style.css``` в нашей папке ```./src```:

```css
div {
  color: red;
}
```

Не забываем включить это в js-файл:

```javascript
import "./style.css";
console.log("hello, world");
```

В Webpack создайте новое правило для файлов css:

```
// Webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader']
          })
      }
    ]
  }
}
```

В терминале запустите:

```
$ npm install extract-text-webpack-plugin --save-dev
$ npm install style-loader css-loader --save-dev
```

Необходимо использовать ExtractTextPlugin для компиляции нашего CSS. Как видите, для ```.css``` мы также добавили новое правило.

Наш webpack.config должен выглядеть так:

```
// Webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader']
          })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin({filename: 'style.css'})
  ]
};
```

Затем ваш css код должен скомпилироваться в ```./dist/style.css```.
Теперь ваш ```style.css``` должен выводиться в папку ```./dist```:


### Поддержка SCSS

Очень часто разрабатываются веб-сайты с SASS и PostCSS. Они весьма полезны. Поэтому мы сперва включим поддержку SASS. Переименуем наш ./src/style.css и создадим другую папку для хранения файлов .scss. Теперь необходимо добавить поддержку форматирования .scss.

```
$ npm install node-sass sass-loader --save-dev
```

## Шаблон HTML

Теперь давайте создадим html-шаблон файла. Добавьте файл ```index.html``` в ```./src``` с точно такой же структурой.

```html
<html>
  <head>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div>Hello, world!</div>
    <script src="main.js"></script>
  </body>
</html>
```

Чтобы использовать этот файл в качестве шаблона, нам понадобится html-плагин.

```
$ npm install html-webpack-plugin --save-dev
```

Добавьте его в ваш вэбпак-файл:

```
// Webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.scss$/,
        use: ExtractTextPlugin.extract(
          {
            fallback: 'style-loader',
            use: ['css-loader', 'sass-loader']
          })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin(
      {filename: 'style.css'}
    ),
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    })
  ]
};
```

Теперь ваш файл из ```./src/index.html``` стал шаблоном для конечного файла ```index.html```.

Чтобы убедиться, что все работает, удалите все файлы из папки ```./dist``` и саму папку. Затем запустите dev-скрипт:

```
$ rm -rf ./dist
$ npm run dev
```

Вы увидите, что появилась папка ./dist, и в ней 3 файла: index.html, style.css, script.js.