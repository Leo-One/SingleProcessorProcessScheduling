<template>
  <div class="main">
    <div class="row strategy">
      <label>选择调度策略：</label>
      <el-select v-model="strategy" placeholder="请选择">
        <el-option
          v-for="item in strategies"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-button type="primary" round id="submit" @click="submit">确认</el-button>
    </div>

    <div class="row time">
      <el-input v-model="setTime" placeholder="请输入时间片长度"></el-input>
    </div>

    <div class="row table">
      <el-table :data="processMessage" style="width: 100%">
        <el-table-column prop="process" label="进程" width="150"></el-table-column>

        <el-table-column prop="arrive" label="到达时间" width="300">
          <template slot-scope="scope">
            <el-select v-model="scope.row.arrive" placeholder="请选择" size="mini">
              <el-option
                v-for="item in arrAndSerTime"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </template>
        </el-table-column>

        <el-table-column prop="service" label="服务时间" width="300">
          <template slot-scope="scope">
            <el-select v-model="scope.row.service" placeholder="请选择" size="mini">
              <el-option
                v-for="item in arrAndSerTime"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </template>
        </el-table-column>

        <el-table-column prop="priority" label="优先级">
          <template slot-scope="scope">
            <el-select v-model="scope.row.priority" placeholder="请选择" size="mini">
              <el-option
                v-for="item in prioritySel"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- ECharts图表测试 -->
    <div id="echarts"></div>
  </div>
</template>

<script>
export default {
  name: "Page",
  data() {
    return {
      setTime: null,
      occupy: "0",
      strategy: null,
      strategies: [
        {
          value: 1,
          label: "时间片轮转 RR"
        },
        {
          value: 2,
          label: "最短进程优先 SPN"
        },
        {
          value: 3,
          label: "最短剩余时间优先 SRT"
        },
        {
          value: 4,
          label: "最高响应比优先 HRRN"
        },
        {
          value: 5,
          label: "优先数"
        }
      ],
      processMessage: [
        {
          process: "A",
          arrive: null,
          service: null,
          priority: null
        },
        {
          process: "B",
          arrive: null,
          service: null,
          priority: null
        },
        {
          process: "C",
          arrive: null,
          service: null,
          priority: null
        },
        {
          process: "D",
          arrive: null,
          service: null,
          priority: null
        },
        {
          process: "E",
          arrive: null,
          service: null,
          priority: null
        }
      ],
      arrAndSerTime: [],
      prioritySel: [],
      processData: [],
      canvasData: [],
      list: []
    };
  },

  created: function() {
    this.generatePriority();
    this.generateArrAndSerTime();
  },
  methods: {
    /**
     * 初始化生成优先级选项的函数
     */
    generatePriority: function() {
      for(var i = 1; i <= 5; i ++) {
        var priority = {
          label: i,
          value: i
        }
        this.prioritySel.push(priority);
      }
    },
    /**
     * 初始化生成到达时间和服务时间选项的函数
     */
    generateArrAndSerTime: function() {
      for(var i = 0; i <= 10; i ++) {
        var item = {
          label: i,
          value: i
        }
        this.arrAndSerTime.push(item);
      }
    },
    submit: function() {
      console.log(this.strategy);
      switch(this.strategy) {
        case 1: this.RR(); break;
        case 2: this.SPN();break;
        case 3: this.SRT();break;
        case 4: this.HRRN();break;
        case 5: this.PSA();break;
      }
      
      /*ECharts图表*/
      var echarts = require("echarts");

      // 基于准备好的dom，初始化echarts实例
      var myChart = echarts.init(document.getElementById("echarts"));

      //var data = [];
      var dataCount = 100; // 测试数据条数
      var startValue = 0; // 2018/5/9 11:16:31
      var categories = ["A", "B", "C", "D", "E"].reverse();
      var types = [
        {
          name: "running",
          color: "#75d874"
        }
      ];

      function renderItem(params, api) {
        var categoryIndex = api.value(0);
        var start = api.coord([api.value(1), categoryIndex]);
        var end = api.coord([api.value(2), categoryIndex]);
        var height = api.size([0, 1])[1] * 0.6;

        return {
          type: "rect",
          shape: echarts.graphic.clipRectByRect(
            {
              x: start[0],
              y: start[1] - height / 2,
              width: end[0] - start[0],
              height: height
            },
            {
              x: params.coordSys.x,
              y: params.coordSys.y,
              width: params.coordSys.width,
              height: params.coordSys.height
            }
          ),
          style: api.style()
        };
      }

      // 绘制图表
      myChart.setOption({
        tooltip: {
          formatter: function(params) {
            //console.log(params);
            return params.marker + params.name + ": " + params.value[3];
          }
        },
        title: {
          text: "单处理器的进程调度",
          left: "center"
        },
        legend: {
          //图例
          data: types,
          bottom: 20,
          right: 150,
          selectedMode: false
        },
        dataZoom: [
          {
            //区域缩放控制器
            type: "slider", //有滑动块的
            filterMode: "weakFilter", //当前数据窗口外的数据，被过滤掉。即会影响其他轴的数据范围。每个数据项，只有当全部维度都在数据窗口同侧外部，整个数据项才会被过滤掉。
            showDataShadow: false,
            top: 400, //组件离容器上侧的距离
            height: 10,
            start: 0, //数据窗口范围的起始百分比
            end: 100, //数据窗口范围的结束百分比
            borderColor: "transparent", //边框颜色
            backgroundColor: "#e2e2e2", //组件的背景颜色
            handleIcon:
              "M10.7,11.9H9.3c-4.9,0.3-8.8,4.4-8.8,9.4c0,5,3.9,9.1,8.8,9.4h1.3c4.9-0.3,8.8-4.4,8.8-9.4C19.5,16.3,15.6,12.2,10.7,11.9z M13.3,24.4H6.7v-1.2h6.6z M13.3,22H6.7v-1.2h6.6z M13.3,19.6H6.7v-1.2h6.6z", // jshint ignore:line
            handleSize: 20,
            handleStyle: {
              shadowBlur: 6,
              shadowOffsetX: 1,
              shadowOffsetY: 2,
              shadowColor: "#aaa"
            },
            labelFormatter: ""
          },
          {
            type: "inside",
            filterMode: "weakFilter"
          }
        ],
        grid: {
          height: 300 //图例高度
        },
        xAxis: {
          type: "value",
          min: startValue,
          scale: true //只在数值轴中（type: 'value'）有效。是否是脱离 0 值比例。设置成 true 后坐标刻度不会强制包含零刻度。在双数值轴的散点图中比较有用。在设置 min 和 max 之后该配置项无效。
        },
        yAxis: {
          data: categories
        },
        series: [
          { name: types[0].name, type: "bar", data: [], color: "#75d874" },
          {
            type: "custom",
            renderItem: renderItem,
            itemStyle: {
              normal: {
                opacity: 0.8 //图标色块深浅
              }
            },
            encode: {
              x: [1, 2, 3],
              y: 0
            },
            data: this.canvasData
          }
        ]
      });
    },
    /**
     * 把表格数据格式化后存到this.processData
     */
    formatData: function() {
      for (var i = 0; i < this.processMessage.length; i++) {
        var process = {};
        process.name = this.processMessage[i].process;
        process.index = this.processMessage.length - i - 1;
        process.arriveTime = this.processMessage[i].arrive;
        process.serviceTime = this.processMessage[i].service;
        process.priority = this.processMessage[i].priority;
        this.processData.push(process);
      }
    },
    /**
     * 所有进程按照进程到达时间进行排序
     */
    sortArriveTime: function(list) {
      for (var i = 0; i < length; i++) {
        for (var j = 0; j < length - 1; j++) {
          if (list[j].arriveTime > list[j + 1].arriveTime) {
            var temp = list[j];
            list[j] = list[j + 1];
            list[j + 1] = temp;
          }
        }
      }
      return list;
    },
    /**
     * 时间片轮转算法
     * setTime为时间片长度
     */
    RR: function() {
      this.formatData(); //从表格中获取数据，并进行格式初始化，存在this.processData中
      var processes = this.sortArriveTime(this.processData);
      var length = processes.length;

      //把进程放进队列this.list中
      for (var i = 0; i < processes.length; i++) {
        this.list.push(processes[i]);
      }

      //开始遍历队列
      var CPUTime = 0; //当前时间
      var setTime = 4;
      while (this.list.length != 0) {
        //队列不为空时进入循环
        var canvasSubData = {
          //封装好的echarts图标数据，用于显示调度运行图像
          name: "running",
          value: [],
          itemStyle: { normal: { color: "#75d874" } }
        };
        var minArriveTime = this.list[0].arriveTime; //标记当前队列中所有进程的最小到达时间
        for (var i = 0; i < this.list.length; i++) {
          if (this.list[i].arriveTime < minArriveTime) {
            minArriveTime = this.list[i].arriveTime;
          }
        }
        var process = new Object();
        process = this.list.shift(); //弹出队列第一个进程
        if (process.arriveTime <= CPUTime && minArriveTime <= CPUTime) {
          //如果当前进程在当前时间已到达，进程执行
          if (process.serviceTime == setTime) {
            //如果当前进程剩余服务时间 == 时间片时间，该进程执行完后不进队，当前时间推进一个时间片长度
            canvasSubData.value = [
              process.index,
              CPUTime,
              CPUTime + setTime,
              setTime
            ];
            CPUTime += 4;
          } else if (process.serviceTime > setTime) {
            //如果当前进程剩余服务时间 > 时间片时间，该进程执行完后更新剩余服务时间，插到队伍最后，当前时间推进一个时间片长度
            process.serviceTime = process.serviceTime - setTime;
            canvasSubData.value = [
              process.index,
              CPUTime,
              CPUTime + setTime,
              setTime
            ];
            CPUTime += 4;
            this.list.push(process);
          } else if (process.serviceTime < setTime) {
            //如果当前进程剩余服务时间 < 时间片时间，该进程执行完后不进队，当前时间推进时间长度为该进程的剩余服务时间
            canvasSubData.value = [
              process.index,
              CPUTime,
              CPUTime + process.serviceTime,
              process.serviceTime
            ];
            CPUTime += process.serviceTime;
          }
          //console.log(canvasSubData);
          this.canvasData.push(canvasSubData);
        } else if (process.arriveTime > CPUTime && minArriveTime > CPUTime) {
          //如果当前进程在此时间仍未到达，并且队列其他进程也未到达，当前进程插到队末，时间+1
          this.list.push(process);
          CPUTime++;
        } else if (process.arriveTime > CPUTime && minArriveTime <= CPUTime) {
          //如果当前进程在此事件仍未到达，可是队列中有进程已到达，当前进程插入队末，时间不变，继续判断后续进程
          this.list.push(process);
        }
      }
    },
    /**
     * 最短进程优先
     */
    SPN: function() {
      this.formatData(); //从表格中获取数据，并进行格式初始化，存在this.processData中
      var processes = this.sortArriveTime(this.processData);
      var length = processes.length;
      var flag = true;
      var CPUTime = 0;
      var readyList = [];
      var addFlag = false;
      while (flag) {
        var canvasSubData = {
          //封装好的echarts图标数据，用于显示调度运行图像
          name: "running",
          value: [],
          itemStyle: { normal: { color: "#75d874" } }
        };

        //遍历时间队列，把当前已到达程序加入就绪队列
        var judgeArrive = true;
        while (judgeArrive && processes.length != 0) {
          var i = 0;
          addFlag = false;
          if (processes[0].arriveTime > CPUTime) {
            addFlag = true;
            judgeArrive = false;
          } else if (processes[0].arriveTime <= CPUTime) {
            var readyProcess = processes.shift();
            readyList.push(readyProcess);
          }
        }

        //就绪队列升序排序
        while (readyList.length > 0) {
          if (readyList.length > 1) {
            for (var m = 0; m < readyList.length; m++) {
              for (var n = 0; n < readyList.length - 1; n++) {
                if (readyList[n].serviceTime > readyList[n + 1].serviceTime) {
                  var temp = readyList[n];
                  readyList[n] = readyList[n + 1];
                  readyList[n + 1] = temp;
                }
              }
            }
          }
          //执行就绪队列队首进程
          canvasSubData.value = [
            readyList[0].index,
            CPUTime,
            CPUTime + readyList[0].serviceTime,
            readyList[0].serviceTime
          ];
          this.canvasData.push(canvasSubData);
          CPUTime += readyList[0].serviceTime;//更新CPU时间
          addFlag = false;
          readyList.shift();
          break;
        }
        if (addFlag == true) {
          CPUTime++;
        }
        if (processes.length == 0 && readyList.length == 0) {
          flag = false;
        }
      }
    },
    /**
     * 最高响应比优先
     */
    HRRN: function() {
      this.formatData(); //从表格中获取数据，并进行格式初始化，存在this.processData中
      var processes = this.sortArriveTime(this.processData);
      var length = processes.length;

      var flag = true;
      var CPUTime = 0;
      var readyList = [];
      var addFlag = false;
      while (flag) {
        var canvasSubData = {
          //封装好的echarts图标数据，用于显示调度运行图像
          name: "running",
          value: [],
          itemStyle: { normal: { color: "#75d874" } }
        };

        //遍历时间队列，把当前已到达程序加入就绪队列
        var judgeArrive = true;
        while (judgeArrive && processes.length != 0) {
          var i = 0;
          addFlag = false;
          if (processes[0].arriveTime > CPUTime) {
            addFlag = true;
            judgeArrive = false;
          } else if (processes[0].arriveTime <= CPUTime) {
            var readyProcess = processes.shift();
            readyList.push(readyProcess);
          }
        }

        //就绪队列升序排序
        while (readyList.length > 0) {
          //计算响应比
          for (var i = 0; i < readyList.length; i++) {
            var R =
              (CPUTime - readyList[i].arriveTime + readyList[i].serviceTime) /
              readyList[i].serviceTime;
            readyList[i].R = R;
          }
          for (var m = 0; m < readyList.length; m++) {
            for (var n = 0; n < readyList.length - 1; n++) {
              if (readyList[n].R < readyList[n + 1].R) {
                var temp = readyList[n];
                readyList[n] = readyList[n + 1];
                readyList[n + 1] = temp;
              }
            }
          }
          canvasSubData.value = [
            readyList[0].index,
            CPUTime,
            CPUTime + readyList[0].serviceTime,
            readyList[0].serviceTime
          ];
          this.canvasData.push(canvasSubData);
          CPUTime += readyList[0].serviceTime;

          addFlag = false;
          readyList.shift();
          break;
        }
        if (addFlag == true) {
          CPUTime++;
        }
        if (processes.length == 0 && readyList.length == 0) {
          flag = false;
        }
      }
    },

    SRT: function() {
      this.formatData(); //从表格中获取数据，并进行格式初始化，存在this.processData中
      var processes = this.processData;
      var length = processes.length;

      //所有进程按照进程剩余服务时间进行排序
      for (var i = 0; i < length; i++) {
        for (var j = 0; j < length - 1; j++) {
          if (processes[j].serviceTime > processes[j + 1].serviceTime) {
            var temp = processes[j];
            processes[j] = processes[j + 1];
            processes[j + 1] = temp;
          }
        }
      }
      console.log("所有进程按照进程剩余服务时间进行排序" + processes);
      //把进程放进队列this.list中
      for (var i = 0; i < processes.length; i++) {
        console.log(processes[i]);
      }

      var CPUTime = 0;
      var readyList = [];
      var flag = true;
      while (flag) {
        var canvasSubData = {
          //封装好的echarts图标数据，用于显示调度运行图像
          name: "running",
          value: [],
          itemStyle: { normal: { color: "#75d874" } }
        };

        for (var i = 0; i < length; i++) {
          if (
            processes[i].arriveTime <= CPUTime &&
            processes[i].serviceTime != 0
          ) {
            //选取当前可运行程序中的最短剩余服务时间程序
            console.log(processes[i]);
            processes[i].serviceTime -= 1;

            canvasSubData.value = [processes[i].index, CPUTime, CPUTime + 1, 1];
            this.canvasData.push(canvasSubData);
            CPUTime += 1;
            console.log("canvasSubData.value" + canvasSubData.value);
            //执行一个时间后重新排序改程序
            for (var m = 0; m < length; m++) {
              for (var n = 0; n < length - 1; n++) {
                if (processes[n].serviceTime > processes[n + 1].serviceTime) {
                  var temp = processes[n];
                  processes[n] = processes[n + 1];
                  processes[n + 1] = temp;
                }
              }
            }
            break;
          }
          if (i == length - 1) {
            CPUTime += 1;
          }
        }
        if (processes[processes.length - 1].serviceTime == 0) {
          flag = false;
        }
      }
    },
    /**
     * 优先数
     */
    PSA: function() {
      this.formatData(); //从表格中获取数据，并进行格式初始化，存在this.processData中
      var processes = this.sortArriveTime(this.processData);
      var length = processes.length;

      var flag = true;
      var CPUTime = 0;
      var readyList = [];
      var addFlag = false;
      while (flag) {
        var canvasSubData = {
          //封装好的echarts图标数据，用于显示调度运行图像
          name: "running",
          value: [],
          itemStyle: { normal: { color: "#75d874" } }
        };

        //遍历时间队列，把当前已到达程序加入就绪队列
        var judgeArrive = true;
        while (judgeArrive && processes.length != 0) {
          var i = 0;
          addFlag = false;
          if (processes[0].arriveTime > CPUTime) {
            addFlag = true;
            judgeArrive = false;
          } else if (processes[0].arriveTime <= CPUTime) {
            var readyProcess = processes.shift();
            readyList.push(readyProcess);
          }
        }

        //就绪队列优先级降序排序
        while (readyList.length > 0) {
          if (readyList.length > 1) {
            for (var m = 0; m < readyList.length; m++) {
              for (var n = 0; n < readyList.length - 1; n++) {
                if (readyList[n].priority < readyList[n + 1].priority) {
                  var temp = readyList[n];
                  readyList[n] = readyList[n + 1];
                  readyList[n + 1] = temp;
                }
              }
            }
          }
          canvasSubData.value = [
            readyList[0].index,
            CPUTime,
            CPUTime + readyList[0].serviceTime,
            readyList[0].serviceTime
          ];
          this.canvasData.push(canvasSubData);
          CPUTime += readyList[0].serviceTime;

          addFlag = false;
          readyList.shift();
          break;
        }
        if (addFlag == true) {
          CPUTime++;
        }
        if (processes.length == 0 && readyList.length == 0) {
          flag = false;
        }
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#occupySel {
  margin-left: 100px;
  font-family: "微软雅黑";
}
.time {
  width: 150px;
  padding-left: 90px;
}
.strategy {
  width: 600px;
}
.row {
  display: block;
  margin-top: 30px;
}
.table {
  margin-left: 90px;
}
#echarts {
  width: 100%;
  height: 500px;
  padding-top: 30px;
}
</style>
