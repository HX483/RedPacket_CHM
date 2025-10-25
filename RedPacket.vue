<template>
  <div class="red-packet-container">
    <!-- 我的红包记录列表 -->
    <div class="my-red-packets" v-if="showRedPacketList">
      <div class="my-red-packets-title">红包记录</div>
      <div class="my-red-packets-list">
        <div v-if="myRedPackets.length === 0" class="empty-record">暂无记录</div>
        <div 
          v-for="(packet, index) in myRedPackets" 
          :key="index" 
          class="my-red-packet-item"
        >
          <div class="packet-user">{{ packet.user }}</div>
          <div class="packet-time">{{ formatTimestamp(packet.timestamp) }}</div>
          <div class="packet-amount">¥{{ packet.money.toFixed(2) }}</div>
        </div>
      </div>
    </div>
    <!-- 设置界面 -->
    <div class="settings-modal" v-if="showSettings">
      <div class="modal-content">
        <div class="modal-header">设置红包</div>
        <div class="settings-form">
          <div class="form-group">
            <label for="totalMoney">总金额（元）</label>
            <input 
              type="number" 
              id="totalMoney" 
              v-model.number="inputMoney"
              min="0.01" 
              max="200" 
              step="0.01"
              placeholder="默认100元"
            >
            <span class="form-hint">最大200元</span>
          </div>
          <div class="form-group">
            <label for="totalUsers">总人数</label>
            <input 
              type="number" 
              id="totalUsers" 
              v-model.number="inputUsers"
              min="1" 
              max="20" 
              step="1"
              placeholder="默认5人"
            >
            <span class="form-hint">最大20人</span>
          </div>
          <button class="modal-btn" @click="saveSettings">开始抢红包</button>
        </div>
      </div>
    </div>
    
    <!-- 红包元素 -->
    <div 
      class="red-packet"
      :class="{ 'opened': isOpened }"
      @click="openPacket"
      v-if="!isOpened && !showResult && !showAllOpened && !showSettings"
    >
      <!-- 红包封口 -->
      <div class="packet-seal"></div>
      <!-- 金色圆形开启按钮 -->
      <div class="open-button">開</div>
      <!-- 红包文字 -->
      <div class="packet-text">红包</div>
      <!-- 红包信息 -->
      <div class="packet-info">
        <div class="packet-info-text">{{ totalUsers - receivedList.length }}/{{ totalUsers }}</div>
          </div>
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
      v-if="isOpened && !showResult && !showAllOpened"
    >
      ¥
    </div>
    
    <!-- 单次抢红包结果弹窗 -->
    <div class="result-modal" v-if="showResult && !showAllOpened">
      <div class="modal-content">
        <div class="confetti"></div> <!-- 彩屑背景 -->
        <div class="modal-header">恭喜发财</div>
        <div class="modal-user">{{ currentUser }} 抢到了</div>
        <div class="modal-money">¥{{ money }}</div>
        <div class="modal-desc">剩余 {{ totalUsers - receivedList.length }} 个红包</div>
        <button class="modal-btn" @click="continueGame">继续抢</button>
      </div>
    </div>
    
    <!-- 所有红包抢完后的结果弹窗 -->
    <div class="result-modal" v-if="showAllOpened">
      <div class="modal-content final-result">
        <div class="confetti"></div> <!-- 彩屑背景 -->
        <div class="modal-header">红包已抢完</div>
        <div class="summary-section">
          <div class="summary-item">
            <div class="summary-label">手气最佳</div>
            <div class="summary-user">{{ bestLucky.user }}</div>
            <div class="summary-money">¥{{ bestLucky.money }}</div>
          </div>
          <div class="summary-item">
            <div class="summary-label">运气最差</div>
            <div class="summary-user">{{ worstLucky.user }}</div>
            <div class="summary-money">¥{{ worstLucky.money }}</div>
          </div>
        </div>
        <div class="all-received">
          <div class="received-title">所有红包：</div>
          <div class="received-list">
            <div 
              v-for="(item, index) in receivedList" 
              :key="index" 
              class="received-item"
              :class="{ 'best-item': item.user === bestLucky.user, 'worst-item': item.user === worstLucky.user }"
            >
              <span class="received-user">{{ item.user }}</span>
              <span class="received-money">¥{{ item.money }}</span>
            </div>
          </div>
        </div>
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
      showResult: false,    // 是否显示单次结果弹窗
      showAllOpened: false, // 是否显示所有红包抢完的结果
      showSettings: true,   // 是否显示设置界面
      showRedPacketList: false, // 是否显示红包记录列表
      money: 0,             // 抢到的金额
      coins: [],            // 金币动画数据
      totalMoney: 100,      // 总金额（元）
      totalUsers: 5,        // 总人数（默认值改为5）
      inputMoney: 100,      // 输入的总金额
      inputUsers: 5,        // 输入的总人数
      remainingMoney: 100,  // 剩余金额
      receivedList: [],     // 已抢红包列表
      myRedPackets: [],     // 我的红包记录
      users: [              // 模拟群聊用户
        '我', '小明', '小红', '小李', '小王', '小张', '小刘', '小陈', '小周', '小林',
        '小美', '小丽', '小海', '大山', '小风', '小雨', '小花', '小朵', '小龙', '小虎'
      ],
      currentUser: '',      // 当前抢红包的用户
      bestLucky: { user: '', money: 0 },  // 手气最佳
      worstLucky: { user: '', money: 999 } // 运气最差
    };
  },
  methods: {
    // 重置游戏（可再次抢红包）
    resetGame() {
      this.isOpened = false;
      this.showResult = false;
      this.showAllOpened = false;
      this.coins = [];
      this.remainingMoney = this.totalMoney;
      this.receivedList = [];
      // 注意：不重置我的红包记录，让记录一直保留
      this.bestLucky = { user: '', money: 0 };
      this.worstLucky = { user: '', money: 999 };
      // 重置时显示设置界面
      this.showSettings = true;
      // 重置输入框为当前值
      this.inputMoney = this.totalMoney;
      this.inputUsers = this.totalUsers;
      // 重置后隐藏记录面板，需要重新打开红包才会显示
      this.showRedPacketList = false;
    },
    // 打开红包
    openPacket() {
      if (this.isOpened) return;
      
      // 选择当前用户
      this.selectRandomUser();
      
      // 标记红包为打开状态（触发动画）
      this.isOpened = true;
      
      // 生成金币飞散数据（15个金币）
      this.generateCoins(15);
      
      // 计算本次抢到的金额
      this.calculateMoney();
      
      // 动画结束后显示结果（1.5秒后）
      setTimeout(() => {
        // 记录本次抢红包结果
        this.recordResult();
        
        // 检查是否所有红包都已抢完
        if (this.receivedList.length >= this.totalUsers) {
          this.calculateLuckyUsers();
          this.showAllOpened = true;
        } else {
          this.showResult = true;
        }
      }, 1500);
    },
    
    // 随机选择一个未抢红包的用户
    selectRandomUser() {
      // 获取未抢红包的用户列表
      const receivedUsers = this.receivedList.map(item => item.user);
      const availableUsers = this.users.filter(user => !receivedUsers.includes(user));
      
      // 如果所有用户都已抢过，重新随机选择
      if (availableUsers.length === 0) {
        this.currentUser = this.users[Math.floor(Math.random() * this.users.length)];
      } else {
        this.currentUser = availableUsers[Math.floor(Math.random() * availableUsers.length)];
      }
    },
    
    // 计算本次抢到的金额
    calculateMoney() {
      // 剩余人数
      const remainingUsers = this.totalUsers - this.receivedList.length;
      
      // 确保remainingMoney是数字类型
      const remainingMoneyNum = parseFloat(this.remainingMoney) || 0;
      
      // 最后一个红包直接获取所有剩余金额
      if (remainingUsers === 1) {
        this.money = remainingMoneyNum.toFixed(2);
      } else {
        // 随机生成金额，确保剩余每人至少有0.01元
        const maxMoney = remainingMoneyNum - 0.01 * (remainingUsers - 1);
        this.money = (Math.random() * maxMoney * 0.8 + 0.01).toFixed(2);
        this.money = Math.min(parseFloat(this.money), maxMoney).toFixed(2);
      }
      
      // 更新剩余金额，保持数字类型
      this.remainingMoney = remainingMoneyNum - parseFloat(this.money);
    },
    
    // 记录抢红包结果
    recordResult() {
      this.receivedList.push({
        user: this.currentUser,
        money: parseFloat(this.money)
      });
      
      // 每次打开红包都记录，并显示记录面板
      this.myRedPackets.push({
        user: this.currentUser,
        money: parseFloat(this.money),
        timestamp: new Date().getTime()
      });
      
      // 第一次打开红包后显示记录面板
      this.showRedPacketList = true;
    },
    
    // 格式化时间戳为可读时间
    formatTimestamp(timestamp) {
      const date = new Date(timestamp);
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      const seconds = date.getSeconds().toString().padStart(2, '0');
      return `${hours}:${minutes}:${seconds}`;
    },
    
    // 计算手气最佳和运气最差
    calculateLuckyUsers() {
      // 重置最佳和最差
      this.bestLucky = { user: '', money: 0 };
      this.worstLucky = { user: '', money: 999 };
      
      // 遍历找到最大和最小值
      this.receivedList.forEach(item => {
        if (item.money > this.bestLucky.money) {
          this.bestLucky = { ...item };
        }
        if (item.money < this.worstLucky.money) {
          this.worstLucky = { ...item };
        }
      });
    },
    
    // 继续抢红包
    continueGame() {
      this.isOpened = false;
      this.showResult = false;
      this.coins = [];
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
      this.showAllOpened = false;
      this.coins = [];
      this.remainingMoney = this.totalMoney;
      this.receivedList = [];
      // 注意：不重置我的红包记录，让记录一直保留
      this.bestLucky = { user: '', money: 0 };
      this.worstLucky = { user: '', money: 999 };
      // 重置时显示设置界面
      this.showSettings = true;
      // 重置输入框为当前值
      this.inputMoney = this.totalMoney;
      this.inputUsers = this.totalUsers;
      // 重置后隐藏记录面板，需要重新打开红包才会显示
      this.showRedPacketList = false;
    },
    
    // 保存设置
    saveSettings() {
      // 验证并设置总金额
      if (this.inputMoney) {
        this.totalMoney = Math.min(Math.max(this.inputMoney, 0.01), 200);
      } else {
        this.totalMoney = 100; // 默认值
      }
      
      // 验证并设置总人数
      if (this.inputUsers) {
        this.totalUsers = Math.min(Math.max(this.inputUsers, 1), 20);
      } else {
        this.totalUsers = 5; // 默认值
      }
      
      // 重置其他状态
      this.remainingMoney = this.totalMoney;
      this.receivedList = [];
      this.isOpened = false;
      this.showResult = false;
      this.showAllOpened = false;
      this.coins = [];
      this.bestLucky = { user: '', money: 0 };
      this.worstLucky = { user: '', money: 999 };
      
      // 隐藏设置界面
      this.showSettings = false;
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
  width: 192px;
  height: 260px;
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

/* 红包信息（剩余数量） */
.packet-info {
  position: absolute;
  top: 85%;
  left: 50%;
  transform: translateX(-50%);
  color: #fff;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.packet-info-text {
  background: rgba(0, 0, 0, 0.3);
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 16px;
}

/* 我的红包记录样式 */
.my-red-packets {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  width: 200px;
  max-height: 70vh;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  overflow: hidden;
  z-index: 50;
}

.my-red-packets-title {
  padding: 15px;
  background: #e63946;
  color: white;
  font-weight: bold;
  font-size: 16px;
  text-align: center;
}

.my-red-packets-list {
  max-height: calc(70vh - 50px);
  overflow-y: auto;
  padding: 10px;
}

.my-red-packet-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  margin-bottom: 8px;
  background: #f8f9fa;
  border-radius: 8px;
  transition: all 0.2s ease;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.my-red-packet-item:hover {
  background: #e9ecef;
  transform: translateX(-2px);
}

.packet-user {
  font-size: 14px;
  font-weight: 500;
  color: #333;
  flex: 0 0 auto;
  margin-right: 10px;
}

.packet-time {
  font-size: 12px;
  color: #666;
  flex: 1;
  text-align: center;
}

.packet-amount {
  font-size: 16px;
  font-weight: bold;
  color: #d62828;
  flex: 0 0 auto;
  margin-left: 10px;
}

.empty-record {
  text-align: center;
  padding: 20px;
  color: #999;
  font-size: 14px;
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

/* 当前用户显示 */
.modal-user {
  color: #666;
  font-size: 16px;
  margin-bottom: 10px;
}

/* 弹窗描述 */
.modal-desc {
  color: #666;
  margin-bottom: 30px;
  font-size: 16px;
}

/* 设置界面样式 */
.settings-modal {
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

.settings-form {
  width: 100%;
  margin-top: 20px;
}

.form-group {
  margin-bottom: 20px;
  text-align: left;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #333;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  box-sizing: border-box;
  transition: border-color 0.3s ease;
}

.form-group input:focus {
  outline: none;
  border-color: #e63946;
}

.form-hint {
  display: block;
  margin-top: 5px;
  font-size: 12px;
  color: #999;
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



/* 最终结果弹窗样式 */
.final-result {
  max-width: 400px;
  max-height: 80vh;
  overflow-y: auto;
}

/* 总结部分 */
.summary-section {
  display: flex;
  justify-content: space-around;
  margin: 20px 0;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 12px;
}

.summary-item {
  text-align: center;
}

.summary-label {
  font-size: 14px;
  color: #666;
  margin-bottom: 5px;
}

.summary-user {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}

.summary-money {
  font-size: 22px;
  font-weight: bold;
  color: #d62828;
}

/* 所有红包列表 */
.all-received {
  margin-top: 20px;
}

.received-title {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 10px;
  text-align: left;
}

.received-list {
  max-height: 300px;
  overflow-y: auto;
}

.received-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  margin-bottom: 5px;
  background: #f8f9fa;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.received-user {
  font-weight: 500;
}

.received-money {
  font-weight: bold;
  color: #d62828;
}

/* 特殊标识 */
.best-item {
  background: linear-gradient(135deg, #fffacd, #fff8dc);
  border: 1px solid #ffd700;
}

.worst-item {
  background: linear-gradient(135deg, #f0f8ff, #e6f3ff);
  border: 1px solid #4cc9f0;
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