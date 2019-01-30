# Webpack
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
devServer: {
  contentBase: path.resolve(__dirname, 'dist'),
  disableHostCheck : true,
  host: '0.0.0.0',
  port: 8080
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjUwOTgzMTI4LC0xOTgxMjU1NDYwLDE1OD
M3ODEwNDBdfQ==
-->