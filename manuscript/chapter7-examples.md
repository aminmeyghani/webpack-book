# Using Webpack with Client-side Frameworks

In this chapter we are going to see how Webpack can be used with the common workflows:

- Webpack with Angular
- Webpack with React
- Webpakc with PostCSS
- Webpack and CSS Modules.

**TODO**

## Using Angular with Webpack

In this section we are going to set up a workflow for developing Angular apps and Webpack.

### Project Setup

Let's start by creating a folder and a package file for our project:

```bash
mkdir -p ~/Desktop/ng-webpack && cd $_ && npm init
```

When prompted accept all the defaults to generate your package file.

After that, install some dependencies:

```bash
npm i webpack concurrently express faker nodemon babel-core babel-loader \
      babel-preset-es2015 babel-plugin-add-module-exports raw-loader -D
```

After all the dependencies are installed, create a symbolic link to the `bin` folder of `node_modules`:

```bash
ln -s ./node_modules/.bin ./bin
```

Then create a `webpack.config.js` file and put in the following in the file:

```javascript
var path = require('path');

module.exports = {
  entry: 'main.js',
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
    publicPath: '/public/',
    library: 'myapp',
    libraryTarget: 'umd'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        include: path.resolve('src'),
        exclude: 'node_modules',
        loader: 'babel',
        query: {
          presets: ['es2015'],
          "plugins": [ "add-module-exports" ]
        }
      },
      {
        test: /\.html$/,
        loader: 'raw'
      }
    ]
  },
  resolve: {
    modulesDirectories: ['src', 'node_modules']
  },
  externals: {
    angular: 'angular'
  }
};
```

We are pointing Webpack to look at the `src` folder for modules. So let's create that folder along with the `main.js` file:

```bash
mkdir src && touch src/main.js
```

Before we go any further let's see if the basics are set up. Put the following in the `src/main.js` and then run `./bin/webpack`:

**src/main.js**

```javascript
class Person {
  walk() {
    return 'walking ....'
  }
}
const p = new Person();
console.log(p.walk());
```

If everything is set up correctly you should see an output in the `dist` folder.

Now that we have Webpack set up, let's create a simple Express app to server our app for development:

```bash
touch server.js
```

After you created the file, copy paste the following to the file:

**server.js**

```javascript
var express = require('express');
var path = require('path');
var faker = require('faker');
var logger = require('morgan');
var app = express();
var router = express.Router();
app.use(logger('dev'));

app.use('/public', express.static(path.resolve(__dirname, 'dist')));

// GET /api/posts
router.use('/posts', function (req, res) {
  var posts = [];
  var count = 20;
  while (count-- > 0) {
    posts.push({
      id: faker.random.uuid(),
      title: faker.lorem.words(),
      content: faker.lorem.sentences(),
    })
  }
  res.json(posts);
});

// For all the requests that does not start with
// /api serve index.html
app.all(/^\/(?!api).*/, function(req, res){
  res.sendFile('index.html', {root: path.resolve(__dirname) });
});

app.all("/404", function(req, res, next) {
  res.sendFile("index.html", {root: path.resolve(__dirname)});
});

app.use('/api', router);


var port = process.env.PORT || 8089;
app.listen(port, function () {
  console.log('Server running at %s', port);
});
```

We also need to create an `index.html` for the project:

```bash
touch index.html
```

After you created the html file, copy past the following:

**index.html**

```html
<!DOCTYPE html>
<html>

<head>
  <title>Example</title>
  <script src="/public/bundle.js"></script>
</head>

<body>
  <p>Hello</p>
</body>

</html>
```

Now, if you run the server with `node server.js`, and go to `http://localhost:8089/`, you should see the text `walking` in the browser console. If you see the text, it means that you are all set up.

For our convenience, we are going to set up some task scripts in our `package.json` file:

**package.json**

```json
{
  "name": "ng-webpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "./bin/concurrently \"npm run watch\" \"npm run server\"",
    "watch": "./bin/webpack -w --debug --devtool eval --output-pathinfo",
    "server": "./bin/nodemon -w server.js -w webpack.config.js server.js"
  },
  "author": "Amin Meyghani <meyghania@gmail.com> (http://meyghani.com)",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.9.0",
    "babel-loader": "^6.2.4",
    "babel-plugin-add-module-exports": "^0.2.1",
    "babel-preset-es2015": "^6.9.0",
    "concurrently": "^2.1.0",
    "express": "^4.13.4",
    "faker": "^3.1.0",
    "morgan": "^1.7.0",
    "nodemon": "^1.9.2",
    "raw-loader": "^0.5.1",
    "webpack": "^1.13.1"
  },
  "dependencies": {
    "angular": "^1.5.5"
  }
}
```

Now you can stop the server that was running before with ctrl + c. After you stopped the server, you do `npm run dev` to start Webpack and the server in watch mode. Try it!

```bash
[1]
[1] > ng-webpack@1.0.0 server /Users/amin.meyghani/Desktop/ng-webpack
[1] > ./bin/nodemon -w server.js -w webpack.config.js server.js
[1]
[0]
[0] > ng-webpack@1.0.0 watch /Users/amin.meyghani/Desktop/ng-webpack
[0] > ./bin/webpack -w --debug --devtool eval --output-pathinfo
[0]
[1] [nodemon] 1.9.2
[1] [nodemon] to restart at any time, enter `rs`
[1] [nodemon] watching: server.js webpack.config.js
[1] [nodemon] starting `node server.js`
[0] Hash: 5bda51e6325813c68069
[0] Version: webpack 1.13.1
[0] Time: 695ms
[0]     Asset     Size  Chunks             Chunk Names
[0] bundle.js  2.67 kB       0  [emitted]  main
[0]     + 1 hidden modules
[1] Server running at 8089
```

### Hello Angular

Now let's start playing with Angular by first install it:

```bash
npm i angular -S
```

Then open your `src/main.js` file and replace the content with the following:

```javascript
const angular = require('angular');
const appModule = angular.module('app', []);

require('services')(appModule);
require('page/page-directive')(appModule);

angular.element(document).ready(function () {
  angular.bootstrap(document.getElementsByTagName('body')[0], ['app']);
});

export default appModule;
```

As you can see, first we load Angular and then create and Angular module called 'app'. Then we load `services` and the `page-directive` by passing the `appModule` to the result of require:

```javascript
const s = require('services'); // This returns a function
// then we call the function passing an instance of our appModule
s(appModule);
```

That's where the magic happens, so make sure you understand that part because that's what makes it easy to create modules and refactor them without worrying about names.

Now let's create the `src/services` folder and define the `src/services/index.js` file:

```bash
mkdir -p src/services && touch src/services/index.js
```

After you created the `src/services/index.js` file, copy past the following:

**src/services/index.js**

```javascript
export default ngModule => {
  ngModule.factory('PostService', function ($http) {
    return {
      getPosts() {
        return $http.get('/api/posts')
      }
    };
  });
};
```

Let's spend some time and understand what's happing here:

First of all we have the `export default` which is equivalent to `module.exports = ...`. Then, we have a function with `ngModule` as the parameter which is equivalent to `function (ngModule) {}`. So the ES5 equivalent of the first line is:

```javascript
module.exports = function (ngModule) {}
```
Now the body of the function is familiar, you can pretend that `ngModule` is simple an instance of the `angular.module`. But the neat part is that you don't have to care what the name of the module is, you simply add stuff to the instance. Again, this is what makes working with Angular and Webpack special, the fact that you get to decouple your modules is a big win. This will protect you from module definitions like the following:

```javascript
var app = angular.module('app', [
  'moduleName1', 'moduleName2', 'moduleName3', 'moduleName4', 'moduleName5'
]);
```

Now let's look at creating our page directive which is going to contain the definition of our directive including the controller and the template definition:

```bash
mkdir -p src/page && touch src/page/page-directive.js && touch src/page/page-tpl.html
```

After you created the files and folders, copy paste the following to their respective files:

**src/page/page-tpl.html**

```html
<p>{{pageCtrl.hello}}</p>
<ul>
  <li ng-repeat="post in pageCtrl.posts">
    {{ post.title }}
  </li>
</ul>
```

**src/page/page-directive.js**

```javascript
export default pageModule => {

  pageModule.run(function ($templateCache) {
    $templateCache.put('page-tpl', require('./page-tpl.html'));
  });

  pageModule.directive('page', function() {
    return {
      restrict: 'E',
      controller: 'pageCtrl',
      controllerAs: 'pageCtrl',
      templateUrl: 'page-tpl'
    };
  });

  pageModule.controller('pageCtrl', function($scope, PostService) {
    const pageCtrl = this;
    pageCtrl.hello = 'hello there';
    PostService.getPosts()
      .then(function ok(resp) {
        pageCtrl.posts = resp.data;
        $scope.$broadcast('posts:loaded', resp.data);
      },
      function err(errResp) {
        console.log(errResp);
      });

    $scope.$watch('pageCtrl.posts', function(newVal, oldVal) {
        if (newVal !== oldVal) {
          console.log(newVal);
        }
    });

    $scope.$on('posts:loaded', function (e, posts) {
      // can do stuff once the posts are loaded.
    });
  });
};
```

Let's go through the Webpack-specific stuff. First of all, we are adding the template of our directive to the cache:

```javascript
pageModule.run(function ($templateCache) {
  $templateCache.put('page-tpl', require('./page-tpl.html'));
});
```
If you notice we are using Webpack to load the content of the `page-tpl.html` template as a string and assigning that to the `page-tpl` value of the cache. This means that we can reference our template using the key, which in this case is `page-tpl`:

```javascript
pageModule.directive('page', function() {
  return {
    restrict: 'E',
    controller: 'pageCtrl',
    controllerAs: 'pageCtrl',
    templateUrl: 'page-tpl' // <--- Referencing the templateUrl
  };
});
```

The rest is pretty much standard Angular code. Now that you have the directive, you can load that in the `index.html` file. So now your `index.html` would like the following:

```html
<!DOCTYPE html>
<html>

<head>
  <title>Example</title>
  <script type="text/javascript" src="/node_modules/angular/angular.js"></script>
  <script src="/public/bundle.js"></script>
</head>

<body>
  <page></page>
</body>

</html>
```

After you load the page, you should be to see the posts that are returned from our Express API!

### Testing

Let's look at how you would go about testing your code. For this section we are going to use the Jasmine testing framework and Testem to run our tests.

First we need to install the Testem Runner, Angular Mocks, and some polyfills:

```bash
npm i testem angular-mocks babel-polyfill -D
```

After that make the `testem.json` config file and copy paste the following:

**testem.json**

```json
{
  "framework": "jasmine2",
  "src_files": [
    "dist/bundle.js",
    "test/client/unit/**/*.js"
  ],
  "serve_files": [
    "node_modules/babel-polyfill/dist/polyfill.min.js",
    "node_modules/angular/angular.js",
    "node_modules/angular-mocks/angular-mocks.js",
    "dist/bundle.js",
    "test/client/unit/**/*.js"
  ],
  "launch_in_dev": ["Chrome", "PhantomJS"],
  "launch_in_ci": ["PhantomJS"]
}
```

As you can see from the configuration file, we are loading the app JavaScript in the `src_files` and the dependencies in the `serve_files` field. Also, we are specifying the browsers that we want to use in different environments, that is PhantomJS in CI and Chrome, PhantomJS in development.

Now that we have the configuration set up, let's create our test folder:

```bash
mkdir -p test/client/unit
```

We are going to put all of unit tests in the `unit` folder and any subdirectory that we choose to make inside there. Let's add a simple test to test the `pageCtrl`:

```bash
touch test/client/unit/page-ctrl-test.js
```

After you created the file, copy paste the following:

**test/client/unit/page-ctrl-test.js**

```javascript
describe('pageCtrl instance:', function() {

  var $controller;
  var $rootScope;
  var $scope;
  var $q;
  var mockPostService;

  beforeEach(function () {
    module('app');
    inject(function(_$controller_, _$rootScope_, _$q_, _PostService_) {
        $controller = _$controller_;
        $rootScope = _$rootScope_;
        $q = _$q_;
        $scope = $rootScope.$new();
        mockPostService = jasmine.createSpyObj('PostService', ['getPosts']);
        mockPostService.getPosts.and.returnValue($q.when({
          data: [{id: 'blah',title: 'blah',content: 'blah'}]
        }));
        underTest = $controller('pageCtrl', {
            '$scope': $scope,
            'PostService': mockPostService
        });
        $scope.$apply();
      })
  });

  it('should have its `hello` property set. At this point we dont care what the \
    value is.', function() {
    expect(underTest.hello).toBeDefined();
  });

  it('should have the `posts` field set with bunch of values which would confirm \
    `PostService.getPosts` was called.', function () {
    expect(mockPostService.getPosts).toHaveBeenCalled();
    expect(underTest.posts).toEqual([{
      id: 'blah', title: 'blah', content: 'blah'
    }]);
    expect(underTest.posts.length).toBe(1);
    expect(mockPostService.getPosts.calls.count()).toBe(1);
  });

});
```
This is a pretty standard Angular test, we are asserting that our controller have the fields set properly. Now, to run the tests, just run `./bin/testem`. After that you should a chrome window opening where you can see the status of the tests. You could also add that command to the scripts section of the `package.json` file:

```json
"tdd": "./bin/testem",
"test": "bin/testem  ci -P 2"
```

Now you can run the following to run the tests:

```bash
npm run test # to run in ci mode
npm run tdd # to run in tdd mode
```

If you want to run the tests with PhantomJS, make sure you have it installed:

```bash
npm i phantomjs -g
```

And then you can run the tests in CI mode and you would get some output like the following:

```
1..2
# tests 2
# pass  2
# skip  0
# fail  0

# ok
```

## Using React with Webpack

Webpack and Babel makes working with React pretty straightforward. In this section we are going to set up a very simple React project that uses Babel and Webpack to bundle the app.

### Project Setup

First, create a folder on your desktop and call `npm init`:

```bash
cd ~/Desktop
mkdir webpack-react && cd $_
npm init
```

After you run `npm init` it will ask you a couple of questions. You can just use the defaults by keep hitting enter on the keyboard. Then, run the following to install the development dependencies:

```bash
npm install babel-core babel-loader \
babel-plugin-add-module-exports \
babel-preset-es2015 \
babel-preset-react webpack -D
```

After all the dev dependencies are installed, we need to install React:

```bash
npm install react react-dom -S
```

### Creating the Config File

Now that we have all the dependencies installed, we can add the `webpack.config.js` file:

```bash
touch webpack.config.js
```

After creating the the config file, add the following configuration:

```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = {
  entry: {
    app: 'main'
  },
  output: {
    filename: '[name].js',
    path: path.resolve('./dist')
  },
  devtool: "cheap-module-source-map",
  module: {
    loaders: [
      {
        test: /\.jsx$/,
        loader: 'babel',
        exclude: /node_modules/,
        query: {
          presets: ['es2015', 'react'],
          retainLines: 'true',
          plugins: ['add-module-exports']
        }
      },
    ]
  },
  resolve: {
    extensions: ['', '.webpack.js', 'web.js', '.js', '.jsx'],
    modulesDirectories: [
     'node_modules',
      path.resolve('./src'),
    ]
  }
};
```
Let's examine the config file a bit further:

- The entry point of the app is a file called `main` that is expected to exist in the resolve modules directories path.
- We are outputing the result to the `dist` folder.
- We are using a fast dev tool to generate row-only source-maps
- In the loaders section we are processing any file that ends with the `.jsx` extension. All these files are passed through the babel loader
- In the resolve section we are resolving `.js` and `.jsx` extensions in addition to Webpack default file extensions.
- Using the `modulesDirectories` we are specifying that the `src` directory is to be considered as a module folder

Now that we have the config file, we can start by creating the `src` modules directory:

```bash
mkdir src
```

Now we need to create the entry point. Remember that the entry point in our config file is set to be `main` which means that Webpack will resolve any of the following patterns:

```bash
src/main.js
src/main.jsx
...
```

### Adding the Main File

Because our main file is going to contain some JSX, we are going to name the file `main.jsx`. Let's create it in the `src` folder:

```bash
touch src/main.jsx
```

Now open the `main.jsx` file and add the following:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
const App = props => (<h1>Hello</h1>);
ReactDOM.render(<App />, document.getElementById('app'));
```

Now we can run Webpack to bundle the project:

```bash
./node_modules/.bin/webpack
```

After running this you should be able to see the bundle result in the `dist` folder. Now if you notice, you can see that React is also bundled with the package. And that's not quite what we want. We want to tell Webpack not to bundle react and react-dom and assume that somehow they are available. In order to do that, we can use the `externals` property. Let's add that to the `webpack.config.js` file:

```javascript
externals: {
  react: 'React',
  'react-dom': 'ReactDOM'
}
```

Now the full config file contains the following:

```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = {
  entry: {
    app: 'main'
  },
  output: {
    filename: '[name].js',
    path: path.resolve('./dist')
  },
  devtool: "cheap-module-source-map",
  module: {
    loaders: [
      {
        test: /\.jsx$/,
        loader: 'babel',
        exclude: /node_modules/,
        query: {
          presets: ['es2015', 'react'],
          retainLines: 'true',
          plugins: ['add-module-exports']
        }
      },
    ]
  },
  resolve: {
    extensions: ['', '.webpack.js', 'web.js', '.js', '.jsx'],
    modulesDirectories: [
     'node_modules',
      path.resolve('./src'),
    ]
  },
  externals: {
    react: 'React',
    'react-dom': 'ReactDOM'
  }
};
```

Now if you run `./node_modules/.bin/webpack` you can see that the size of the `dist/app.js` file is much smaller. And if you look closely you can see the part that has been converted from jsx to javascript:


```
var App = function App(props) {return _react2.default.createElement('h1', null, 'Hello');};
  _reactDom2.default.render(_react2.default.createElement(App, null), document.getElementById('app'));
```

If you don't want to run `./node_modules/.bin/webpack` everytime, you can just create a task in the `package.json` file:

```javascript
{
  "name": "webpack-react",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "./node_modules/.bin/webpack -w"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.21.0",
    "babel-loader": "^6.2.10",
    "babel-plugin-add-module-exports": "^0.2.1",
    "babel-preset-es2015": "^6.18.0",
    "babel-preset-react": "^6.16.0",
    "webpack": "^1.14.0"
  },
  "dependencies": {
    "react": "^15.4.2",
    "react-dom": "^15.4.2"
  }
}
```

Notice the script section:

```javascript
"scripts": {
  "dev": "./node_modules/.bin/webpack -w"
},
```

Now we can just use `npm run dev` and Webpack will bundle our app and also re builds the project on any new changes.

The last step is just adding an `index.html` file and verifying that our app works. So first create the index file:

```bash
touch index.html
```

And then add the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <script src="//cdnjs.cloudflare.com/ajax/libs/react/15.4.2/react.js"></script>
  <script src="//cdnjs.cloudflare.com/ajax/libs/react/15.4.2/react-dom.js"></script>
  <title>React with Webpack</title>
</head>
<body>
  <div id="app"></div>
  <script src="/dist/app.js"></script>
</body>
</html>
```

Now all we have to do is to server the folder. You can use `http-server` to do that. If you don't have it, you can install it by `npm i http-server -g` and then run `http-server .` Simply navigate to `http://localhost:8080/` and you should be able to see `hello` on the screen.



## Webpack with PostCSS

Webpack combined with PostCSS creates a very nice workflow for authoring CSS. In this section we are going to explore how you can use Webpack with the autoprefixer PostCSS plugin to autoprefix your css.

### Set up

Like always we are going to set up a project folder and the webpack.config.js file:

```bash
mkdir -p ~/Desktop/webpack-css-example && cd $_ && npm init
```

Once prompted, accept all the defaults to create the package.json file. Once the file has been created you are ready to go. First, let's install all the dependencies including Webpack to get started:

```bash
npm i autoprefixer css-loader postcss-loader precss style-loader webpack -D
```

If you are noticing we are installing the modules as dev dependencies because we only need them for development purposes.

Once the dependencies are installed, we are ready to set up the configuration file for our Webpack:

**webpack.config.js**

```javascript
var path = require('path');
var webpack = require('webpack');
var precss = require('precss');
var autoprefixer = require('autoprefixer');

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve('dist')
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: 'style!css!postcss'
      }
    ]
  },
  postcss: function () {
    return {
      defaults: [precss, autoprefixer],
      cleaner:  [autoprefixer({ browsers: ['ie >= 10', 'last 2 versions'] })]
    };
  }
};
```

After creating the config file, we just need to create the `main.js` file and the `main.css` file to demonstrate how the autoprefixing would works. So let's create the two files right now and put the following code in them:


```bash
touch main.js main.css
```

**mian.js**

```javascript
require('./main.css');
```

**main.css**

```css
.container {
  display: flex;
}
```

After you created the files, you can compile the code with Webpack. Just run `./node_modules/.bin/webpack` and you should see the output in the `dist` folder. And of course you can put Webpack on watch mode by using the `-w` flag. After you compile the code, you should see an output like the following in the `dist/bundle.js` file:

**dist/bundle.js**

```javascript
//.....
//......
exports.push([module.id, ".container {\n  display: -webkit-box;\n  display: -ms-flexbox;\n  display: flex;\n}\n", ""]);
```

As you can see the CSS has be automatically prefixed using the options that we have defined in the 'webpack.config.js' file. You can obviously change that based on your project or spit out a new file if you need to have separate CSS files.

I have found this to be very usefully specially when working with flexbox because flexbox has different specifications that have been implemented differently across browsers. Autoprefixer solves that problem so you don't have to worry about the prefixes or different specifications.



## Webpack with CSS Modules

You can use Webpack to namespace your CSS classes, which essentially means creating local scopes for your selectors. For example, if you have two components, a button and an input, no longer would you have to worry about naming your classes. You can simply name your selector and it would only apply to your given component. Now the downside is that this works in JavaScript and would not work in the pure CSS world. This is perfect when working with Angular directives or React components.

**TODO**

