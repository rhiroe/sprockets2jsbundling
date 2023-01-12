# Sprocketsからjsbunding-railsのesbuildに移行する検証

## rails + sprockets の環境を作る

```shell
bundle exec rails new . --skip-javascript --skip-turbolinks
```

- `app/assets/javascripts/application.js`を作成
- `app/views/layout/application.html.erb`に`<%= javascript_include_tag 'application' %>`を追加
- `assets/config/manifest.js` に `//= link application.js` の1行を追加

## 動作確認

```shell
bin/rails g controller home index
```

`config/routes.rb`を修正
```diff
- get 'home/index'
+ root 'home#index'
```

`app/assets/javascripts/alert.js`を追加
```js
const alertFunc = () => alert();
```

`app/assets/javascripts/application.js`に以下を記述
```js
//= require './alert'
window.addEventListener('load', () => {
  alertFunc();
})
```

```shell
bin/rails s
```

## jsbundling-railsを入れる

```shell
bundle add jsbundling-rails
yarn init
bin/rails javascript:install:esbuild
```

- `app/javascript/application.js`が生成された
- `app/views/layout/application.html.erb`に`<%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>`が追加された
- `app/assets/manifest.js`に`//= link_tree ../builds`が追加された
  - `app/javascript/`以下のJavaScriptに変更があると、ビルドされて`app/assets/build`に配置されるみたい
- `package.json`に自動で変更が入った
- その他もろもろなファイルが自動追加された

## バンドラーをwatchモードで起動する

```shell
yarn build --watch
```

## 既存のファイルを修正する

- `app/views/layout/application.html.erb`から最初に追加した `<%= javascript_include_tag 'application' %>`の記述を削除 
- `app/assets/manifest.js`から`//= link application.js`の記述を削除 
- `app/assets/javascripts/alert.js`を`app/javascript/alert.js`に移動、内容を以下に変更
```diff
- const alertFunc = () => alert();
+ export const alertFunc = () => alert();
```
- app/javascript/application.js に以下を記述
```javascript
import { alertFunc } from "./alert";
window.addEventListener('load', () => {
  alertFunc();
})
```
- `app/assets/javascripts/`を削除

## 動作確認

```shell
bin/rails s
```
