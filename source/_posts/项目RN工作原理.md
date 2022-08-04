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
const hahahaPageContainer = createPageContainer<Store.IState>(PACKAGE_NAME, store, PluginInit)
export const PageContainer: typeof hahahaPageContainer = (
  pageName,
  OriginalComponent,
  mapStateToProps,
  mapDispatchToProps
) => {
  hahahaPageContainer(
    pageName,
    // @ts-ignore
    // OriginalComponent,
    pageWrapper(pageName, OriginalComponent),
    mapStateToProps,
    mapDispatchToProps
  )
}

xxxxRegistry = createhahahaRegistry(AppRegistry.registerComponen）
xxxxRegistry.registerComponent(pageName, () => Container);
```
