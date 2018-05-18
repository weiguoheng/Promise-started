# Promise-started
## Promise基础

为什么要用Promise？node.js是异步执行，哪个快，先执行哪个。

```
//例1：
console.log('start!');
console.log('xiaohong:1');
setTimeout(()=>{
    console.log('xiaoming:2')
},2000);
setTimeout(()=>{
    console.log('xiaoqiang:3');
},1000);

//result:
start!
xiaohong:1
xiaoqiang:3
xiaoming:2
```
假如我们希望输出顺序是xiaohong:1->xiaoming:2->xiaoqiang:3。就需要用到Promise

```
//例2：
console.log('start!');
console.log('xiaohong:1');
new Promise((resolve,reject)=>{
    setTimeout(()=>{
        console.log('xiaoming:2');
        resolve('anything');
    },2000);
}).then(val=>{
    setTimeout(()=>{
        console.log('xiaoqiang:3');
    },1000);
});

//result:
start!
xiaohong:1
xiaoming:2
xiaoqiang:3

```
  Promise是一个构造函数。
  resolve表示成功后的回调，只有当resolve有回调值的时候，Promise下面的.then才会继续执行。
  resolve的值会在.then的形参里。
  Promise后面可以无限加.then。第一个.then里面的形参是resolve过来的值。后面的.then里面输出的是上一级return过来的数据。
  
  
如果在.then里面有需要上一个.then执行完的成功回调的话。上一个.then如果不是立即执行的，那么在下一个.then的形参将获取不到值。例如：
```
//例3：
new Promise((resolve,reject)=>{
  setTimeout(()=>{
      console.log('xiaoming:2');
      resolve('anything');
  },2000);
}).then(val_1=>{
  setTimeout(()=>{
      console.log('xiaoqiang:3');
      return 'xiaoqiang3'
  },1000);
}).then(val_2=>{
  console.log('val_2',val_2)
});

//result:
xiaoming:2
val_2 undefined
xiaoqiang:3
```
由此可知，最后一个.then没有等上一个.then执行完就已经开始执行了。
解决办法，在.then里再return 一个 Promise
```
//例4：
new Promise((resolve,reject)=>{
    setTimeout(()=>{
        console.log('xiaoming:2');
        resolve('anything');
    },2000);
}).then(val_1=>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            console.log('xiaoqiang:3');
            resolve('xiaoqiang3')
        },1000);
    })
}).then(val_2=>{
    console.log('val_2',val_2)
});

//result:
xiaoming:2
xiaoqiang:3
val_2 xiaoqiang3
```
例4和例3不同的是，val_2不是undefined了。最后一个.then等到上一个.then执行结束才执行的;

reject什么用呢？resolve是成功回调的话，reject就是失败回调。
```
//例5:
new Promise((resolve,reject)=>{
    const random = parseInt(Math.random()*10);
    if(random<5){
        resolve(random);
    }else{
        reject(random)
    }
}).then(val=>{
    console.log('success',val)
}).catch(err=>{
    console.log('err',err)
});

//result
err 6||success 4
```
例5里面，生成一个1-10的随机数。如果小于5，那么，值会传给.then的第一个函数，如果值不小于于5，那么，值会通过reject传给.then的第二个函数

.catch里的函数也可以写在.then的第二个函数位置，效果是一样的
```
//例6：
new Promise((resolve,reject)=>{
    const random = parseInt(Math.random()*10);
    if(random<5){
        resolve(random);
    }else{
        reject(random)
    }
}).then(val=>{
    console.log('success',val)
},err=>{
    console.log('err',err);
});

//result
err 6||success 4
```
