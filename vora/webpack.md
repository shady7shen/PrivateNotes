# Webpack
(https://webpack.js.org/guides/asset-management/#loading-css)
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
eyJoaXN0b3J5IjpbLTY5NDI5NjI4NCwtMTk4MTI1NTQ2MCwxNT
gzNzgxMDQwXX0=
-->