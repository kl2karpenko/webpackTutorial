### Транспиляция вашего кода .js

Современный JS-код в основном написан на ES6, а ES6 поддерживается не всеми браузерами. Поэтому вам необходимо транспилировать его. Транспиляция — волшебное слово для перевода ES6 в ES5. Для этого можно использовать babel — наиболее популярный инструмент для транспиляции.

```
$ npm install babel-core babel-loader babel-preset-env --save-dev
```

Далее нужно создать файл конфигурации для babel - .babelrc в корне папки:

```
{
"presets": [
   "env"
   ]
}
```

Теперь у нас есть два варианта конфигурации babel-загрузчика:

Теперь у нас есть два варианта конфигурации babel-загрузчика:

Использовать конфигурационный файл webpack.config.js.
Использовать --module-bind в npm-скриптах.
Технически вы многое можете сделать с новыми флагами, представленными в Webpack, однако для простоты я бы предпочла webpack.config.js.

### Конфигурационный файл

На этом этапе мы создадим webpack.config.js со следующим содержимым:

```
// Webpack v4
const path = require('path');
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
      }
    ]
  }
}
```

Также теперь мы удалим флаги из наших npm-скриптов:

```
"scripts": {
  "build": "webpack --mode production",
  "dev": "webpack --mode development"
}
```

Теперь, когда мы запускаем npm run build, он должен вывести нам небольшой js-файл в ./dist/main.js. Если это не происходит, попробуйте переустановить babel-загрузчик.
