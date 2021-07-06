# diff算法
>1. 只对同级元素进行Diff。如果一个DOM节点在前后两次更新中跨越了层级，那么React不会尝试 复用他。
>2. 两个不同类型的元素会产生出不同的树。如果元素由div变为p，React会销毁div及其子孙节点，并新建p及其子孙节点。
>3. 开发者可以通过 key prop来暗示哪些子元素在不同的渲染下能保持稳定

##### 判断dom节点是否可以复用
> 如果key相同且type相同，则可以复用
# 单节点diff

        function reconcileSingleElement(
        returnFiber: Fiber, // 父fiber节点
        currentFirstChild: Fiber | null, // 旧的fiber节点
        element: ReactElement, // 即将更新的虚拟dom节点
        expirationTime: ExpirationTime,
        ): Fiber {
        const key = element.key
        let child = currentFirstChild
        while (child !== null) {
            // 我们无法保证新的children是单个节点的时候老的children也是单个的，所以要用遍历
            // TODO: If key === null and child.key === null, then this only applies to
            // the first item in the list.
            // 注意key为null我们也认为是相等，因为单个节点没有key也是正常的
            if (child.key === key) {
            if (
                child.tag === Fragment
                ? element.type === REACT_FRAGMENT_TYPE
                : child.elementType === element.type
            ) {
                // type相等的情况下，节点可以复用，然后删除剩下的兄弟节点
                deleteRemainingChildren(returnFiber, child.sibling)
                // 复用child节点
                const existing = useFiber(
                child,
                element.type === REACT_FRAGMENT_TYPE
                    ? element.props.children
                    : element.props,
                expirationTime,
                )
                // coerceRef的作用是把规范化ref
                existing.ref = coerceRef(returnFiber, child, element)
                existing.return = returnFiber
                if (__DEV__) {
                existing._debugSource = element._source
                existing._debugOwner = element._owner
                }
                // 退出函数
                return existing
            } else {
                // key相等type不相等，删除接下来的节点，退出循环
                deleteRemainingChildren(returnFiber, child)
                break
            }
            } else {
                // 如果key不相等，则删除该节点
            deleteChild(returnFiber, child)
            }
            child = child.sibling
        }
            // 没有可复用的节点，上面操作之后，创建新的节点
        if (element.type === REACT_FRAGMENT_TYPE) {
            const created = createFiberFromFragment(
            element.props.children,
            returnFiber.mode,
            expirationTime,
            element.key,
            )
            created.return = returnFiber
            return created
        } else {
            const created = createFiberFromElement(
            element,
            returnFiber.mode,
            expirationTime,
            )
            created.ref = coerceRef(returnFiber, currentFirstChild, element)
            created.return = returnFiber
            return created
        }
        }