# react-dnd 拖拽能力教程

## 前言

近几年来，低代码、零代码的热度在国内逐年递增。“复杂度同力一样不会消失，也不会凭空产生，它总是从一个物体转移到另一个物体或一种形式转为另一种形式”。用户在使用低零代码构建应用程序时，这些能力只是被平台研发人员提前编写完了。

作为低零代码的基础，前端拖拽能力就尤为重要。

一个完整的拖拽流程分为两部分：拖动+放置

让一个元素支持拖动是一件非常容易做到的事情，只需要在对应的 dom 节点增加 `draggable="true"` 的属性即可。

真正麻烦的是放置的能力。我们需要监听 `ondragstart`、`ondragenter`、`ondragover` 等各个阶段的事件，处理起来过分的繁琐。而社区已经提供了成熟的库 `react-dnd` 来帮助我们实现这些细节，我们只需要关心业务逻辑即可。

本文将手把手带着小伙伴们掌握拖拽能力，并且提供 demo 让小伙伴们能够更细致的研究。

[!img](img/img1.gif)

## 创建拖拽容器

首先进入到项目中，安装该能力需要用到的依赖。

```bash
pnpm i react-dnd

pnpm i react-dnd-html5-backend
```

```js
import { HTML5Backend } from "react-dnd-html5-backend";
import { DndProvider } from "react-dnd";

<DndProvider backend={HTML5Backend}>
  <App />
</DndProvider>;
```

从 `react-dnd` 中导出 `DndProvider`，包裹要实现拖动和放置的区域。

初次使用 `dnd` 库的小伙伴肯定对 `react-dnd-html5-backend` 存在疑惑。并且不理解 `DndProvider` 为什么要传递一个 `backend`。这里来解释下什么是 `backend`。

因 `pc端` 和 `移动端` 的 dom 层存在不同的事件监听和处理方式。所以 dnd 将这部分单独抽出来。方便后续的扩展。

- react-dnd-html5-backend：用于控制 html5 事件的 backend。
- react-dnd-touch-backend：用于控制移动端 touch 事件的 backend。
- react-dnd-test-backend：用户可参考自定义的 backend。

接下来要创建一个拖动区和放置区：

`pages/dnd/index.jsx`

```js
// 忽略部分内容，小伙伴们自行补齐
import { CustDrag, CustDrop } from "@/components";

const dndList = [
  { label: "标签1", value: "值1" },
  { label: "标签2", value: "值2" },
  { label: "标签3", value: "值3" },
];

const DndPage = () => {
  const [list, setList] = useState(dndList);
  return (
    <DndProvider backend={HTML5Backend}>
      <div className={styles.center}>
        <span>请拖拽：</span>
        <div style={{ border: "1px solid #000", minHeight: "200px" }}>
          {list.map((item) => {
            return <CustDrag key={item?.value} data={item} />;
          })}
        </div>
        <div style={{ marginTop: "10px" }}>请放置：</div>
        <CustDrop onChange={dropChange} />
      </div>
    </DndProvider>
  );
};
```

## 拖动能力

接下来实现第一个核心能力：

`components/CustDrag/index.jsx`

```js
import { useDrag } from "react-dnd";

const CustDrag: FC<CustDragProps> = ({ data }) => {
  const [{ opacity }, dragRef] = useDrag({
    type: "Field",
    item: { ...data },
    collect: (monitor) => ({
      opacity: monitor.isDragging() ? 0.5 : 1,
    }),
  });

  return (
    <div ref={dragRef} style={{ opacity, cursor: "move" }}>
      {data?.label}
    </div>
  );
};
```

通过 gif 图得知，列表中的每一项都为一个可拖动项，因此我们要给每个数据都设置上拖动的属性和效果。

```js
const [collectedProps, dragRef] = useDrag({ type, item, canDrag, collect });
```

**通过 `useDrag` 生成的第二个参数 `dragRef` 指向某一个 `div`**。此 `div` 将会被赋予 `draggable=true` 的属性，同时被拖动时所发生的所有事件都会被监听。

想要获取监听后的信息，**只需要在 `collect` 参数里配置好即可在 `collectedProps` 获取到实时数据。**

比如说上述的代码中，通过 `monitor.isDragging()` 监听到拖动的状态，并且定义一个 `opacity` 的属性来代表样式的透明度。

接下来看参数传递。

- `type`: 自定义一个名称。拖动的 `type` 和放置的 `type` 保持一致。
- `item`:参数传递。拖动时的数据能够传递到放置区。
- `collect`: 收集监听整个拖动事件的状态数据，比如是否正在进行拖动、拖动偏移量等数据。可以通过[源代码](https://github.com/react-dnd/react-dnd/blob/main/packages/react-dnd/src/types/monitors.ts)获取完整的数据。
- `end`: 拖动结束时执行的方法。
- `canDrag`: 指定当前是否允许拖动。若希望始终允许被拖动，则可以忽略此方法。

[!img](img/img2.gif)

当我们定义的项可被拖动，且拖动时，有 `0.5` 的透明度时，就说明这部分的代码编写成功。

## 放置能力

接下来完成第二个核心内容：

这里笔者将分为几个小部分，小伙伴们按着步骤走，逻辑会更加清晰。

- 实现放置能力
- 实现放置区数据唯一性
- 实现被放置后的数据不展示在拖动区

`components/CustDrop/index.jsx`

### 实现放置能力

```js
// 伪代码，省略部分重复代码
import { useDrop } from "react-dnd";

const CustDrop: FC<CustDropProps> = ({ onChange }) => {
  const [value, setValue] = useState<any[]>([]);
  const [{ canDrop, isOver }, drop] = useDrop({
    accept: 'Field',
    drop: (item) => {
      const targetValue = [...value];
      targetValue.push(item);
      setValue(targetValue);
      onChange(targetValue);
    },
    collect: (monitor) => ({
      // 是否放置在目标上
      isOver: monitor.isOver(),
      // 是否开始拖拽
      canDrop: monitor.canDrop(),
    }),
  });

  // 展示拖动时的界面效果
  const showCanDrop = () => {
    if (canDrop && !isOver && !value.length) return <div>请拖拽到此处</div>;
  };

  const delItem = (ind: number) => {
    const newValue = [...value];
    newValue.splice(ind, 1);
    setValue(newValue);
    onChange(newValue);
  };

  // 展示值
  const showValue = () => {
    return value.map((item, index: number) => {
      return (
        <div key={item?.value}>
          {item?.label} <span onClick={() => delItem(index)}>删除</span>
        </div>
      );
    });
  };

  return (
    <div
      ref={drop}
      style={{ border: '1px solid #000', marginTop: '10px', minHeight: '200px', background: '#fff' }}
    >
      {showCanDrop()}
      {showValue()}
    </div>
  );
};
```

核心的代码是 `useDrop` 的使用。

- `accept` 接收对应的对应的拖动标识。
- `drop` 接受拖动传递过来的数据。
- `collect` 收集拖动事件在放置区的数据。比如：是否有成功的拖动到放置区上、是否已经开始拖动，距离放置区的坐标等。并且将监听的数据传递到 `useDrop` 第一个参数来。

通过抛出的 `isOver`、`canDrop` 来判断用户是否正在拖拽中。若用户在拖拽中，则在放置区展示 `请拖拽到此处` 的文字标识。

当完成上面的代码就实现了最简单的拖拽功能。并且增加 `删除` 的按钮实现放置区的新增和删除能力。

### 实现放置区数据唯一性

若我们要保证放置区数据的唯一性，我们就需要对正在拖拽的数据进行判断。有两种方案：

- 若拖拽了相同的数据到放置区，则放置区置红提示用户 `数据已经被放置`。并且不让用户将数据放置到放置区中。
- 当成功拖拽了某一个数据后，将该数据在拖动区中删除。

那么接下来按顺序实现上面的两种场景。

```js
const [error, setError] = useState < string > "";
const [{ canDrop, isOver }, drop] = useDrop({
  // 这里省略部分重复代码
  canDrop: (item: any) => {
    setError(undefined);
    const filter = value.filter((it) => it.value === item.value);
    if (!!filter.length) {
      setError("数据已经被放置");
      return false;
    }
    return true;
  },
});

const showCanDrop = () => {
  if (error && isOver) return <div>{error}</div>;
  if (canDrop && !isOver && !value.length) return <div>请拖拽到此处</div>;
};

return (
  <div
    ref={drop}
    style={{
      // ...省略部分代码
      background: error && isOver ? "red" : "#fff",
    }}
  >
    {showCanDrop()}
    {showValue()}
  </div>
);
```

通过 `canDrop` 重新自定义 `collect` 下的 `canDrop` 数据。当数据重复时，`return false` 并且设置错误的信息，让用户无法将重复的数据放到放置区。

当然界面上也会给出错误信息的提示，为了让错误信息展示得更醒目，会把背景也标记为红色。

[!img](img/img3.gif)

当然在某些场景下，我们只需要将拖拽出去的数据删除，就能保证数据的唯一性了。

这时组件传递的 `onChange` 方法就显得尤为重要。

回到 `pages/dnd/index.jsx` 文件中来：

```js
const dropChange = (res: any[]) => {
  const valList = (res || []).map((item) => item?.value);
  const filterList = dndList.filter((item) => !valList.includes(item.value));
  setList(filterList);
};

return (
  <DndProvider backend={HTML5Backend}>
    {/* 省略部分重复代码 */}
    <div style={{ marginTop: "10px" }}>请放置：</div>
    <CustDrop onChange={dropChange} />
  </DndProvider>
);
```

每次放置区的数据改变时，都将当前放置区的数据抛出去进行过滤。

[!img](img/img4.gif)

至此，最基本的拖拽功能小伙伴们已经顺利掌握了。

## 调整顺序能力

放置区的内容因为是用户自行拖拽的，所以存在用户自身操作问题导致的顺序错误，如果无法通过拖拽实现顺序的调整，则需要用户全部删除错误的数据重新拖拽，操作过于繁琐，因此更体现了放置区调整顺序能力的重要性。

要实现调整顺序的能力，需要给放置区的每一项都增加 `react-dnd`。

所以我们需要小小调整下放置区的代码。

`/components/CustDrop/index.tsx`

```js
import DropItem from "@/components/DropItem";

const delItem = (ind: number) => {
  const newValue = [...value];
  newValue.splice(ind, 1);
  setValue(newValue);
  onChange(newValue);
};

const showValue = () => {
  return value.map((item, index: number) => {
    return (
      <DropItem key={item?.value} data={item} moveRow={moveRow} index={index} delItem={delItem} />
    );
  });
};
```

`/components/DropItem/index.tsx`

给每个项都设置 `drop(drag(ref));`

```js
import { useDrop, useDrag } from "react-dnd";

const DropItem: FC<DropItemProps> = ({ data, index, moveRow, delItem }) => {
  const subFormItemRef = useRef(null);

  const [{ isDragging }, drag] = useDrag({
    type: "SubFormItem",
    item: {
      index,
      type: "SubFormItem",
    },
    collect: (monitor) => ({
      isDragging: monitor.isDragging(),
    }),
  });

  const [, drop] = useDrop({
    accept: "SubFormItem",
    drop: (item: any) => {
      moveRow(item.index, index);
    },
  });

  drop(drag(subFormItemRef));

  return (
    <div style={{ display: "flex" }}>
      <div ref={subFormItemRef} style={{ opacity: isDragging ? 0.5 : 1, cursor: "move" }}>
        {data?.label}
      </div>
      <span style={{ paddingLeft: "20px" }} onClick={() => delItem(index)}>
        删除
      </span>
    </div>
  );
};
```

至此，教程接近尾声。小伙伴快点操练起来吧。

[demo 地址](https://github.com/hang1017/react-dnd-demo)

demo 启动流程：

```bash
pnpm i

pnpm start

http://localhost:8000/#/dnd
```
