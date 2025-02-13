
This is a fork of https://github.com/BenceSzalai/vue-components-ide-helper. And changed to work with VueJs 3. Below in this readme is the original documentation from BenceSzalai.

I use this helper in combination with Laravel. I comment out the old dynamic component registrator and include the file generated by this script. This script gets triggered by the Laravel mix when files change. See the implementation below.

Install:

```js
npm install --save vue-components-ide-helper-vuejs3
```

Import in app.js:

```js
// import vue components
// const files = require.context('./', true, /\.vue$/i)
// files.keys().map(key => app.component(key.split('/').pop().split('.')[0], files(key).default))

import VueComponents from "./components";
app.use(VueComponents);
```

Add command to webpack.js (Laravel mix):

```js
    .before(() => {
        exec('$(npm bin)/vue-components-ide-helper -s $(pwd)/resources/js -o resources/js/components.js -r .');
    });
```

```js
// import vue components
// const files = require.context('./', true, /\.vue$/i)
// files.keys().map(key => app.component(key.split('/').pop().split('.')[0], files(key).default))

import VueComponents from "./components";
app.use(VueComponents);
```

# Vue Components IDE Helper

Can be used to generate JS import and Vue.component() statements for all files in a folder.

## Explanation

Your setup may utilise Webpack's `require.context()` feature to scan a dedicated folder for Vue Components and register them with Vue. This is useful, as with such setup there is no need to manually register your global Vue Components. They are available to be used just by adding them to the dedicated global components' folder.

However, in most cases IDEs' static code analysis facilities fail to process the complex expressions required to make such setups work. This means you may end up without code completion and/or warnings about unknown components in your IDE when using the auto-registered global Vue Components, even thought they work perfectly fine in the built application.

This small utility attempts to solve that problem in the following way:
* it looks at your global components folder
* finds all .vue files in there
* generates a `.js` file which:
  * imports all of these Vue Components
  * registers all of them with Vue
  
The purpose of the output `.js` file is not to be executed (even though it could be). The purpose of this file is to be indexed by the IDE and used for code completion.

## Usage

### Installation
Install using npm, e.g.: 

`npm install --save vue-components-ide-helper`

### Execution
* If installed as a node (dev) dependency, use one of these:
  * `$(npm bin)/vue-components-ide-helper <args>`
  * `node node_modules/.bin/vue-components-ide-helper <args>`

* In the folder of the package, use one of these:
  * `node . <args>`
  * `npm run generate -- <args>`
  
Where `<args>` is the placeholder for the command arguments, e.g.: `-s /path/to/src/components/global -o "path/to/output/file.js -r ../src/components/global`

### Arguments
The tool supports the following command line arguments:
* `-s <components_dir_path>` *(mandatory)* : The path to the global Component files.
* `-o <output_file_path>` *(mandatory)* : The path to write the generated `.js` file as.
* `-r <relative_path>` *(optional)* : If provided, this path will be written as the `import` path of the Components in the output `.js` file instead of the path provided under `-s`. 
  * If parameter `-s` is an absolute path, this is not necessary, but can be used to simplify the `import` statements in the generated `.js` file or to make it independent of the project path. (See the example below.)
  * If parameter `-s` is a relative path, this may be required, because the relative path of the components folder may be different looking from the place of the execution of this tool and looking from the place of the output `.js` file. 

### Automation

It is recommended to set up a file watch either in your IDE or using [npm-watch](https://www.npmjs.com/package/npm-watch) to run the above command every time there is a change made to the `components/global` folder.

## Example case

* Webpack is set up to automatically import and register all Vue Components in the `components/global` folder
* `vue-components-ide-helper` is installed using `npm`
* Create a `.ide-helper` folder in your project and make sure it is included in your IDE's indexing
* Run in the `project-root` folder:
`$(npm bin)/vue-components-ide-helper -s $(pwd)/src/components/global -o .ide-helper/components.js -r ../src/components/global` where :
  * `-s` is the path to look for the components,
  * `-o` is the output file, and
  * `-r` is the relative path of the components folder looking from the perspective of the output file.

Now your project looks like this:
```
project-root
├── .ide-helper
│   └── components.js
├── src
│   ├── components
│   │   └── global
│   │       ├── Component1.Vue
│   │       └── Component2.Vue
│   └── ...
├── node_modules
│   └── ...
```

And `components.js` contains this:
```js
import Component1 from '../src/components/global/Component1.vue'
import Component2 from '../src/components/global/Component2.vue'

export default async ({ Vue }) => {
  Vue.component('Component1', Component1)
  Vue.component('Component2', Component2)
}
```

This means your IDE can read `.ide-helper/components.js` and understand the details of the global Vue components.


## Additional info

If you are interested how to set up Webpack's `require.context()` feature, check out these resources:
* https://vuejs.org/v2/guide/components-registration.html#Automatic-Global-Registration-of-Base-Components
* https://webpack.js.org/guides/dependency-management/#requirecontext

***

Bence Szalai - https://sbnc.eu/
