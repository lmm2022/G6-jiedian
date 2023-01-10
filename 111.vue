<template>
  <div>
    <div id="flowchart"></div>
  </div>
</template>

<script>
  import G6 from "@antv/g6";

  export default {
    data() {
      return {
        // G6画布节点
        canvasData: {
          nodes: [
            {
              id: "999",
              x: 50,
              y: 200,
            },
          ],
          edges: [],
        },
        currentNode: {}, //当前node
      };
    },
    methods:{
        
    // 初始化画布及其节点事件
    initG6() {
      var _that = this
      if (this.canvasData.nodes.length === 0) {
        this.node = [
          {
            id: '999',
            x: 50,
            y: 200
          }
        ]
      }
      // 自定义节点（带删除按钮的长方形）
      G6.registerNode(
        'deletedRect',
        {
          afterDraw(cfg, group) {
            const x = 30
            const y = -20
            const bottomNode = group.addShape('rect', {
              attrs: {
                stroke: 'white',
                fill: 'white',
                x: x - 1,
                y: y - 4,
                width: 20,
                height: 20,
                zIndex: 0,
                capture: true,
                radius: 10,
                lineWidth: 1
              },
              name: 'deletedNode'
            })
            const deleteNode = group.addShape('path', {
              attrs: {
                stroke: '#000',
                zIndex: 2,
                path: [
                  ['M', 0 + x, 0 + y],
                  ['L', 10 + x, 10 + y],
                  ['M', 0 + x, 10 + y],
                  ['L', 10 + x, 0 + y]
                ],
                capture: true,
                lineWidth: 2
              },
              name: 'deletedNode'
            })
            return deleteNode, bottomNode
          },
          // 响应状态变化（修复单击后样式重叠的官方bug 勿动）
          setState(name, value, item) {
            const group = item.getContainer()
            const shape = group.get('children')[0] // 顺序根据 draw 时确定
          }
        },
        'rect'
      )
      // 自定义节点（加号）
      G6.registerNode('plusSign', {
        draw(cfg, group) {
          const bottomClickNode = group.addShape('rect', {
            attrs: {
              fill: 'white',
              y: -15,
              width: 30,
              height: 30,
              zIndex: 1,
              capture: true,
              lineWidth: 1
            },
            name: 'addNode'
          })
          const plusSignNode = group.addShape('path', {
            attrs: {
              stroke: '#40a9ff',
              zIndex: 2,
              path: [
                ['M', 15, 15],
                ['L', 15, -15],
                ['M', 0, 0],
                ['L', 30, 0]
              ],
              capture: true,
              lineWidth: 3
            },
            name: 'addNode'
          })
          return bottomClickNode, plusSignNode
        }
      })

      // 网格插件
      const grid = new G6.Grid()
      // 创建实例
      this.graph = new G6.Graph({
        container: 'flowchart',
        width: 1000,
        height: 400,
        scroller: {
          enabled: true,
          direction: 'x'
        },
        style: {
          overflow: 'auto'
        },
        modes: {
          default: [
            {
              type: 'drag-canvas', // 拖拽画布
              direction: 'x',
              scalableRange: -1
            },
            {
              type: 'scroll-canvas', // 拖拽画布
              direction: 'x',
              scalableRange: -1
            },
            // 'zoom-canvas', // 缩放画布
            'click-select' // 点击选中节点，再次点击节点或点击 Canvas 取消选中状态
            // 'activate-relations' // 当鼠标移到某节点时，突出显示该节点以及与其直接关联的节点和连线
          ]
        },
        // 节点
        defaultNode: {
          type: 'deletedRect',
          size: [100, 50],
          color: '#000',
          style: {
            fill: '#fff',
            lineWidth: 1,
            radius: 10
          },
          // 节点文字
          labelCfg: {
            style: {
              fill: '#40a9ff',
              fontSize: 16
            }
          }
        },
        // 连线
        defaultEdge: {
          style: {
            stroke: '#000',
            lineWidth: 1,
            startArrow: false,
            endArrow: true
            // startArrow: true
          }
        },
        // 插件
        plugins: [grid]
      })

      // 渲染某一指定节点
      this.graph.node((node) => {
        if (node.id === '999') {
          return {
            type: 'plusSign'
          }
        } else {
          return false
        }
      })

      // 单击加号节点新增节点
      this.graph.on('addNode:click', (ev) => {
        this.currentNode = ev.item
        // 画布有数据
        // _that.graph.focusItem(this.currentNode)
        //选择监视弹窗
        this.displayTree = true
        // 重做增加节点逻辑
        // 画布存在除加号自定义节点的其他节点
        if (_that.canvasData.nodes.length > 1) {
          //新增节点
          //修改节点数据
          let newNode = JSON.parse(
            JSON.stringify(
              _that.canvasData.nodes[_that.canvasData.nodes.length - 1]
            )
          )
          newNode.id = (Number(newNode.id) + 1).toString()
          newNode.label = '请选择节点'
          newNode.x = newNode.x + 200
          _that.canvasData.nodes[0].x = _that.canvasData.nodes[0].x + 200
          _that.canvasData.nodes.push(newNode)
        }
        //画布存在连线
        let newEdgeList = []
        // 遍历出已存在节点的id
        if (_that.canvasData.edges.length > 0) {
          for (let key in _that.canvasData.nodes) {
            if (_that.canvasData.nodes[key].id !== '999') {
              newEdgeList.push(_that.canvasData.nodes[key].id)
            }
          }
          // id排序(10以上顺序错乱 注释掉)
          // newEdgeList.sort()
          // 取上个连接线做修改处理
          let oldEdge = JSON.parse(
            JSON.stringify(
              _that.canvasData.edges[_that.canvasData.edges.length - 1]
            )
          )
          // 连接线清空重新赋值
          _that.canvasData.edges.length = 0
          // 根据id重新布置连接线
          for (let index = 0; index < newEdgeList.length - 1; index++) {
            let newEdge = JSON.parse(JSON.stringify(oldEdge))
            newEdge.id = 'edge-' + Math.random()
            newEdge.source = newEdgeList[index]
            newEdge.target = newEdgeList[(Number(index) + 1).toString()]
            _that.canvasData.edges.push(newEdge)
          }
        }
        // 画布不存在连线
        if (
          _that.canvasData.edges.length === 0 &&
          _that.canvasData.nodes.length > 1
        ) {
          let addFirstEdgeData = {
            source: _that.canvasData.nodes[1].id,
            target: (Number(_that.canvasData.nodes[1].id) + 1).toString()
          }
          _that.canvasData.edges.push(addFirstEdgeData)
        }
        // 画布不存在除加号自定义节点的其他节点
        if (_that.canvasData.nodes.length === 1) {
          let addFirstNodeData = { id: '1', label: '请选择节点', x: 50, y: 200 }
          _that.canvasData.nodes[0].x = _that.canvasData.nodes[0].x + 100
          _that.canvasData.nodes.push(addFirstNodeData)
        }
        // 读取数据
        _that.graph.data(_that.canvasData)
        // 渲染图
        _that.graph.refresh()
      })

      //单击删除按钮
      this.graph.on('deletedNode:click', (e) => {
        if (_that.canvasData.nodes) {
          // 删除节点
          for (let key in _that.canvasData.nodes) {
            if (_that.canvasData.nodes[key].id == e.item._cfg.id) {
              _that.canvasData.nodes.splice(Number(key), 1)
              _that.value.params.patrolTarget.places.splice(Number(key) - 1, 1)
            }
          }
          //判断是否为最后的节点
          if (_that.canvasData.nodes.length === 1) {
            _that.canvasData.nodes[0].x = _that.canvasData.nodes[0].x - 100
          }
          //删除连接线
          for (let index in _that.canvasData.edges) {
            // 中心节点
            if (
              _that.canvasData.edges[Number(index)].target == e.item._cfg.id &&
              _that.canvasData.edges[Number(index) + 1] &&
              _that.canvasData.edges[Number(index) + 1].source == e.item._cfg.id
            ) {
              _that.canvasData.edges[Number(index)].target =
                _that.canvasData.edges[Number(index) + 1].target
              _that.canvasData.edges.splice(Number(index) + 1, 1)
              // 计算该节点后面节点的x横坐标
              for (let keys in _that.canvasData.nodes) {
                if (Number(index) + 1 < Number(keys)) {
                  _that.canvasData.nodes[keys].x =
                    _that.canvasData.nodes[keys].x - 200
                }
              }
              _that.canvasData.nodes[0].x = _that.canvasData.nodes[0].x - 200
            }
            //判断是否为首节点
            else if (
              Number(_that.canvasData.edges[index].source) == e.item._cfg.id &&
              !_that.canvasData.edges[Number(index) - 1]
            ) {
              _that.canvasData.edges.splice(Number(index), 1)
              // 计算该节点后面节点的x横坐标
              for (let keys in _that.canvasData.nodes) {
                _that.canvasData.nodes[keys].x =
                  _that.canvasData.nodes[keys].x - 200
              }
            }
            //判断是否为末节点
            else if (
              Number(_that.canvasData.edges[index].target) == e.item._cfg.id &&
              !_that.canvasData.edges[Number(index) + 1]
            ) {
              _that.canvasData.edges.splice(Number(index), 1)
              _that.canvasData.nodes[0].x = _that.canvasData.nodes[0].x - 200
            }
          }
          //读取数据
          _that.graph.data(_that.canvasData)
          //渲染图
          _that.graph.render()
        }
      })
      // 单击画布
      this.graph.on('canvas:click', (ev) => {
        this.displayTree = false
      })
      // 单击节点
      this.graph.on('node:click', (ev) => {
        this.currentNode = ev.item
        const node = ev.item
        this.graph.focusItem(this.currentNode, true, {
          easing: 'easeCubic',
          duration: 500
        })
        this.graph.setItemState(
          node,
          'deletedNode',
          !node.hasState('deletedNode')
        )
        // 切换选中
        //选择监视弹窗
        this.displayTree = true
      })
      // 读取数据
      this.graph.data(this.canvasData)
      // 渲染图
      this.graph.render()
    },
    }
  };
</script>
