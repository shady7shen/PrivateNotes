[Web Pack](#webpack)
	[Installation](#Installation)


# Webpack
[documentation](https://webpack.js.org/guides/asset-management/#loading-css)
## Installation

    npm install --save-dev webpack webpack-cli

## Basics
Using configuration file
**webpack.config.js**
```
const path = require('path');
module.export = {
	entry : './src/index.js',
	output : {
		filename : 'main.js',
		path: path.resolve(__dirname, 'dist')
	}
}
```
## Asset Management
### Loading CSS
```
npm instal --save-dev style-loader css-loader
```
add in **webpackage.config.js**
```
module: {
	rules: [
		{
			test : /\.css$/,
			use : [	'style-loader', 
					'css-loader'
				]
		}
	]
}
```
 ### Loading Images
 ```
 npm i -D file-loader
 ```
 add in **webpack.config.js**
 ```
 {
   test : /\.(jpg|jpeg|png|gif|svg)$/,
   use: [ 'file-loader']
 }
 ```
 same loader can be used to load fonts as well
 ```
 test: /\.(woff|woff2|eot|ttf|otf)$/
 ```
 ### loading Data
####  CSV
 ```
 npm i -D csv-loader papaparser
 ```
 ```
 {
	 test : /\.csv$/,
	 loader: 'csv-loader',
	 options : {    // more options in papaparser documentation
	   dynamicTyping: true,
	   header: true,
	   skipEmptyLines: true
	 }
```
after which one can import csv as json objects as,
```
import data from './a.csv';
```

#### XML
```
npm i -D xml-loader
```
```
{
	test : /\.xml$/,
	loader : 'xml-loader',
	options : {
	  explicitArray: false
	}
}
```	 
## Development
### development mode
```
mode : 'development'
```
### Source Map
```
devtool : 'inline-source-map'
```
## Webpack dev server
```
npm install --save-dev webpack-dev-server
```
add in **webpackage.config.js**
```
...
const webpack = require('webpack');  //required for hot module replacement

module.exports = {
...
  devServer: {
    public: 'localhost:30080',   //the url for the browser to allow NAT or reverse proxy
    contentBase: path.resolve(__dirname, 'dist'),
    disableHostCheck: true,
    host: '0.0.0.0',
    port: 8080,
    hot: true   // required for hot module replacement
  }
  ,plugins: [
    new webpack.HotModuleReplacementPlugin()  //required for hot module replacement
  ]
};

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE1MjMwNTQ2MywtNjk0Mjk2Mjg0LC0xOT
gxMjU1NDYwLDE1ODM3ODEwNDBdfQ==
-->