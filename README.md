# vue-loader-test


## vue-loader 流程

1. `vue-plugin` 解析记录 rules，并在 vue 文件的 loader 中添加 `pitcher` loader，pitcher 的规则为导入路径存在`?vue&type=...`

2. webpack 开始解析，进入 `index.js` ，中解析字符串，分成 3 部分

    ```javascript
    import { render } from "./App.vue?vue&type=template&id=e920d152&scoped=true"
    import script from "./App.vue?vue&type=script&lang=ts"
    export * from "./App.vue?vue&type=script&lang=ts"

    import "./App.vue?vue&type=style&index=0&id=e920d152&lang=scss&scoped=true"
    script.render = render
    script.__scopeId = "data-v-e920d152"
    script.__hmrId = "e920d152"
    /* hot reload */
    if (module.hot) {
      const api = __VUE_HMR_RUNTIME__
      module.hot.accept()
      if (!api.createRecord('e920d152', script)) {
        api.reload('e920d152', script)
      }

      module.hot.accept("./App.vue?vue&type=template&id=e920d152&scoped=true", () => {
        api.rerender('e920d152', render)
      })

    }

    script.__file = "src/views/App.vue"

    export default script
    ```

3. 解析后的字符串有新的 import，于是 webpack 再次分析，因为导入路径中存在 `?vue&type=template&id=e920d152&scoped=true` 这部分，进入 pitcher loader 中。

4. 在 `pitcher` loader 中，按照 type 生成新的 loader 链

   ```javascript
   export * from "-!../../node_modules/vue-loader/dist/templateLoader.js??ref--5!../../node_modules/vue-loader/dist/index.js??ref--10-0!./App.vue?vue&type=template&id=e920d152&scoped=true"
   ```

5. webpack 再次解析新的 import，再次进入 `index.js` 。`index.js` 会判断路径是否存在 `type=...`，存在，再把之前解析好的仅包含该部分的字符串传递到下一个 loader。

总结： `vue-loader` 相当于把单文件解析成统一目录下的多个文件，再提供一个统一的出口。
