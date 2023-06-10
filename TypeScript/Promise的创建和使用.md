# Promise的创建和使用

```typescript

function add(a:number,b:number) :Promise<number>{
    //resolve  真正的执行回调
    //reject  异常回调
    return new Promise((reslove,reject)=>{
        if(b % 17 === 0){
            reject(`bad number ${b}`)
        }
        setTimeout(()=>{
            reslove(a+b)
        }
            , 2000
        )
    })
}

//只有返回值为Promise才能一直then下去
add(1,2).then(res=>add(res,3)).then(res=>add(res,17)).catch((err)=>console.log(err))
```

-   同时等待多个Promise

```typescript
function add(a:number,b:number) :Promise<number>{
    return new Promise((reslove,reject)=>{
        if(b % 17 === 0){
            reject(`bad number ${b}`)
        }
        setTimeout(()=>{
            reslove(a+b)
        }
            , 2000
        )
    })
}
function mul(a:number,b:number):Promise<number>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve(a*b)
        },3000)
    })
}
//同时等待多个Promise
//(2+3)*(4+5)
Promise.all([add(2,3),add(4,5)]).then((res=>{
   let [a,b] = res
   return mul(a,b)
})).then((res)=>{
    console.log(res)
})

```

```typescript
// 多个Promise比赛，看谁先执行，且只执行最先完成那个
Promise.race([add(1,2),add(3,4)]).then((res)=>{
    console.log(res)
})
```

-   async、await语法糖

```typescript
//async awite 语法糖
async function cal(){
    //等待完成之后才进行下面的操作
    let [a,b] = await Promise.all([add(1,2),add(3,4)])
    return mul(a,b)
}
cal().then(res =>{
    console.log(res)
})
```
