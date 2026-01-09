<template>
  <div class="home-page">
    <div class="home-container">
      <h1>SPICE HTML5 客户端</h1>
      <p class="description">点击下方按钮连接到远程桌面</p>
      <button class="connect-button" @click="openSpiceViewer">
        打开 SPICE 客户端
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomePage',
  methods: {
    /**
     * 打开SPICE客户端页面
     */
    openSpiceViewer() {
      const host = '192.168.106.142';
      const port = '5959';
      const url = `${window.location.origin}${window.location.pathname}?page=spice&host=${host}&port=${port}`;
      window.open(url, '_blank');
    }
  }
}
</script>

<style scoped>
.home-page {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.home-container {
  text-align: center;
  background: white;
  padding: 60px 80px;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-width: 500px;
}

h1 {
  color: #333;
  margin-bottom: 20px;
  font-size: 2.5em;
}

.description {
  color: #666;
  margin-bottom: 40px;
  font-size: 1.2em;
}

.connect-button {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 15px 40px;
  font-size: 1.1em;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.connect-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
}

.connect-button:active {
  transform: translateY(0);
}
</style>