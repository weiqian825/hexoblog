---
title: banner轮播图
date: 2018-10-22 22:01:15
categories: 
- shopee
tags:
- banner
- componets
---
## 一、banner ui部分
1. UI部分BannerItem和BannerDots banerItem.js
```
import React, { Component } from 'react'
import classNames from 'classnames'
import style from './style.scss'

export const BannerItem = ({ count, item }) => (
  <li className={style.bannerItem} style={{ width: (100 / count + '%') }}>
    <a href={item.link || 'javascript:;'}> <img src={item.src} alt={item.alt || ''} /></a>
  </li>
)
export class BannerDots extends Component {
  render () {
    const { count, curIndex } = this.props
    let dotNodes = []
    for (let i = 0; i < count; i++) {
      dotNodes[i] = (
        <span
          key={i}
          className={classNames({
            [style.bannerDots]: true,
            [style.bannerDotsSelected]: i + 1 === curIndex
          })}
        />
      )
    }
    return (
      <div className={style.bannerDotsWrap}>
        {dotNodes}
      </div>
    )
  }
}

```
2. scss
```
*{
  margin: 0;
  padding: 0;
  touch-action: pan-y;
}

.banner {
  position: relative;
  overflow: hidden;
  margin-bottom: 10px;

  .bannerItem {
    display: inline-block;
    &>a>img {
      display: block;
      height: calc(25vw);
      width: 100%;
    }
  }
}

.bannerDotsWrap {
  position: absolute;
  z-index: 2;
  margin-left: 12px;
  margin-bottom: 10px;
  width: 100%;
  bottom: 0;
  
  .bannerDots {
    display: inline-block;
    width: 6px;
    border: 1px solid white;
    margin-right: 6px;
    cursor: pointer;
    &.bannerDotsSelected {
      margin-bottom: -2px;
      width: 4px;
      height: 4px;
      border: 1px  solid white;
      border-radius: 4px;
    }
  }
}
```
## 二、banner 轮播部分逻辑
1. Banner.js
```
import React, { Component } from 'react'
import { BannerItem, BannerDots } from './BannerItem'
import style from './style.scss'
export default class Banner extends Component {
  constructor (props) {
    super(props)
    const { sourceList, duration } = this.props
    const initTransit = 'all ' + duration + 's ease '
    const sourceCount = sourceList.length
    const itemList = [sourceList[sourceCount - 1]].concat(sourceList).concat([sourceList[0]])
    const itemCount = itemList.length

    this.state = {
      index: 1,
      move: -100 / itemCount,
      transition: initTransit,
      itemList: itemList,
      itemCount: itemCount,
      sourceCount: sourceCount,
      initTransit: initTransit
    }

    this.startPos = null
    this.endPos = null
    this.isScrolling = 0
  }

  componentDidMount () {
    this.autoPlay()
  }

  turn = (step) => {
    const { index, itemCount, sourceCount, initTransit } = this.state
    const { duration } = this.props
    let curIndex = index + step
    let move = -curIndex * 100 / itemCount
    this.setState({
      move: move,
      transition: initTransit
    })
    if (curIndex < 1 || curIndex > sourceCount) {
      if (curIndex < 1) {
        curIndex = curIndex + sourceCount
      } else if (curIndex > sourceCount) {
        curIndex = curIndex - sourceCount
      }
      move = -curIndex * 100 / itemCount
      setTimeout(() => {
        this.setState({ move: move, transition: 'none' })
      }, duration * 1000)
    }
    this.setState({
      index: curIndex
    })
  }

  autoPlay = () => {
    if (this.props.autoplay) {
      this.autoPlayFlag = setInterval(() => {
        this.turn(1)
      }, this.props.delay * 1000)
    }
  }

  pausePlay = () => {
    clearInterval(this.autoPlayFlag)
  }

  onTouchStart = (event) => {
    this.pausePlay()
    let touch = event.targetTouches[0]
    this.startPos = { x: touch.pageX, y: touch.pageY, time: +new Date() }
  }

  onTouchMove = (event) => {
    if (event.targetTouches.length > 1 || (event.scale && event.scale !== 1)) return
    let touch = event.targetTouches[0]
    this.endPos = { x: touch.pageX - this.startPos.x, y: touch.pageY - this.startPos.y }
    this.isScrolling = Math.abs(this.endPos.x) < Math.abs(this.endPos.y) ? 1 : 0
    if (this.isScrolling === 0) {
      event.preventDefault()
    }
  }

  onTouchEnd = (event) => {
    let duration = +new Date() - this.startPos.time
    if (this.isScrolling === 0) {
      if (Number(duration) > 10) {
        if (this.endPos && this.endPos.x > 10) {
          this.turn(-1)
        } else if (this.endPos && this.endPos.x < -10) {
          this.turn(1)
        }
      }
    }
    this.autoPlay()
  }

  render () {
    const { sourceCount, itemCount, itemList, move, transition, index } = this.state
    const { pause } = this.props
    const curStyle = {
      transform: 'translateX(' + (move) + '%)',
      transition: transition,
      width: itemCount * 100 + '%'
    }
    return sourceCount ? (
      <div
        className={style.banner}
        onMouseOver={pause ? this.pausePlay : null}
        onMouseOut={pause ? this.autoPlay : null}>
        <ul style={curStyle}
          onTouchStart={this.onTouchStart}
          onTouchMove={this.onTouchMove}
          onTouchEnd={this.onTouchEnd}
        >
          { itemList.map((item, idx) => {
            return (<BannerItem
              item={item}
              count={itemCount}
              key={'item' + idx} />)
          })}
        </ul>
        {<BannerDots count={sourceCount} curIndex={index} />}
      </div>
    ) : null
  }
}

Banner.defaultProps = {
  duration: 1,
  delay: 3,
  pause: false,
  autoplay: true,
  dots: true,
  sourceList: []
}
Banner.autoPlayFlag = null

```
## 三、banner 数据请求逻辑部分
1. index.js
```
import React, { Component } from 'react'
import { bannerApi } from '@api/index.js'
import Banner from './Banner.js'

class ShowBanner extends Component {
  constructor (props) {
    super(props)
    this.state = {
      itemList: []
    }
  }

  componentDidMount () {
    bannerApi.fetchBannerInfo({ query: {} }).then((data) => {
      if (data && data.data[0] && data.data[0].imgInfo) {
        const itemList = data.data[0].imgInfo.map((item) => {
          return {
            src: item.location,
            link: item.url,
            alt: 'images' + item.ordinal
          }
        })
        this.setState({
          itemList: itemList
        })
      }
    })
  }

  render () {
    const { itemList } = this.state
    return itemList.length ? (
      <Banner
        sourceList={itemList}
        duration={2}
        delay={3}
        autoplay
        dots
      />
    ) : null
  }
}

export default ShowBanner

```
## 四、调用demo
```
import React from 'react'
import ReactDom from 'react-dom'
import style from './index.scss'
import ShowBanner from '@comp/Banner/index.js'

ReactDom.render(
  (<div>
    <h1 className={style.main}>Banner Componets</h1>
    <ShowBanner />
  </div>),
  document.getElementById('root')
)

```
