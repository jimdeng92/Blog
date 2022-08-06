---
title: React实现分页组件
date: 2021-12-09 09:40:14
permalink: /pages/88c38c/
categories: 
  - 开发者手册
  - React
tags: 
  - 
---

一直想好好学习一下 React ，也偶尔去学习过，但是因为公司业务没有涉及到，了解得都不是很透彻，基本学过了就忘了，这一次我想用 React 做一个全程自己手写的博客系统，包括 前端、后台系统和后端 Api 服务（用 Node），在开始时就遇到一个翻页组件，以前都用现成的轮子，这次乘着学习的功夫也来造一造轮子。基本的需求如下：

![react_01](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/react_01.png)

![react_02](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/react_02.png)

![react_03](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/react_03.png)

组件使用示例：

``` jsx
<Pagination
  total={111}
  current={1}
  pageSize={10}
  hideOnSinglePage={false}
  onChange={handlePageChange}
/>
```

组件属性我参考了 ant.design 的分页组件，它只有 total 属性是必须的，其他都为选填。

我实现组件的主要思路分为：

- 总页数小于等于 5 页时，直接遍历总页数然后渲染；
- 总页数大于 5 页时，根据省略号的显示又可以分为只有左省略号，只有右省略号和两边都有省略号的情况，当当前页在前 3 页时，此时只有右省略号；当当前页在 后 3 页时，此时只有左省略号；当不属于前 3 页也不在后 3 页时，此时两边都有省略号；（因为第二种和第三种情况非常类似，所有我就一起处理了）
- 只有一页并且 hideOnSinglePage 属性为 true 时，不渲染组件；
- 默认当前页是第一页，默认每页 10 条，页码改变时的回调参数为当前页码。

看看具体的实现：

``` jsx
import React from 'react'
import './index.scss'

// 左边的省略号
const LeftEllipsis = (props) => {
  return (
    <>
      <div className="Pagination-item" onClick={() => { props.handleItemClick(0) }}> 1 </div>
      <div>...</div>
    </>
  )
}

// 右边的省略号
const RightEllipsis = (props) => {
  return (
    <>
      <div>...</div>
      <div className="Pagination-item" onClick={() => { props.handleItemClick(props.totalPage - 1) }}>
        {props.totalPage}
      </div>
    </>
  )
}

/**
 * 分页器
 * @param {number} total 必须，总条数
 * @param {number} current 非必须，当前页，默认 1
 * @param {number} pageSize 非必须，每页条数，默认 10
 * @param {boolean} hideOnSinglePage 非必须，只有一页时是否隐藏分页器， 默认 false
 * @param {function} onChange 非必须，页码改变的回调，参数是当前页码
 */
const Pagination = ({
  total,
  onChange,
  current = 1,
  pageSize = 10,
  hideOnSinglePage = false
}) => {
  const [totalPage] = React.useState(Math.ceil(total/pageSize))
  const [activeIndex, setActiveIndex] = React.useState(current - 1)
  // 点击 item
  function handleItemClick(index) {
    if (index === activeIndex) return
    setActiveIndex(index)
    onChange && onChange(index)
  }
  // 只有一页时是否隐藏分页器
  if (hideOnSinglePage && totalPage <= 1) return null
  return (
    <div className="Pagination">
      {/* 上一页 */}
      <div
        className={activeIndex === 0 ? 'Pagination-item hidden' : 'Pagination-item'}
        onClick={() => { handleItemClick(activeIndex - 1) }}
      > « </div>
      {/* 小于等于 5 页 */}
      {
        totalPage <= 5 && [...Array(totalPage).keys()].map((item, index) => {
          return (
            <div
              className={activeIndex === index ? 'Pagination-item active' : 'Pagination-item'}
              key={index}
              onClick={() => { handleItemClick(index) }}
            >
              {index + 1}
            </div>
          )
        })
      }
      {/* 大于 5 页 && 在前 3 页 */}
      {
        (totalPage > 5 && activeIndex <= 2) && (
          <>
            {
              [...Array(activeIndex + 2).keys()].map((item, index) => {
                return (
                  <div
                    className={activeIndex === index ? 'Pagination-item active' : 'Pagination-item'}
                    key={index}
                    onClick={() => { handleItemClick(index) }}
                  >
                    {index + 1}
                  </div>
                )
              })
            }
            <RightEllipsis handleItemClick={handleItemClick} totalPage={totalPage}/>
          </>
        )
      }
      {/* 大于 5 页 && 不在前 3 页 */}
      {
        (totalPage > 5 && activeIndex > 2) && (
          <>
            <LeftEllipsis handleItemClick={handleItemClick}/>
            {
              // 在后 3 页数组长度取 totalPage - activeIndex + 1
              // 在中间数组长度取 3
              [...Array(totalPage - activeIndex <= 3 ? (totalPage - activeIndex + 1) : 3).keys()].map((item, index) => {
                return (
                  <div
                    className={index === 1 ? 'Pagination-item active' : 'Pagination-item'}
                    key={activeIndex + index - 1}
                    onClick={() => { handleItemClick(activeIndex + index - 1) }}
                  >
                    {activeIndex + index}
                  </div>
                )
              })
            }
            {
              // 不在后 3 页：右边的省略号
              totalPage - activeIndex > 3 && <RightEllipsis handleItemClick={handleItemClick} totalPage={totalPage}/>
            }
          </>
        )
      }
      {/* 下一页 */}
      <div
        className={activeIndex === totalPage - 1 ? 'Pagination-item hidden' : 'Pagination-item'}
        onClick={() => { handleItemClick(activeIndex + 1) }}
      > » </div>
    </div>
  )
}

export default Pagination
```

``` scss
// index.scss
.Pagination {
  padding: 10px;
  display: flex;
  justify-content: flex-start;
  .Pagination-item {
    margin: 0 5px;
    padding: 5px 18px;
    border-radius: 3px;
    border: 1px solid var(--border-color);
    -webkit-user-select:none;
    user-select: none;
    &:hover {
      cursor: pointer;
      color: #fff;
      background-color: var(--link-color);
      border: 1px solid var(--link-color);
      transition: all .5s;
    }
    &.active {
      color: #fff;
      background-color: var(--link-color);
      border: 1px solid var(--link-color);
    }
    &.hidden {
      visibility: hidden;
    }
  }
}
```

理清思路然后一头扎进去，真正去实现的时候其实会发现没有那么难。
