[javascript - useMemo vs. useEffect + useState - Stack Overflow](https://stackoverflow.com/questions/56028913/usememo-vs-useeffect-usestate)

[useMemo vs useState and useEffect | by Zoe Katz | Medium](https://medium.com/@zoenkatz/usememo-vs-usestate-and-useeffect-d906901d88c)

this version renders two times:
```js
const [data, setData] = useState(initialVal);
useEffect(()=>{
  setData(data)
}, [dep])
```

this version renders only once:
```js
const data = useMemo(()=>{return data}, [dep])
```

useState 提供更改state的方法，useMemo只能监听依赖值改变
