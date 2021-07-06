# 多节点diff

第一轮循环，先去一一匹配key相同并且type相同的节点

第二轮循环，比对剩余的节点  
lastplacedIndex 存储的是 “当前最后一个可复用的节点的位置”  
如果当前的oldFiber节点的index大于lastplacedIndex，则不需要移动，并且lastplacedIndex=oldFiber.index  
如果当前的oldFiber节点的index小于lastplacedIndex，则表示需要移动，标记移动。

##### 全局变量 shouldTrackSideEffects

* reconcileChildFibers的时候是true （非初始化）
* mountChildFibers的时候是false （初始化）

        function reconcileChildrenArray(
            returnFiber: Fiber,
            currentFirstChild: Fiber | null,
            newChildren: Array<*>,
            expirationTime: ExpirationTime,
        ): Fiber | null {
        if (__DEV__) {
            // First, validate keys.
            let knownKeys = null
            for (let i = 0; i < newChildren.length; i++) {
            const child = newChildren[i]
            knownKeys = warnOnInvalidKey(child, knownKeys)
            }
        }
            // 更新后的fiber节点链表起始位置
        let resultingFirstChild: Fiber | null = null
        // 上一个被添加到更新后的fiber链表中的fiber节点，即新fiber链表的尾节点
        let previousNewFiber: Fiber | null = null
        // 旧fiber节点
        let oldFiber = currentFirstChild
        let lastPlacedIndex = 0
        let newIdx = 0
        let nextOldFiber = null
            // 开始第一轮循环
        for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
            if (oldFiber.index > newIdx) {
                    // 如果旧的fiber节点的index大于当前循环到的
                nextOldFiber = oldFiber
                oldFiber = null
            } else {
            nextOldFiber = oldFiber.sibling
            }
                // 对比新旧children相同index的对象的key是否相等，如果是，返回该对象，如果不是，返回null
            const newFiber = updateSlot(
                returnFiber,
                oldFiber,
                newChildren[newIdx],
                expirationTime,
            )
            // 如果key不相等，则发生了位置变化，跳出循环
            if (newFiber === null) {
            if (oldFiber === null) {
                oldFiber = nextOldFiber
            }
            break
            }
            // key相等的情况下，如果不是初始化的时候，新的节点是新建的，那么删除旧的节点
            if (shouldTrackSideEffects) {
            if (oldFiber && newFiber.alternate === null) {
                deleteChild(returnFiber, oldFiber)
            }
            }
            // 根据lastPlacedIndex判断newFiber是移动，插入还是保持原位
            lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx)
            if (previousNewFiber === null) {
            resultingFirstChild = newFiber
            } else {
            previousNewFiber.sibling = newFiber
            }
            previousNewFiber = newFiber
            oldFiber = nextOldFiber
        } // 第一轮循环结束
            // 如果新的节点循环完毕，则表示删除了一些节点
        if (newIdx === newChildren.length) {
            deleteRemainingChildren(returnFiber, oldFiber)
            return resultingFirstChild
        }
            // 如果旧的节点先循环完，那么表示新增了一些节点
        if (oldFiber === null) {
            for (; newIdx < newChildren.length; newIdx++) {
            const newFiber = createChild(
                returnFiber,
                newChildren[newIdx],
                expirationTime,
            )
            if (!newFiber) {
                continue
            }
            lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx)
            if (previousNewFiber === null) {
                resultingFirstChild = newFiber
            } else {
                previousNewFiber.sibling = newFiber
            }
            previousNewFiber = newFiber
            }
            return resultingFirstChild
        }

        // Add all children to a key map for quick lookups.
        // 两个都没循环完，开始处理移动的节点
        // 首先把key的map存储下来
        const existingChildren = mapRemainingChildren(returnFiber, oldFiber)

        // Keep scanning and use the map to restore deleted items as moves.
        for (; newIdx < newChildren.length; newIdx++) {
            const newFiber = updateFromMap(
            existingChildren,
            returnFiber,
            newIdx,
            newChildren[newIdx],
            expirationTime,
            )
            if (newFiber) {
            if (shouldTrackSideEffects) {
                if (newFiber.alternate !== null) {
                existingChildren.delete(newFiber.key === null ? newIdx : newFiber.key)
                }
            }
            lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx)
            if (previousNewFiber === null) {
                resultingFirstChild = newFiber
            } else {
                previousNewFiber.sibling = newFiber
            }
            previousNewFiber = newFiber
            }
        }

        if (shouldTrackSideEffects) {
            existingChildren.forEach(child => deleteChild(returnFiber, child))
        }

        return resultingFirstChild
        }