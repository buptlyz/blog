---
title: rc-notification源码解析
date: 2019-04-22 22:57:22
tags: rc-component
---

[github地址](https://github.com/react-component/notification)
[示例地址](http://react-component.github.io/notification/examples/simple.html)

## [Notification](https://github.com/react-component/notification/blob/master/src/Notification.jsx)

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import ReactDOM from 'react-dom';
import Animate from 'rc-animate';
import createChainedFunction from 'rc-util/lib/createChainedFunction';
import classnames from 'classnames';
import Notice from './Notice';

// 用来生成唯一标识符uuid
let seed = 0;
const now = Date.now();

function getUuid() {
  return `rcNotification_${now}_${seed++}`;
}

// Notification -> Animate -> Notices
class Notification extends Component {
  static propTypes = {
    prefixCls: PropTypes.string,
    transitionName: PropTypes.string,
    animation: PropTypes.oneOfType([PropTypes.string, PropTypes.object]),
    style: PropTypes.object,
    maxCount: PropTypes.number,
    closeIcon: PropTypes.node,
  };

  static defaultProps = {
    prefixCls: 'rc-notification',
    animation: 'fade',
    style: {
      top: 65,
      left: '50%',
    },
  };

  state = {
    notices: [],
  };

  // transitionName
  getTransitionName() {
    const props = this.props;
    let transitionName = props.transitionName;
    if (!transitionName && props.animation) {
      transitionName = `${props.prefixCls}-${props.animation}`;
    }
    return transitionName;
  }

  // 添加notice
  add = (notice) => {
    const key = notice.key = notice.key || getUuid();
    const { maxCount } = this.props;
    this.setState(previousState => {
      const notices = previousState.notices;
      const noticeIndex = notices.map(v => v.key).indexOf(key);
      const updatedNotices = notices.concat();

      // 已存在key值相同的notice，则用新的替换旧的
      if (noticeIndex !== -1) {
        updatedNotices.splice(noticeIndex, 1, notice);
      } else {
        // 超出notice的maxCount，则先删除第一个，再添加到队列末尾
        // 没有超出则直接添加到队列末尾
        if (maxCount && notices.length >= maxCount) {
          // 使用第一项的key来更新新添加的项（让react移除现有的，而不是先删除再挂载）。
          // 复用旧的key值是为了：a) 外部人工控制；b)内部react的key属性不是很好用。

          // XXX, use key of first item to update new added (let React to move exsiting
          // instead of remove and mount). Same key was used before for both a) external
          // manual control and b) internal react 'key' prop , which is not that good.
          notice.updateKey = updatedNotices[0].updateKey || updatedNotices[0].key;
          updatedNotices.shift();
        }
        updatedNotices.push(notice);
      }
      return {
        notices: updatedNotices,
      };
    });
  }

  // 删除特定key的notice
  remove = (key) => {
    this.setState(previousState => {
      return {
        notices: previousState.notices.filter(notice => notice.key !== key),
      };
    });
  }

  render() {
    const props = this.props;
    const { notices } = this.state;

    // notice节点，每个notice对应一个Notice组件实例
    const noticeNodes = notices.map((notice, index) => {
      const update = Boolean(index === notices.length - 1 && notice.updateKey);
      const key = notice.updateKey ? notice.updateKey : notice.key;
      const onClose = createChainedFunction(this.remove.bind(this, notice.key), notice.onClose);
      return (<Notice
        prefixCls={props.prefixCls}
        {...notice}
        key={key}
        update={update}
        onClose={onClose}
        onClick={notice.onClick}
        closeIcon={props.closeIcon}
      >
        {notice.content}
      </Notice>);
    });
    const className = {
      [props.prefixCls]: 1,
      [props.className]: !!props.className,
    };
    return (
      <div className={classnames(className)} style={props.style}>
        <Animate transitionName={this.getTransitionName()}>{noticeNodes}</Animate>
      </div>
    );
  }
}

// 静态方法，创建Notification实例
Notification.newInstance = function newNotificationInstance(properties, callback) {

  // getContainer是Notification instance的容器，最终append到body里
  const { getContainer, ...props } = properties || {};
  const div = document.createElement('div');
  if (getContainer) {
    const root = getContainer();
    root.appendChild(div);
  } else {
    document.body.appendChild(div);
  }
  let called = false;

  // 拿到notification的ref
  // 包装成{notice, removeNotice, destroy, component}，并传给callback
  function ref(notification) {
    if (called) {
      return;
    }
    called = true;
    callback({
      notice(noticeProps) {
        notification.add(noticeProps);
      },
      removeNotice(key) {
        notification.remove(key);
      },
      component: notification,
      destroy() {
        ReactDOM.unmountComponentAtNode(div);
        div.parentNode.removeChild(div);
      },
    });
  }
  ReactDOM.render(<Notification {...props} ref={ref} />, div);
};

export default Notification;
```

## [Notice](https://github.com/react-component/notification/blob/master/src/Notice.jsx)

```jsx
import React, { Component } from 'react';
import classNames from 'classnames';
import PropTypes from 'prop-types';

/** 
* 1. 展示notice内容，关闭icon
* 2. 点击事件、关闭事件
* 3. 持续时间后调用`props.onClose()`
*/
export default class Notice extends Component {
  static propTypes = {
    duration: PropTypes.number,
    onClose: PropTypes.func,
    children: PropTypes.any,
    update: PropTypes.bool,
    closeIcon: PropTypes.node,
  };

  static defaultProps = {
    onEnd() {
    },
    onClose() {
    },
    duration: 1.5,
    style: {
      right: '50%',
    },
  };

  componentDidMount() {
    this.startCloseTimer();
  }

  componentDidUpdate(prevProps) {
    if (this.props.duration !== prevProps.duration
      || this.props.update) {
      this.restartCloseTimer();
    }
  }

  componentWillUnmount() {
    this.clearCloseTimer();
  }

  close = (e) => {
    if (e) {
      e.stopPropagation();
    }
    this.clearCloseTimer();
    this.props.onClose();
  }

  startCloseTimer = () => {
    if (this.props.duration) {
      this.closeTimer = setTimeout(() => {
        this.close();
      }, this.props.duration * 1000);
    }
  }

  clearCloseTimer = () => {
    if (this.closeTimer) {
      clearTimeout(this.closeTimer);
      this.closeTimer = null;
    }
  }

  restartCloseTimer() {
    this.clearCloseTimer();
    this.startCloseTimer();
  }

  render() {
    const props = this.props;
    const componentClass = `${props.prefixCls}-notice`;
    const className = {
      [`${componentClass}`]: 1,
      [`${componentClass}-closable`]: props.closable,
      [props.className]: !!props.className,
    };
    return (
      <div
        className={classNames(className)}
        style={props.style}
        onMouseEnter={this.clearCloseTimer}
        onMouseLeave={this.startCloseTimer}
        onClick={props.onClick}
      >
        <div className={`${componentClass}-content`}>{props.children}</div>
          {props.closable ?
            <a tabIndex="0" onClick={this.close} className={`${componentClass}-close`}>
              {props.closeIcon || <span className={`${componentClass}-close-x`}/>}
            </a> : null
          }
      </div>
    );
  }
}
```

## 用法

```jsx
/* eslint-disable no-console */
import 'rc-notification/assets/index.css';
import Notification from 'rc-notification';
import React from 'react';
import ReactDOM from 'react-dom';
let notification = null;
Notification.newInstance({}, (n) => notification = n);

function simpleFn() {
  notification.notice({
    content: <span>simple show</span>,
    onClose() {
      console.log('simple close');
    },
  });
}

function durationFn() {
  notification.notice({
    content: <span>can not close...</span>,
    duration: null,
  });
}

function closableFn() {
  notification.notice({
    content: <span>closable</span>,
    duration: null,
    onClose() {
      console.log('closable close');
    },
    closable: true,
    onClick() {
      console.log('clicked!!!');
    },
  });
}

function close(key) {
  notification.removeNotice(key);
}

function manualClose() {
  const key = Date.now();
  notification.notice({
    content: <div>
      <p>click below button to close</p>
      <button onClick={close.bind(null, key)}>close</button>
    </div>,
    key,
    duration: null,
  });
}

let counter = 0;
let intervalKey;
function updatableFn() {
  if (counter !== 0) {
    return;
  }

  const key = 'updatable-notification';
  const initialProps = {
    content: `Timer: ${counter}s`,
    key,
    duration: null,
    closable: true,
    onClose() {
      clearInterval(intervalKey);
      counter = 0;
    },
  };

  notification.notice(initialProps);
  intervalKey = setInterval(() => {
    notification.notice({ ...initialProps, content: `Timer: ${++counter}s` });
  }, 1000);
}

let notification2 = null;
const clearPath = 'M793 242H366v-74c0-6.7-7.7-10.4-12.9' +
  '-6.3l-142 112c-4.1 3.2-4.1 9.4 0 12.6l142 112c' +
  '5.2 4.1 12.9 0.4 12.9-6.3v-74h415v470H175c-4.4' +
  ' 0-8 3.6-8 8v60c0 4.4 3.6 8 8 8h618c35.3 0 64-' +
  '28.7 64-64V306c0-35.3-28.7-64-64-64z';

const getSvg = (path, props = {}, align = false) => {
  return (
    <i {...props}>
      <svg
        viewBox="0 0 1024 1024"
        width="1em"
        height="1em"
        fill="currentColor"
        style={align ? { verticalAlign: '-.125em ' } : {}}
      >
        <path d={path} />
      </svg>
    </i>
  );
};
Notification.newInstance({
  closeIcon: getSvg(clearPath, {}, true),
}, (n) => {
  notification2 = n;
});
function customCloseIconFn() {
  notification2.notice({
    content: 'It is using custom close icon...',
    closable: true,
    duration: 0,
  });
}

ReactDOM.render(<div>
  <div>
    <button onClick={simpleFn}>simple show</button>
    <button onClick={durationFn}>duration=0</button>
    <button onClick={closableFn}>closable</button>
    <button onClick={manualClose}>controlled close</button>
    <button onClick={updatableFn}>updatable</button>
    <div>
      <button onClick={customCloseIconFn}>custom close icon</button>
    </div>
  </div>
</div>, document.getElementById('__react-content'));
```

