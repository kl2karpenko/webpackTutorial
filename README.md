# Кэширование и хеширование

Одна из наиболее распространенных проблем в разработке — реализация кэширования. Очень важно понять, как это работает, ведь вы же хотите, чтобы у ваших пользователей всегда была последняя версия кода.

Одним из самых популярных способов решения проблем кеширования является добавление хеш-номера в активные (находящиеся в разработке) файлы (asset), такие как style.css и script.js.
Хеширование необходимо, чтобы браузер «научился» запрашивать только измененные файлы.

Webpack 4 имеет встроенные функции, реализованные через chunkhash. Это можно сделать следующим образом:

```javascript
// Webpack v4
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js'
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
      {filename: 'style.[chunkhash].css', disable: false, allChunks: true}
    ),
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    }),
  ]
};
```

В файл ```./src/index.html``` добавьте:

```html
<html>
  <head>
    <link rel="stylesheet" href="<%=htmlWebpackPlugin.files.chunks.main.css %>">
  </head>
  <body>
    <div>Hello, world!</div>
    <script src="<%= htmlWebpackPlugin.files.chunks.main.entry %>"></script>
  </body>
</html>
```

С помощью такого синтаксиса ваш шаблон «научится» использовать хешированные файлы. Это новая фича, реализованная после возникновения этой проблемы.

Мы будем использовать описанный там ```htmlWebpackPlugin.files.chunks.main```.

Теперь в нашей папке ```./dist``` есть файл ```index.html```:

Далее, если мы ничего не изменим в файлах .js и .css и запустим:

```
npm run dev
```

То увидим, что, независимо от количества запусков, числа в хешах должны быть идентичны в обоих файлах.

### Интегрирование PostCSS


Чтобы выходной css-файл был безупречным, мы можем добавить наверх PostCSS.

PostCSS предоставляет вам autoprefixer, cssnano и другие приятные и удобные штуковины. Буду продвигать то, что использую регулярно. Нам понадобится postcss-загрузчик. Также по мере необходимости установим autoprefixer.

```
npm install postcss-loader --save-dev
npm i -D autoprefixer
```

Создайте ```postcss.config.js```, где вам требуюся соответствующие плагины, вставьте:

```
module.exports = {
    plugins: [
      require('autoprefixer')
    ]
}
```

Теперь наш ```webpack.config.js``` должен выглядеть так:

```
// Webpack v4
const path = require('path');
// const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WebpackMd5Hash = require('webpack-md5-hash');
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  entry: { main: './src/index.js' },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[chunkhash].js'
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
        use:  [  'style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin('dist', {} ),
    // new ExtractTextPlugin(
    //   {filename: 'style.[hash].css', disable: false, allChunks: true }
    // ),
    new MiniCssExtractPlugin({
      filename: 'style.[contenthash].css',
    }),
    new HtmlWebpackPlugin({
      inject: false,
      hash: true,
      template: './src/index.html',
      filename: 'index.html'
    }),
    new WebpackMd5Hash()
  ]
};
```


Пожалуйста, обратите внимание на порядок плагинов, которые мы используем для ```.scss```

```
use: [ 'style-loader', MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']
```