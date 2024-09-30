mkdir my-frontend-project
cd my-frontend-project

npm init -y

npm install react react-dom
npm install --save-dev webpack webpack-cli babel-loader @babel/core @babel/preset-env @babel/preset-react
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000,
  },
};

{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}


import React from 'react';
import ReactDOM from 'react-dom';

const App = () => <h1>Hello, world!</h1>;

ReactDOM.render(<App />, document.getElementById('root'));

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Frontend Project</title>
</head>
<body>
  <div id="root"></div>
  <script src="bundle.js"></script>
</body>
</html>

npm start

# Frontend Setup

## Prerequisites
- Node.js and npm
- Code editor (e.g., VS Code)

## Steps

1. **Set Up Your Development Environment**
   - Install Node.js and npm from Node.js website.
   - Install VS Code.

2. **Initialize Your Project**
   - Create a new directory and initialize npm.

3. **Install Dependencies**
   - Install React, Webpack, and Babel.

4. **Configure Webpack**
   - Create `webpack.config.js`.

5. **Configure Babel**
   - Create `.babelrc`.

6. **Create Source Files**
   - Create `src/index.js`.

7. **Set Up HTML Template**
   - Create `dist/index.html`.

8. **Update `package.json` Scripts**
   - Add start and build scripts.

9. **Run Your Project**
   - Start the development server and open `http://localhost:9000`.

const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.html$/,
        use: ['html-loader'],
      },
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        loader: 'file-loader',
        options: {
          name: '[name]-[hash].[ext]',
          outputPath: 'images',
        },
      },
    ],
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: 'index.html',
    }),
  ],
};
