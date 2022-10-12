# 自定义触发式Modal函数组件(React)
## 常见的组件库([Antd-Modal](https://ant.design/components/modal-cn/))
开发过程中，有些业务场景需要实现modal组件，点击一个按钮然后展开一个modal，
modal上面可能会展示一些数据内容，或者有一些数据交互的通道。
Ant Design作为基于React的前端组件库集成好了modal这么一个对话框组件。
但是其首推的使用方式是一种预埋式的调用方法，首先在需要用到modal的页面内写入`<Modal/>`组件，然后设置state`const [isModalVisible,setIsModalVoisbile] = useState(false);`,
通过`isModalVisbile`这个state去控制modal的显示，一般会在点击的按钮中设置`setIsModalVisible(true)` 。
## 存在的问题
### 实现模式与业务逻辑不符
正常的业务逻辑应该是当用户点击某个按钮后，创建并显示出一个对话框modal，这个modal里的内容可以自定。而上述的方法则是预先创建好modal，设置好modal里展示的内容，然后把modal隐藏，当用户点击按钮后，把隐藏改变为显示，这与一般的业务逻辑有出入。
### 同一页面下的多个modal会产生多个state
上述方法通过state控制modal的显示与否，当某个业务场景下，同一页面内有多种modal需要展示，那就需要预埋多种modal，并设置数量与之相对应的state去控制各个modal的显示。
### 增加不必要的数据获取次数
因为是预埋式的modal，modal里展示的数据我们需要在用户实际点开之前就预先获取好并完成设置。如果用户在本页面不点开modal，那么就会浪费这样一次获取数据的操作。这样的方式无法实现当用户点击按钮后，再去从后端获取数据并展示的业务流程。
### 无法在Service层中调用
一些业务场景下，我们可能会需要在service层弹出一个modal去获取用户的某些数据以完成后续操作。这种情况下，Service层没有办法预埋modal组件，而且Components中的state也与Service割裂开来，无法简单实现二者之间的关联。
## 解决方案
除了上述的方式调用modal以外，antd还推出了另一种函数语法糖的方式展开一个modal，如[Modal.method()](https://ant.design/components/modal-cn/#Modal.method()) ，但这种方式并不是主流方式，所以有些时候不能完全满足开发者的需求，为此可以自定义一个函数触发式的modal组件。
这种函数式的modal组件，不需要事先进行预埋，只需要在某个按钮的OnClick中调用该modal函数，便能创建并展开一个modal对话框。   
例如： 
``` jsx
<button onClick={() => {  //设置OnClick方法
    httpclient.getModalData().then((data) => {  //先获取modal需要的数据
        PopModal(config); //调用PopModal函数，并将获取到的data处理成config传入
    })}                      
}></button>  //PopModal为完全受控组件，本身不具备任何数据
```
###具体实现
PopModal.js
```jsx
import React from "react";
import "./PopModal.scss"; //css文件，请自行设计样式
import ReactDOM from 'react-dom';
import {CloseCircleOutlined} from '@ant-design/icons';

function PopModal(config) { //接受config作为参数，决定显示内容
    const modalRoot = document.createElement('div');  //利用原生js创建modal根结点
    document.body.appendChild(modalRoot); //创建dom元素
    const Modal = () => {  //自定义modal样式与内容，以下仅供参考
        return (
            <div className="modal-wrapper">
                <div className="modal-layer"></div>
                <div style={{width: config.width + "px",left: "calc(50vw - "+config.width/2 +"px)",top:config.top + "px",borderRadius:config.borderRadius + "px"}} className="modal-card">
                    {!!config.closeButtonPosition === "top" ? <div className="close-button-top" onClick={() =>{if(config.onClose){config.onClose()}ReactDOM.unmountComponentAtNode(modalRoot);document.body.removeChild(modalRoot)}}><CloseCircleOutlined /></div>
                    : <div className="close-button-bottom" style={{left: "calc("+config.width/2 +"px - 10px)"}} onClick={() =>{if(config.onClose){config.onClose()}ReactDOM.unmountComponentAtNode(modalRoot);document.body.removeChild(modalRoot)}}><CloseCircleOutlined /></div>}
                    {!!config.title && <div className="modal-title"><div className="modal-title-message">{config.title}</div></div>}
                    {!!config.body && <div className="modal-body"><div>{config.body}</div><div onClick={() => {if(config.onClose){config.onClose()}ReactDOM.unmountComponentAtNode(modalRoot);document.body.removeChild(modalRoot)}}>{config.closeText}</div></div>}
                    {!!(config.onCancel || config.onOk) && <div className="modal-button">
                        {!!config.onCancel && <div className="cancel-button" onClick={() => {if(config.onClose){config.onClose()};config.onCancel();ReactDOM.unmountComponentAtNode(modalRoot);document.body.removeChild(modalRoot)}}>{config.cancelText || "Cancel"}</div>}
                        {!!config.onOk && <div className="ok-button" onClick={() => {if(config.onClose){config.onClose()};config.onOk();ReactDOM.unmountComponentAtNode(modalRoot);document.body.removeChild(modalRoot)}}>{config.okText || "Ok"}</div>}
                    </div>}
                </div>
            </div>
        )
    }
    ReactDOM.render(<Modal/>, modalRoot); //通过react mount组件
}

export default PopModal;
```


