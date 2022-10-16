# 顶部面包屑的折叠省略效果(hidden DOM && useResizeObserver)

## 官方地址
[use-resize-observer](https://github.com/ZeeCoder/use-resize-observer#readme)
## 安装
```
yarn add use-resize-observer --dev
# or
npm install use-resize-observer --save-dev
```

## 具体实现
```jsx
import useResizeObserver from "@react-hook/resize-observer";

function useSize(target) {
    const [size, setSize] = React.useState();

    React.useLayoutEffect(() => {
        target && setSize(target.getBoundingClientRect());
    }, [target]);

    useResizeObserver(target, (entry) => setSize(entry.contentRect));
    return size;
}

function Pathtree(type, pathdata,nodeid) {
    const [target, setTarget] = React.useState();
    const size = useSize(target);
    const [judge,setJudge] = useState(null);
    useEffect(() => {
        setJudge(document.getElementById("judge"));
    },[size]);
  if (pathdata) {
    const pathtree = [...pathdata.path].reverse().map((content,index) => (
        index !== 0 && <span>
        <NavLink className="path-name" to={routeUrls("category", content.id)}>
          {content.name}
        </NavLink>{" "}
            {index !==pathdata.path.length - 1 && <span className="right-angle"> > </span>}
      </span>
    ));
      return (
        <div>
          <div id="judge" className="path-tree-judge" ref={setTarget}>   {/*hidden DOM，获取引用*/}
              <span>
                 <NavLink
                     className="path-name"
                     to="/"
                 >
                  Home
              </NavLink>
              </span>
              {pathdata.path.length !== 1 && <span className="right-angle"> > </span>}
                {pathtree}  {/*中间路径*/}
              <span className="right-angle"> > </span>
            <span>
              <NavLink
                className="path-name"
                to={routeUrls("category", nodeid)} {/*自定义生成url*/}
              >
                {pathdata.category.name}  {/*当前路径*/}
              </NavLink>
            </span>
          </div>
            {!!judge && (judge.offsetHeight <= 50 ?  //根据hidden DOM的宽高判断有无换行
                <div className="path-tree">
                    <span>
                 <NavLink
                     className="path-name"
                     to="/"
                 >
                  Home
              </NavLink>
              </span>
                    {pathdata.path.length !== 1 && <span className="right-angle"> > </span>}
                    {pathtree} {/*中间路径*/}
                    <span className="right-angle"> > </span>
            <span>
              <NavLink
                className="path-name"
                to={routeUrls("category", nodeid)}  {/*自定义生成url*/}
              >
                {pathdata.category.name}  {/*当前路径*/}
              </NavLink>
            </span>
          </div>:
                <div className="path-tree">
                    <span>
                      <NavLink
                          className="path-name"
                          to="/"
                      >
                         Home
                      </NavLink>
                    </span>
                    <span className="right-angle"> > </span>
                    <span className="right-angle" style={{cursor:"pointer"}}><Popover placement="bottom" content={pathtree} trigger="hover">......</Popover></span> {/*折叠隐藏中间路径*/}
                    <span className="right-angle"> > </span>
                    <span>
                      <NavLink
                          className="path-name"
                          to={routeUrls("category", nodeid)}  {/*自定义生成url*/}
                      >
                          {pathdata.category.name}  {/*当前路径*/}
                      </NavLink>
                    </span>
            </div>)}
        </div>
      );
    }
}
```
