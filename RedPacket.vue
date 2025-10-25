<template>
  <div class="red-packet-container">
    <!-- 红包元素 -->
    <div 
      class="red-packet"
      :class="{ 'opened': isOpened }"
      @click="openPacket"
      v-if="!isOpened && !showResult"
    >
      <!-- 红包封口 -->
      <div class="packet-seal"></div>
      <!-- 金色圆形开启按钮 -->
      <div class="open-button">開</div>
      <!-- 红包文字 -->
      <div class="packet-text">红包</div>
    </div>

    <!-- 金币飞散效果（动态生成） -->
    <div 
      class="coin" 
      v-for="(coin, index) in coins" 
      :key="index"
      :style="{ 
        left: coin.left, 
        top: coin.top,
        animationDelay: coin.delay,
        animationDuration: coin.duration,
        '--endX': `${coin.endX}%`,
        '--endY': `${coin.endY}%`
      }"
      v-if="isOpened && !showResult"
    >
      ¥
    </div>

    <!-- 结果弹窗 -->
    <div class="result-modal" v-if="showResult">
      <div class="modal-content">
        <div class="confetti"></div> <!-- 彩屑背景 -->
        <div class="modal-header">恭喜发财</div>
        <div class="modal-money">¥{{ money }}</div>
        <div class="modal-desc">抢到红包啦！</div>
        <button class="modal-btn" @click="resetGame">再来一个</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isOpened: false,      // 红包是否已打开
      showResult: false,    // 是否显示结果弹窗
      money: 0,             // 抢到的金额
      coins: []             // 金币动画数据
    };
  },
  methods: {
    // 打开红包
    openPacket() {
      if (this.isOpened) return;
      
      // 标记红包为打开状态（触发动画）
      this.isOpened = true;
      
      // 生成金币飞散数据（15个金币）
      this.generateCoins(15);
      
      // 生成随机金额（0.01-200元，保留两位小数）
      this.money = (Math.random() * (200 - 0.01) + 0.01).toFixed(2);
      
      // 动画结束后显示结果（1.5秒后）
      setTimeout(() => {
        this.showResult = true;
      }, 1500);
    },
    
    // 生成金币飞散动画参数
    generateCoins(count) {
      this.coins = [];
      const centerX = 50; // 红包中心X轴百分比
      const centerY = 50; // 红包中心Y轴百分比
      
      for (let i = 0; i < count; i++) {
        // 随机方向和距离
        const angle = Math.random() * Math.PI * 2; // 0-360度随机角度
        const distance = 30 + Math.random() * 50;  // 飞散距离（30-80%视窗）
        // 计算终点位置
        const endX = centerX + Math.cos(angle) * distance;
        const endY = centerY + Math.sin(angle) * distance;
        
        this.coins.push({
          left: `${centerX}%`,   // 起始X位置（红包中心）
          top: `${centerY}%`,    // 起始Y位置（红包中心）
          endX: endX,            // 终点X位置
          endY: endY,            // 终点Y位置
          delay: `${Math.random() * 0.5}s`, // 随机延迟（0-0.5秒）
          duration: `${0.8 + Math.random() * 0.7}s` // 随机动画时长（0.8-1.5秒）
        });
      }
    },
    
    // 重置游戏（可再次抢红包）
    resetGame() {
      this.isOpened = false;
      this.showResult = false;
      this.coins = [];
    }
  }
};
</script>

<style scoped>
.red-packet-container {
  width: 100vw;
  height: 100vh;
  background: #f8f0e3;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  position: relative;
}

/* 红包样式 */
.red-packet {
  width: 180px;
  height: 220px;
  background: linear-gradient(135deg, #e63946 0%, #d62828 100%);
  border-radius: 12px;
  position: relative;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: all 0.5s ease;
  transform-origin: center top; /* 以顶部为旋转原点 */
}

/* 红包打开动画 */
.red-packet.opened {
  transform: rotateX(180deg) translateY(20px);
  opacity: 0;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
}

/* 红包封口 */
.packet-seal {
  position: absolute;
  width: 100%;
  height: 30px;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  background: linear-gradient(90deg, #c1121f 0%, #7f1d1d 100%);
  border-top: 2px solid #ffb3c1;
  border-bottom: 2px solid #ffb3c1;
}

/* 金色圆形开启按钮 */
.open-button {
  position: absolute;
  width: 50px;
  height: 50px;
  top: calc(50% + 35px); /* 封口下方居中 */
  left: 50%;
  transform: translateX(-50%);
  background: linear-gradient(135deg, #ffd700 0%, #ffb347 100%);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 19px;
  font-weight: bold;
  color: #000;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  z-index: 5;
}
.open-button:hover {
  transform: translateX(-50%) scale(1.25); /* 悬停时放大1.25倍 */
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3), 
              inset 0 2px 2px rgba(255, 255, 255, 0.5);
}
/* 红包文字 */
.packet-text {
  position: absolute;
  top: 19%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #fff;
  font-size: 28px;
  font-weight: bold;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

/* 金币样式 */
.coin {
  position: absolute;
  width: 30px;
  height: 30px;
  background: linear-gradient(135deg, #ffd700 0%, #ffb347 100%);
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  z-index: 10;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #8b5a00;
  font-weight: bold;
  font-size: 16px;
  animation: coinFly 1s forwards;
}

/* 金币飞移动画 */
@keyframes coinFly {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 1;
  }
  80% {
    opacity: 0.8;
  }
  100% {
    transform: translate(calc(var(--endX) - 50%), calc(var(--endY) - 50%)) scale(0.3);
    opacity: 0;
  }
}

/* 结果弹窗背景 */
.result-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  animation: fadeIn 0.3s ease;
}

/* 弹窗内容 */
.modal-content {
  width: 80%;
  max-width: 300px;
  background: #fff;
  border-radius: 16px;
  padding: 30px 20px;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  animation: popUp 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  position: relative; /* 为内部绝对定位元素提供参考 */
}
.confetti {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* 不影响点击交互 */
  background-image: 
    radial-gradient(#ffd700 2px, transparent 2px),
    radial-gradient(#e63946 2px, transparent 2px),
    radial-gradient(#4cc9f0 2px, transparent 2px); /* 三色彩屑 */
  background-size: 40px 40px;
  background-position: 0 0, 20px 20px, 30px 10px;
  opacity: 0.2; /* 半透明效果 */
  z-index: 0; /* 放在内容下层 */
}

/* 弹窗标题 */
.modal-header {
  font-size: 24px;
  color: #e63946;
  margin-bottom: 15px;
  font-weight: bold;
}

/* 金额显示 */
.modal-money {
  font-size: 40px;
  font-weight: bold;
  color: #d62828;
  margin: 20px 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  position: relative; /* 为伪元素定位做准备 */
  display: inline-block;
}
.modal-money::after {
  content: '';
  position: absolute;
  bottom: 5px;
  left: 0;
  width: 100%;
  height: 3px;
  background: linear-gradient(90deg, transparent, #ffd700, transparent); /* 渐变下划线 */
  opacity: 0.7;
}

/* 确保弹窗内容在彩屑之上 */
.modal-header, .modal-money, .modal-desc, .modal-btn {
  position: relative;
  z-index: 1; /* 确保内容在彩屑之上 */
}

/* 弹窗描述 */
.modal-desc {
  color: #666;
  margin-bottom: 30px;
  font-size: 16px;
}

/* 按钮样式 */
.modal-btn {
  background: #e63946;
  color: white;
  border: none;
  padding: 12px 30px;
  border-radius: 30px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.modal-btn:hover {
  background: #d62828;
  transform: translateY(-2px);
}



/* 弹窗动画 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes popUp {
  from { transform: scale(0.7); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
</style>