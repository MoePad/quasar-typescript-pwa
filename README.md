== Init Standard Kit

* `npm install -g quasar-cli`
* `quasar init pwa`
* `cd pwa`
* `npm install`
* `quasar dev`
  * test if everything works (i.e. starts without errors)


== Add TypeScript support

=== Remove Babel and ESLint

* remove `.babelrc`
* remove `.eslintignore`
* remove `.eslintrc.js`
* from `package.json`
  * all `babel-` dependencies
  * `eslint` dependency
  * all `eslint-` dependencies
* from `build/webpack.base.conf.js`
  * module.rules
    * remove `eslint-loader`
    * remove `babel-loader`
    * adapt `vue-loader`
      * ```
        {
          test: /\.vue$/,
          loader: 'vue-loader',
          options: {
            postcss: cssUtils.postcss,
            loaders: cssUtils.styleLoaders({
              sourceMap: useCssSourceMap,
              extract: env.prod
            })
          }
        }
        ```
* test if everything still works (i.e. starts without errors)
  * `rm -rf node_modules`
  * `npm install`
  * `quasar dev`


=== Add TypeScript

* `npm install typescript ts-loader vue-class-component @types/webpack @types/webpack-merge @types/html-webpack-plugin @types/node @types/extract-text-webpack-plugin @types/express karma-sourcemap-loader mocha-webpack --save-dev`
* in `build/webpack.base.conf.js`
  * adapt entry point
    * `app: './src/main.ts'`
      * i.e. rename main.**js** to main.**ts**
  * adapt resolve section
    * `extensions: ['.js', '.ts', '.vue', '.json'`
      * i.e. add *.ts* to the extension section
  * add to module.rules:
    * ```
      {
        test: /\.ts$/,
        include: [commons.resolve('src')],
        exclude: /node_modules|vue\/src/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/]
        }
      }
      ```
  * adapt module.rules for `vue-loader`
    * ```
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          esModule: true,
          postcss: cssUtils.postcss,
          loaders:  cssUtils.styleLoaders({
            sourceMap: useCssSourceMap,
            extract: env.prod
          }),
          transformToRequire: {
            video: 'src',
            source: 'src',
            img: 'src',
            image: 'xlink:href'
          }
        }
      }
      ```
      * i.e. add *esModule: true*
* in `App.vue` adapt `<script>` section
  * ```
    <script lang="ts">
    import Vue from 'vue'
    import Component from 'vue-class-component'

    @Component({})
    export default class App extends Vue {
    }
    </script>
    ```
* in `components/Hello.vue` adapt `<script>` section
  * ```
    <script lang="ts">
    import Vue from 'vue'
    import Component from 'vue-class-component'

    @Component({})
    export default class App extends Vue {
    }
    </script>
    ```
* rename `main.js` to `main.ts` and change the content
  * ```
    require(`quasar/dist/quasar.${__THEME}.css`)

    import Vue from 'vue'
    import Quasar from 'quasar'
    import router from './router'
    import App from './App.vue'

    Vue.config.productionTip = false
    Vue.use(Quasar)

    console.log(JSON.stringify(App))
    const vm = new Vue({
      el: '#q-app',
      router,
      render: h => h(App)
    })

    export {vm}
    ```
* rename `router.js` to `router.ts` and change the content
  * ```
    import Vue from 'vue'
    import VueRouter from 'vue-router'

    import Hello from '@/Hello.vue'

    Vue.use(VueRouter)

    /*
    * Uncomment this section and use "load()" if you want
    * to lazy load routes.
    function load (component) {
      // '@' is aliased to src/components
      return () => import(`@/${component}.vue`)
    }
    */

    const routeOptions = <VueRouter.RouterOptions>{
      routes: [
        { path: '/', component: Hello}
      ]
    }

    export default new VueRouter(routeOptions)
    ```
* add `quasar.d.ts` to `src` folder
  * ```
    declare module "*.vue" {
      import * as Vue from 'vue';
      export default typeof Vue
    }

    declare module "quasar"
    declare const __THEME
    ```
* add `tsconfig.json`
  * ```
    {
      "compileOnSave": false,
      "compilerOptions": {
          "baseUrl": ".",
          "allowSyntheticDefaultImports": true,
          "lib": [
              "dom",
              "es2015",
              "es2016"
          ],
          "jsx": "preserve",
          "target": "es2016",
          "module": "commonjs",
          "moduleResolution": "node",
          "noImplicitAny": true,
          "noUnusedLocals": true,
          "noUnusedParameters": true,
          "removeComments": true,
          "preserveConstEnums": true,
          "sourceMap": true,
          "skipLibCheck": true,
          "experimentalDecorators": true
      },
      "exclude": [
        "node_modules"
      ],
      "include": [
          "./src/**/*"
      ]
    }
    ```

== Set Up TSLint
* `npm install tslint --save-dev`
* add `tslint.json` to project root
  * ```
    {
      "extends": "tslint:recommended",
      "rules": {
          "max-line-length": {
              "options": [140]
          },
          "quotemark": [true, "single"],
          "semicolon": [true, "never"],
          "curly": [true, "ignore-same-line"],
          "whitespace": [true, "check-decl", "check-operator", "check-typecast"],
          "comment-format": false,
          "ordered-imports": false,
          "trailing-comma": false,
          "arrow-parens": false,
          "no-debugger": {
            "severity": "warning"
          },
          "new-parens": true,
          "no-arg": true,
          "no-bitwise": true,
          "no-conditional-assignment": true,
          "no-consecutive-blank-lines": false,
          "object-literal-sort-keys": false,
          "no-console": {
              "options": [
                  "debug",
                  "info",
                  "log",
                  "time",
                  "timeEnd",
                  "trace"
              ],
              "severity": "warning"
          },
          "variable-name": [
            true,
            "ban-keywords",
            "check-format",
            "allow-leading-underscore"
          ]
      }
    }
    ```
* adapt `packag.json` script section
  * `"lint": "tslint src/**/*.ts -c tslint.json"`
* run `npm run lint` and fix the lint errors

* add tslint to webpack
  * `npm install tslint-loader --save-dev`
  * add to module.rules in `webpack.base.conf.js`
    * ```
      {
        enforce: 'pre',
        test: /\.ts$/,
        loader: 'tslint-loader',
        exclude: /(node_modules)/,
        options: {
            configFile: './tslint.json'
        }
      }
      ```
    * adapt vueloader (add ts-loader and tslint-loader for ts files)
      * ```
        {
          test: /\.vue$/,
          include: [commons.resolve('src/main')],
          loader: 'vue-loader',
          options: {
            esModule: true,
            postcss: cssUtils.postcss,
            loaders:  {
              css: cssUtils.styleLoaders({
              sourceMap: useCssSourceMap,
              extract: env.prod
              }),
              ts: 'ts-loader!tslint-loader'
            },
            transformToRequire: {
              video: 'src',
              source: 'src',
              img: 'src',
              image: 'xlink:href'
            }
          }
        }
        ```

== Set Up Testing
* `npm install mocha karma @types/mocha chai @types/chai jsdom @types/jsdom mocha-typescript karma-mocha karma-mocha-reporter karma-chrome-launcher karma-typescript karma-typescript-preprocessor karma-webpack karma-coverage karma-chai @types/karma @types/karma-chai --save-dev`
* add `webpack.test.conf.js`
  * ```
    ```
