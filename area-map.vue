<template>
  <div id="area-map" class="area-map"></div>
</template>
<script lang="ts">
import { EChartOption } from "echarts";
import light from "@/assets/mock/light-china.json";
import { Component, Vue } from "vue-property-decorator";

@Component
export default class AreaMap extends Vue {
  private jsonList: Array<any> = [];
  public $echarts: any;
  private chart: any;
  private areaName = "china";
  chartOption: EChartOption = {};
  private mapConfig: any = {
    backcolor: {
      start: "#1889e1",
      end: "#025ec9",
      direction: "",
    },
    shadow: {
      color: "#014a8c",
      offsetX: -15,
      offsetY: 6,
      blurRadius: 0,
    },
    border: {
      outer: {
        width: 1,
        color: "#39ca95",
      },
      inner: {
        width: 1,
        color: "#cfecff",
      },
    },
    effectScatter: {
      isShow: true,
      symbolSize: 8,
      color: "#059de3",
      label: {
        isShow: true,
        // position: "right",
        color: "#ebc309",
        fontSize: 12,
      },
      shadow: {
        color: "#ffffff",
        blurRadius: 8,
      },
    },
  };

  private lightData = this.getLightData();

  mounted(): void {
    this.initCharts();
  }
  initCharts(): void {
    const ele = document.getElementById("area-map");
    if (this.chart) {
      this.chart.clear();
    } else {
      this.chart = this.$echarts.init(ele);
    }
    this.requestJsonList().then((jsonList) => {
      this.jsonList = jsonList;
      console.log("jsonList", jsonList);
      this.showCharts(jsonList);
      this.chart.setOption(this.chartOption);
    });
  }
  showCharts(jsonList: any[]): void {
    this.chart.clear();
    if (jsonList.length > 1) {
      this.$echarts.registerMap(this.areaName, jsonList[1]);
      this.$echarts.registerMap(`${this.areaName}border`, jsonList[0]);
    } else {
      this.$echarts.registerMap(`${this.areaName}border`, jsonList[0]);
    }

    this.chartOption = {
      tooltip: {
        trigger: "item",
        formatter: "{b}",
      },
      geo: this.getGeoConfigList(jsonList),
      series: [this.getSeriesData(jsonList)],
    };
  }
  getSeriesData(jsonList: string | any[]): Record<string, any> {
    return this.mapConfig.effectScatter.isShow
      ? {
          name: "涟漪散点",
          type: "effectScatter",
          coordinateSystem: "geo",
          symbolSize: this.mapConfig.effectScatter.symbolSize,
          data: this.convertData(jsonList[jsonList.length - 1]),
          rippleEffect: {
            color: this.getEffectScatterShadowColor(),
          },
          markPoint: {
            size: this.mapConfig.effectScatter.size,
          },
          label: {
            normal: {
              show: this.mapConfig.effectScatter.label.isShow,
              formatter: "{b}",
              position: this.mapConfig.effectScatter.label.position,
              color: this.mapConfig.effectScatter.label.color,
            },
          },
          itemStyle: {
            color: this.mapConfig.effectScatter.color,
            shadowBlur: this.mapConfig.effectScatter.blurRadius,
            shadowColor: this.mapConfig.effectScatter.shadow.color,
          },
        }
      : {};
  }
  getEffectScatterShadowColor(): string {
    return this.mapConfig.effectScatter.shadow.color;
  }
  getGeoConfigList(jsonList: any[]): object {
    return jsonList.map((item, index) => {
      if (index === 0) {
        return this.getGeoBorderConfig();
      }
      return this.getGeoInnerConfig();
    });
  }
  // tslint:disable-next-line:ban-types
  getGeoBorderConfig(): Record<string, any> {
    return {
      map: `${this.areaName}border`,
      name: `${this.areaName}border`,
      type: "map",
      mapType: `${this.areaName}border`, // 自定义扩展图表类型

      cursor: "default",
      markPoint: {
        symbol: "circle",
      },
      label: {
        normal: {
          show: false,
        },
        emphasis: {
          show: false,
        },
      },
      itemStyle: {
        normal: {
          label: {
            show: false,
            position: "left",
            fontSize: 14,
            color: "gold",
          },
          areaColor: {
            type: "linear",
            x: 0,
            y: 0,
            x2: 1,
            y2: 0,
            colorStops: [
              {
                offset: 0,
                color: this.mapConfig.backcolor.start, // 100% 处的颜色
              },
              {
                offset: 1,
                color: this.mapConfig.backcolor.end, // 0% 处的颜色
              },
            ],
            global: false, // 缺省为 false
            globalCoord: false, // 缺省为 false
          },
          borderColor: this.mapConfig.border.outer.color,
          shadowBlur: this.mapConfig.shadow.blurRadius,
          shadowColor: this.mapConfig.shadow.color,
          shadowOffsetX: this.mapConfig.shadow.offsetX,
          shadowOffsetY: this.mapConfig.shadow.offsetY,
        },

        emphasis: {
          areaColor: "rgb(44,80,208)",
        },
        silent: true,
      },
      // silent: true
    };
  }
  getGeoInnerConfig(): any {
    return {
      map: this.areaName,
      name: this.areaName,
      type: "map",
      zoom:2,
      large: true,
      mapType: this.areaName, // 自定义扩展图表类型
      zIndex: 1,
      cursor: "default",
      markPoint: {
        symbol: "circle",
      },
      label: {
        normal: {
          show: false,
        },
        emphasis: {
          show: false,
        },
      },
      itemStyle: {
        normal: {
          label: {
            show: true,
          },
          areaColor: "rgba(0,0,0,0)",
          borderColor: this.mapConfig.border.inner.color,
        },
        emphasis: {
          areaColor: "rgb(44,80,208)",
        },
        silent: true,
      },
      // silent: true
    };
  }

  requestJsonList() {
    return Promise.all([
      import("@/assets/map/100000.json"),
      import("@/assets/map/100000_full.json"),
    ]);
  }
  getLightData() {
    return light.map((serieData: Array<number>) => {
      let px: number = serieData[0] / 1000;
      let py = serieData[1] / 1000;
      const res = [[px, py]];

      for (let i = 2; i < serieData.length; i += 2) {
        const dx: number = serieData[i] / 1000;
        const dy: number = serieData[i + 1] / 1000;
        const x: number = px + dx;
        const y: number = py + dy;
        res.push([parseFloat(x.toFixed(2)), parseFloat(y.toFixed(2)), 1]);
        px = x;
        py = y;
      }
      return res;
    });
  }
  getOptions() {
    const option = {
      tooltip: {
        trigger: "item",
        formatter: "{b}",
      },
      grid: {
        left:0 ,
        right: 0,
        top: 0,
        bottom: 0,
      },
      geo: [
        {
          map: `${this.areaName}full`,
          name: `${this.areaName}full`,
          type: "map",
          mapType: `${this.areaName}full`, // 自定义扩展图表类型
          zoom: 1.25,
          cursor: "default",
          markPoint: {
            symbol: "circle",
          },
          label: {
            normal: {
              show: false,
            },
            emphasis: {
              show: false,
            },
          },
          itemStyle: {
            normal: {
              label: {
                show: false,
                position: "right",
                fontSize: 14,
                color: "gold",
              },
              areaColor: "#0d47a1",
              borderColor: "rgba(255, 255, 255,1)",
              borderWidth: 2,
            },
            emphasis: {
              areaColor: "rgb(44,80,208)",
            },
            silent: true,
          },
        },
      ],
      series: [
        {
          name: `${this.areaName}full`,
          type: "scatter",
          radius: ['52%', '70%'],
          coordinateSystem: "geo",
          symbolSize: 2,
          large: true,
          zoom:2.2,
          // layoutCenter: ['30%', '30%'],
          // layoutSize: 100,
          itemStyle: {
            normal: {
              shadowBlur: 2,
              shadowColor: "#66bb6a78",
              color: "#0BA6FF",
            },
          },
          data: this.lightData[1],
        },
      ],
    };
    console.log("option", option);
    return option;
  }
  convertData(fullJson: any) {
    const res = [];
    for (const index in fullJson.features) {
      res.push({
        name: fullJson.features[index].properties.name,
        value: fullJson.features[index].properties.center,
      });
    }
    return res;
  }
}
</script>
<style lang="less" scoped>
.area-map {
  width: 100%;
  height: 100%;
  // position: absolute;
  // margin: 0 auto;
}
</style>