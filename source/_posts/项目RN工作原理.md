---
title: RN工作原理
date: 2020-07-03 11:48:42
tags:
---

### 一、基础原理

```sh
PageContainer(
  MODULES.OUTLET_DETAIL,
  OutletDetailPage
)
const shopeePageContainer = createPageContainer<Store.IState>(PACKAGE_NAME, store, PluginInit)
export const PageContainer: typeof shopeePageContainer = (
  pageName,
  OriginalComponent,
  mapStateToProps,
  mapDispatchToProps
) => {
  shopeePageContainer(
    pageName,
    // @ts-ignore
    // OriginalComponent,
    pageWrapper(pageName, OriginalComponent),
    mapStateToProps,
    mapDispatchToProps
  )
}

createShopeeRegistry （AppRegistry.registerComponen）

```
