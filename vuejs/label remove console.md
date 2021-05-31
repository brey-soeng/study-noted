This plugin removes all console.* calls.

=> install package

**npm i babel-plugin-transform-remove-console**



in the label.config.js code we need to remove all the console log in the production

// Remove the console from the production environment

```
if (process.env.NODE_ENV === 'production') {
  plugins.push('transform-remove-console')
}

module.exports = {

plugins: plugins,

presets: [
    '@vue/app'
  ]
}
```

==========link===========

https://www.npmjs.com/package/babel-plugin-transform-remove-console