<template>
  <div class="GradientMaster">
    <section class="left">
      <div class="color-box">
        <div ref="lgBox" class="linear-gradient-box" :style="style" @click="handleBoxClick">
          <div class="diagonal-line"></div>
          <div
            v-for="item in sideCorners"
            :key="item"
            :class="['to-side-corner', item.split(' ').join('-'), { active: sideOrCorner === 'to ' + item }]"
             :data-to="item">
            </div>
          <div class="gradient-line" :style="gradientLineStyle">
            <div class="grip" id="grip" @mousedown="handleMousedown"></div>
          </div>
        </div>
      </div>
      <div class="input-box">
        <div class="input-group">
          <label>{{gradientValue}}</label>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
  function calculateAngle(pointA, pointB) {
    // 计算水平和垂直距离
    var dx = pointB.x - pointA.x;
    var dy = pointB.y - pointA.y;

    // 计算角度（弧度值）
    var angleRad = Math.atan2(dy, dx);

    // 将弧度转换为角度（0-360度）
    var angleDeg = Math.round((angleRad * 180 / Math.PI + 360) % 360);

    return angleDeg;
  }
  export default {
    data: () => ({
      angle: 0,
      sideOrCorner: 'to right',
      // #e66465, #9198e5
      colorStops: [
        { color: '#e66465', stop: null },
        { color: '#9198e5', stop: null }
      ],
      sideCorners: ['top', 'right', 'bottom', 'left', 'left top', 'right top', 'left bottom', 'right bottom']
    }),
    computed: {
      gradientValue() {
        const colorStops = this.colorStops.map(item => {
          return `${item.color} ${item.stop || ''}`.trim();
        }).join(', ');

        let angle;
        if (this.sideOrCorner === 'custom') {
          angle = `${this.computedAngle}deg`;
        } else {
          angle = this.sideOrCorner
        }

        return `linear-gradient(${angle}, ${colorStops})`
      },
      style() {
        return {
          color: 'red',
          backgroundImage: this.gradientValue
        }
      },
      computedAngle() {
        switch (this.sideOrCorner) {
          case 'to right': return 90;
          case 'to right bottom': 
          case 'to bottom right': 
            return 135;
          case 'to bottom': return 180;
          case 'to left bottom':
          case 'to bottom left':
            return 225;
          case 'to left': return 270;
          case 'to left top':
          case 'to top left':
            return 315;
          case 'to top': return 0;
          case 'to top right': 
          case 'to right top': 
            return 45;
          default: return this.angle;
        }
      },
      gradientLineStyle() {
        return {
          transform: `rotate(${this.computedAngle - 90}deg)`
        }
      }
    },
    created() {
      this.handleMousemove = this.handleMousemove.bind(this);
      this.handleMouseup = this.handleMouseup.bind(this);
    },
    methods: {
      handleBoxClick(ev) {
        const dataset = ev.target.dataset;
        if (dataset.to) {
          this.sideOrCorner = 'to ' + dataset.to;
          return;
        }
      },
      handleMousedown() {
        this.centerPoint = (function(myDiv) {
          var rect = myDiv.getBoundingClientRect();
          var x = rect.left + rect.width / 2;
          var y = rect.top + rect.height / 2;
          return {x, y};
        })(this.$refs.lgBox);
        document.addEventListener('mousemove', this.handleMousemove);
        document.addEventListener('mouseup', this.handleMouseup);
      },
      handleMousemove(ev) {
        const {x, y} = ev;
        const angle = calculateAngle(this.centerPoint, {x, y}) + 90;
        if (this.sideOrCorner != 'custom') {
          this.sideOrCorner = 'custom';
        }
        this.angle = angle;
        ev.preventDefault();
      },
      handleMouseup() {
        document.removeEventListener('mouseup', this.handleMouseup);
        document.removeEventListener('mousemove', this.handleMousemove);
      }
    }
  }
</script>

<style scoped>
.GradientMaster {
  display: flex;
  padding: 16px;
}
.left {
  flex: 1 1 auto;
  height: 100%;
  display: flex;
  flex-direction: column;
  margin-right: 16px;
}
.right {
  width: 400px;
}

.color-box {
  flex: 1 1 auto;
  border-radius: 12px;
  background-color: #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
}

.linear-gradient-box {
  width: 50vh;
  height: 50vh;
  background-color: white;
  position: relative;
}

.diagonal-line {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.diagonal-line::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  width: 200%;
  border-bottom: 1px dashed gray;
  transform: rotate(45deg);
  transform-origin: 0;
}

.grip {
  position: absolute;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background-color: yellowgreen;
  right: 0;
  top: 50%;
  transform: translate(400%, -50%);
}

.diagonal-line::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: 0;
  width: 200%;
  border-bottom: 1px dashed gray;
  transform: rotate(-45deg);
  transform-origin: 0;
}

.to-side-corner {
  position: absolute;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: 2px solid gray;
  cursor: pointer;
}

.to-side-corner:hover {
  border-color: green;
}
.to-side-corner.active {
  background: green;
}

.to-side-corner.left { left: 0; top: 50%; transform: translate(-150%, -50%); }
.to-side-corner.top { left: 50%; top: 0; transform: translate(-50%, -150%); }
.to-side-corner.right { right: 0; top: 50%; transform: translate(150%, -50%); }
.to-side-corner.bottom { left: 50%; bottom: 0; transform: translate(-50%, 150%); }
.to-side-corner.left-top { left: 0; top: 0; transform: translate(-100%, -100%); }
.to-side-corner.left-bottom { left: 0; bottom: 0; transform: translate(-100%, 100%); }
.to-side-corner.right-top { right: 0; top: 0; transform: translate(100%, -100%); }
.to-side-corner.right-bottom { right: 0; bottom: 0; transform: translate(100%, 100%); }

.gradient-line {
  width: 150%;
  height: 2px;
  background-color: black;
  position: absolute;
  left: calc(0px - 25%);
  top: calc(50% - 1px);
  cursor: pointer;
}
.gradient-line::after {
  content: '';
  position: absolute;
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 10px;
  border-color: transparent;
  border-left-width: 30px;
  border-left-color: black;
  right: -40px;
  top: -8px;
}

.input-box {
  margin: 16px;
}
.input-group {
  display: flex;
  align-items: center;
}

.input-group input {
  border: 0;
  flex: 1 1 auto;
}

.input-group input:focus-visible {
  outline: none;
}

.input-group, .input-group input {
  font-family: monospace;
  font-size: 20px;
}
</style>