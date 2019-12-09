# useEffect

## useLayoutEffect

与useEffect不同，useLayoutEffect会在dom渲染之前执行

useLayoutEffect相当于是同步函数，因此不能传入异步匿名函数，他会阻塞渲染，因此只有在必要时再使用

可以使用useLayoutEffect来防止抖动

