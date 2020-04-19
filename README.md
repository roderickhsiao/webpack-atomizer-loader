[![npm version](https://badge.fury.io/js/webpack-atomizer-loader.svg)](http://badge.fury.io/js/webpack-atomizer-loader)
[![Build Status](https://travis-ci.org/acss-io/webpack-atomizer-loader.svg?branch=master)](https://travis-ci.org/acss-io/webpack-atomizer-loader)
# webpack-atomizer-loader
Webpack loader for compiling [Atomic CSS](https://acss.io).

## Table of Contents

1. [Install](#install)
1. [Atomic CSS configuration](#atomic-css-configuration)
1. [Usage with React or Vue](#usage-with-react-or-vue)
1. [Usage with `mini-css-extract-plugin` and `webpack-html-plugin`](#usage-with-mini-css-extract-plugin-and-webpack-html-plugin)
1. [Advanced](#advanced)

## Install
```bash
$ npm install webpack-atomizer-loader --save-dev
```
or
```bash
$ yarn add webpack-atomizer-loader -D
```

## Atomic CSS configuration

You need in a JavaScript file the Atomic CSS configuration that will be feed to the loader. For example, `atomCssConfig.js` will looks something like:
```js
// atomCssConfig.js

module.exports = {
    cssDest: './generatedAtoms.css',
    options: {
        namespace: '#atomic',
    },
    configs: {
        breakPoints: {
            sm: '@media screen(min-width=750px)',
            md: '@media(min-width=1000px)',
            lg: '@media(min-width=1200px)'
        },
        custom: {
            1: '1px solid #000',
        },
        classNames: []
    }
}
```

 To assign the output destination of the generated CSS the parameter `cssDest` should be set. If no specified the default value of `cssDest` is `./build/css/atomic.css`.

 To know more about Atomic CSS configuration check https://github.com/acss-io/atomizer#api.

## Usage with React or Vue

If the CSS atoms on your project are written in the `className` prop of JSX files follow these intructions.

Find `babel-loader` or `jsx-loader` on your webpack configuration and insert `webpack-atomizer-loader` before it:

```js
// webpack.config.js

const path = require('path');

module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?$/, // or /\.vue?$/
                exclude: /node_modules/,
                use: [
                    {
                        loader: 'webpack-atomizer-loader',
                        options: {
                            configPath: path.resolve('./atomCssConfig.js')
                        }
                    },
                    {
                        loader: 'babel-loader',
                        options: {
                            presets: ['react', 'es2015']
                        }
                    },
                ]
            },
            {
                test: /\.css$/, // or /\.scss$/
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    {
                        loader: 'postcss-loader', // or 'sass-loader'
                        options: {
                            // ...
                        }
                    },
                ]
            }
        ]
    }
};
```

You need `css-loader` which will convert CSS into a JavaScript variable every time you do `import CSS from './generatedAtoms.css`. If you use some CSS preprocessor like SASS or PostCSS you'll have to also add the corresponding loader, which will have to go after the `css-loader` on the `rules` array as shown on the above example.

`webpack-atomizer-loader` will generate a `generatedAtoms.css` file which will have to be imported in some Javascript or CSS file:

```javascript
// index.js

import React from 'react'
import ReactDOM from 'react-dom'

import Style from './generatedAtoms.css'
import App from './app'

ReactDOM.render(<App />, document.getElementById('root'))
```

or

```css
/* index.css */

@import "projectStyles.css";
@import "generatedAtoms.css";
```

After this you should be able to see the Atom CSS classes loaded on the browser and applied to your components.

## Usage with `mini-css-extract-plugin` and `webpack-html-plugin`

If the CSS atoms are on `class` attributes on `.htm` files (or any other template system like [`hbs`](https://github.com/pcardune/handlebars-loader), `pug` or `ejs`) you must use [`mini-css-extract-plugin`](https://github.com/webpack-contrib/mini-css-extract-plugin) and optionally [`webpack-html-plugin`](https://github.com/jantimon/html-webpack-plugin):

```javascript
// webpack.config.js

const path = require('path');

module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/, // or /\.scss$/
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    {
                        loader: 'postcss-loader', // or 'sass-loader'
                        options: {
                            // ...
                        }
                    },
                ]
            },
            {
                test: /\.htm$/, // or /\.hbs$/
                use: [
                    {
                        loader: 'html-loader', // Or the corresponding loader for the template system you're using
                        options: {
                            attributes: false,
                            minimize: true
                        }
                    },
                    {
                        loader: 'webpack-atomizer-loader',
                        options: {
                            configPath: path.resolve('./atomCssConfig.js')
                        }
                    }
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ // Only needed if you're using this plugin
            template: 'src/originTemplate.htm',
            filename: 'dist/destinationFile.htm'
        }),
        new MiniCssExtractPlugin()
    ]
};
```

By default if no HTML loader is specified `webpack-html-plugin` will use
a simple [`ejs` loader](https://github.com/jantimon/html-webpack-plugin/blob/master/docs/template-option.md). However as soon as `webpack-atomizer-loader` is included the default HTML loader will be disabled and we'll have to include ours. That's why on the above configuration in addition to `webpack-atomizer-loader` it's also included `html-loader`.

You only need `webpack-html-plugin` if you're using this plugin to also generate the HTML. If you have your own `public/index.htm` with a `<link href="projectStyles.css" />` element remember to include the generated Atoms with another `<link href="generatedAtoms.css" />` or in a `css` file with `@import "generatedAtoms.css"`.

  ## Advanced

  ### PostCSS plugins
  `webpack-atomizer-loader` now supports processing output CSS file with postcss by doing this:

  ```js
  var path = require('path');
  var autoprefixer = require('autoprefixer');
    .
    .
    .

    {
        test: /\.jsx?$/,
        exclude: /(node_modules)/,
        loader: 'webpack-atomizer-loader',
        query: {
            postcssPlugins: [autoprefixer]
            configPath: [
                path.resolve('./atomCssConfig.js')
            ]
        }
    }

  ```

  ### Minimize output CSS file

  Set `minimize` to `true` to loader's config
  ```js
  var path = require('path');
  var autoprefixer = require('autoprefixer');
    .
    .
    .

    {
        test: /\.jsx?$/,
        exclude: /(node_modules)/,
        loader: 'webpack-atomizer-loader',
        query: {
            postcssPlugins: [autoprefixer]
            minimize: true,
            configPath: [
                path.resolve('./atomCssConfig.js')
            ]
        }
    }

  ```

  
  Please visit [acss-io/atomizer](https://github.com/acss-io/atomizer) for more information.
  
