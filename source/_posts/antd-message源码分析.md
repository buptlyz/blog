---
title: antd message源码分析
date: 2019-04-22 22:20:57
tags:
---

[github地址](https://github.com/ant-design/ant-design/blob/master/components/message/index.tsx)。
调用了 [`rc-notification`](https://github.com/react-component/notification) 模块。

```typescript
/* global Promise */
import * as React from 'react';
import Notification from 'rc-notification';
import Icon from '../icon';

let defaultDuration = 3;            // 持续时间 s
let defaultTop: number;             // css top值
let messageInstance: any;           // message实例
let key = 1;                        // 自增key
let prefixCls = 'ant-message';      // prefix cls
let transitionName = 'move-up';     // transition
let getContainer: () => HTMLElement;// container
let maxCount: number;               // message个数阈值

/** 
* 如果有 messageInstance 则 callback(messageInstance)
* 否则创建一个 messageInstance 再 callback(messageInstance)
* @param {any} callback
*/
function getMessageInstance(callback: (i: any) => void) {
  if (messageInstance) {
    callback(messageInstance);
    return;
  }
  Notification.newInstance({
    prefixCls,
    transitionName,
    style: { top: defaultTop }, // 覆盖原来的样式
    getContainer,
    maxCount,
  }, (instance: any) => {
    if (messageInstance) {
      callback(messageInstance);
      return;
    }
    messageInstance = instance;
    callback(instance);
  });
}

type NoticeType = 'info' | 'success' | 'error' | 'warning' | 'loading';

export interface ThenableArgument {
  (_: any): any;
}

export interface MessageType {
  (): void;
  then: (fill: ThenableArgument, reject: ThenableArgument) => Promise<any>;
  promise: Promise<any>;
}

/**
* @param {React.ReactNode} content message组件的content
* @param {func | number} duration 持续时间
* @param {NoticeType} type 类型
*/
function notice(
  content: React.ReactNode,
  duration: (() => void) | number = defaultDuration,
  type: NoticeType,
  onClose?: () => void,
): MessageType {
  const iconType = ({
    info: 'info-circle',
    success: 'check-circle',
    error: 'cross-circle',
    warning: 'exclamation-circle',
    loading: 'loading',
  })[type];

  if (typeof duration === 'function') {
    onClose = duration;
    duration = defaultDuration;
  }

  const target = key++;

  // 调用 messageInstance，并将 onClose 作为 callback 传进去
  const closePromise = new Promise((resolve) => {
    const callback =  () => {
      if (typeof onClose === 'function') {
        onClose();
      }
      return resolve(true);
    };
    getMessageInstance((instance) => {
      instance.notice({
        key: target,
        duration,
        style: {},
        content: (
          <div className={`${prefixCls}-custom-content ${prefixCls}-${type}`}>
            <Icon type={iconType} />
            <span>{content}</span>
          </div>
        ),
        onClose: callback,
      });
    });
  });
  const result: any = () => {
    if (messageInstance) {
      messageInstance.removeNotice(target);
    }
  };
  result.then = (filled: ThenableArgument, rejected: ThenableArgument) => closePromise.then(filled, rejected);
  result.promise = closePromise;

  // 返回remove notice的函数
  return result;
}

type ConfigContent = React.ReactNode | string;
type ConfigDuration = number | (() => void);
export type ConfigOnClose = () => void;

export interface ConfigOptions {
  top?: number;
  duration?: number;
  prefixCls?: string;
  getContainer?: () => HTMLElement;
  transitionName?: string;
  maxCount?: number;
}

// 入口
export default {
  info(content: ConfigContent, duration?: ConfigDuration, onClose?: ConfigOnClose) {
    return notice(content, duration, 'info', onClose);
  },
  success(content: ConfigContent, duration?: ConfigDuration, onClose?: ConfigOnClose) {
    return notice(content, duration, 'success', onClose);
  },
  error(content: ConfigContent, duration?: ConfigDuration, onClose?: ConfigOnClose) {
    return notice(content, duration, 'error', onClose);
  },
  // Departed usage, please use warning()
  warn(content: ConfigContent, duration?: ConfigDuration, onClose?: ConfigOnClose) {
    return notice(content, duration, 'warning', onClose);
  },
  warning(content: ConfigContent, duration?: ConfigDuration, onClose?: ConfigOnClose) {
    return notice(content, duration, 'warning', onClose);
  },
  loading(content: ConfigContent, duration?: ConfigDuration, onClose?: ConfigOnClose) {
    return notice(content, duration, 'loading', onClose);
  },
  config(options: ConfigOptions) {
    if (options.top !== undefined) {
      defaultTop = options.top;
      messageInstance = null; // delete messageInstance for new defaultTop
    }
    if (options.duration !== undefined) {
      defaultDuration = options.duration;
    }
    if (options.prefixCls !== undefined) {
      prefixCls = options.prefixCls;
    }
    if (options.getContainer !== undefined) {
      getContainer = options.getContainer;
    }
    if (options.transitionName !== undefined) {
      transitionName = options.transitionName;
      messageInstance = null; // delete messageInstance for new transitionName
    }
    if (options.maxCount !== undefined) {
      maxCount = options.maxCount;
      messageInstance = null;
    }
  },
  destroy() {
    if (messageInstance) {
      messageInstance.destroy();
      messageInstance = null;
    }
  },
};
```
