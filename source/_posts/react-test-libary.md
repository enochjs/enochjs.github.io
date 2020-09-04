---
title: react-test-libary
date: 2020-08-25 17:23:03
categories: 
  - 测试
tags:
---

### reader
```javascript
function render(
  ui: React.ReactElement<any>,
  options?: {
    /* You won't often use this, expand below for docs on options */
  }
): RenderResult

// custom render
// test-utils.js
import React from 'react'
import { render } from '@testing-library/react'
import { ThemeProvider } from 'my-ui-lib'
import { TranslationProvider } from 'my-i18n-lib'
import defaultStrings from 'i18n/en-x-default'

const AllTheProviders = ({ children }) => {
  return (
    <ThemeProvider theme="light">
      <TranslationProvider messages={defaultStrings}>
        {children}
      </TranslationProvider>
    </ThemeProvider>
  )
}

const customRender = (ui, options) =>
  render(ui, { wrapper: AllTheProviders, ...options })

// re-export everything
export * from '@testing-library/react'

// override render method
export { customRender as render }
```
1. options
  - container
  - baseElement
  - hydrate
  - wrapper
  - queries

2. renderResult
  - ...queries
    1. document.body
      - getBy
        > 返回第一个匹配到的值，没有报错
      - getAllBy
        > 返回所有匹配到的值，没有报错
      - queryBy
        > 返回第一个匹配到的值，没找到返回null
      - queryAllBy
        > 返回所有匹配到的值，没找到返回[]
      - findBy
        > 返回一个promise, resolve第一个匹配到的值，没有reject
      - findAllBy
        > 返回一个promise, resolve element[]，没有reject
    2. Queries
      - ByLabelText
      - ByPlaceholderText
      - ByText
      - ByAltText
      - ByTitle
      - ByDisplayValue
      - ByRole
      - ByTestId

  - container  
  - baseElement
    > default document.body
  - debug
    > console.log(prettyDOM(baseElement))
  - rerender
    ```javascript
    import { render } from '@testing-library/react'

    const { rerender } = render(<NumberDisplay number={1} />)

    // re-render the same component with different props
    rerender(<NumberDisplay number={2} />)
    ```
  - unmount
  - asFragment

### cleanup
  > 卸载render的组件
  > test.afterEach(cleanup)

### act

### react-test-libary文档
[jest](https://testing-library.com/docs/intro)
