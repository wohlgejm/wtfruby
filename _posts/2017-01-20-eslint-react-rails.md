---
layout: post
comments: true
title: eslint config for react-rails
date: 2017-01-20 20:20:30
categories: rails react
short_description: Configuring eslint with react-rails gem
---

[react-rails](https://github.com/reactjs/react-rails) is a gentle way to introduce react into your rails app.
You can still use the asset pipeline and it gives you an easy way to mount components in your views.
It also gives you an easy way to generate components, `rails g react:component FancyDropdown`.
You'll probably out grow this quickly, but jumping into the react ecosystem is taking a leap into
buzzword alphabet soup - it's almost impossible to figure out how to get started.

If you want to use eslint with `react-rails` here's what you're `.eslintrc` can look like in
`app/assets/javascripts/components/.eslintrc`:

```
env:
 es6: true
parserOptions:
  ecmaFeatures:
    jsx: true
plugins:
  - react
extends:
  - eslint:recommended
  - plugin:react/recommended

rules:
  no-unused-vars: off
  no-undef: off
  react/jsx-no-undef: off
  react/react-in-jsx-scope: off
```

You'll want to install the eslint plugin [eslint-plugin-react](https://github.com/yannickcr/eslint-plugin-react).

Some of the rules are disabled because you're using asset pipeline. React is available on the window, so eslint
believes it is undefined.

The biggest issue I'm running into now is the need to use react libraries. Without [webpack](https://webpack.github.io/), getting these to work is either a hack or not possible.
