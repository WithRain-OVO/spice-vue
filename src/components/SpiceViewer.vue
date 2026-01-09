<template>
  <div class="spice-container">
    <div id="login">
      <button @click="openNav">&#9776; SPICE</button>
      <p id="hostname">{{ title }}</p>
    </div>

    <div id="Sidenav" :class="{'SidenavClosed': !isNavOpen, 'SidenavOpen': isNavOpen}">
      <p class="closebtn" @click="closeNav">&#10006;</p>
      <label for="host">Host:</label> <input type='text' id='host' v-model='host' value='localhost'><br>
      <label for="port">Port:</label> <input type='text' id='port' v-model='port' value='5959'><br>
      <label for="password">Password:</label> <input type='password' id='password' v-model='password' value=''><br>
      <button id="connectButton" @click="toggleConnection">{{ isConnected ? 'Stop Connection' : 'Start Connection' }}</button><br>
      <button id="sendCtrlAltDel" @click="sendCtrlAltDel">Send Ctrl-Alt-Delete</button>
      <button id="debugLogs" @click="toggleDebugLogs">Toggle Debug Logs</button>
      <div id="message-div" class="spice-message" :style="{display: isDebugVisible ? 'block' : 'none'}"></div>
      <div id="debug-div">
        <!-- If DUMPXXX is turned on, dumped images will go here -->
      </div>
    </div>

    <div id="spice-area">
      <div id="spice-screen" class="spice-screen"></div>
    </div>
  </div>
</template>

<script>
// 直接使用全局的SpiceHtml5对象，该对象在index.html中加载
let sc = null;

export default {
  name: 'SpiceViewer',
  props: {
    title: {
      type: String,
      default: 'Host Console'
    }
  },
  data() {
    return {
      host: 'localhost',
      port: '5959',
      password: '',
      isConnected: false,
      isNavOpen: false,
      isDebugVisible: false
    };
  },
  mounted() {
    // 检查全局SpiceHtml5对象是否已经加载
    if (window.SpiceHtml5) {
      console.log('SpiceHtml5 already loaded from global scope');
    } else {
      console.warn('SpiceHtml5 not found in global scope, waiting for it to load...');
    }
    
    /**
     * 从URL参数中读取host和port
     */
    const urlParams = new URLSearchParams(window.location.search);
    const urlHost = urlParams.get('host');
    const urlPort = urlParams.get('port');
    
    if (urlHost) {
      this.host = urlHost;
    }
    if (urlPort) {
      this.port = urlPort;
    }
  },
  beforeDestroy() {
    // 断开连接并清理资源
    this.disconnect();
  },
  methods: {
    /**
     * 打开侧边导航栏
     */
    openNav() {
      this.isNavOpen = true;
    },
    
    /**
     * 关闭侧边导航栏
     */
    closeNav() {
      this.isNavOpen = false;
    },
    
    /**
     * 切换连接状态
     */
    toggleConnection() {
      if (this.isConnected) {
        this.disconnect();
      } else {
        this.connect();
      }
    },
    
    /**
     * 连接到SPICE服务器
     */
    connect() {
      if (!this.host || !this.port) {
        console.log("must set host and port");
        return;
      }

      if (sc) {
        sc.stop();
      }

      const scheme = "ws://";
      const uri = scheme + this.host + ":" + this.port;

      this.isConnected = true;

      try {
        sc = new window.SpiceHtml5.SpiceMainConn({
          uri: uri, 
          screen_id: "spice-screen", 
          dump_id: "debug-div",
          message_id: "message-div", 
          password: this.password, 
          onerror: this.spiceError, 
          onagent: this.agentConnected 
        });
      } catch (e) {
        alert(e.toString());
        this.disconnect();
      }
    },
    
    /**
     * 断开与SPICE服务器的连接
     */
    disconnect() {
      console.log(">> disconnect");
      if (sc) {
        sc.stop();
        sc = null;
      }
      this.isConnected = false;
      
      // 清理文件传输区域
      if (window.File && window.FileReader && window.FileList && window.Blob) {
        const spiceXferArea = document.getElementById('spice-xfer-area');
        if (spiceXferArea != null) {
          document.getElementById('spice-area').removeChild(spiceXferArea);
        }
        document.getElementById('spice-area').removeEventListener('dragover', window.SpiceHtml5.handle_file_dragover, false);
        document.getElementById('spice-area').removeEventListener('drop', window.SpiceHtml5.handle_file_drop, false);
      }
      console.log("<< disconnect");
    },
    
    /**
     * SPICE连接错误处理
     */
    spiceError(e) {
      console.error('Spice error:', e);
      this.disconnect();
    },
    
    /**
     * 发送Ctrl-Alt-Delete命令
     */
    sendCtrlAltDel() {
      if (sc && window.SpiceHtml5) {
        window.SpiceHtml5.sendCtrlAltDel(sc);
      }
    },
    
    /**
     * 切换调试日志可见性
     */
    toggleDebugLogs() {
      this.isDebugVisible = !this.isDebugVisible;
    },
    
    /**
     * 代理连接回调
     */
    agentConnected(connection) {
      window.addEventListener('resize', window.SpiceHtml5.handle_resize);
      window.spice_connection = this;

      window.SpiceHtml5.resize_helper(connection);

      if (window.File && window.FileReader && window.FileList && window.Blob) {
        const spiceXferArea = document.createElement("div");
        spiceXferArea.setAttribute('id', 'spice-xfer-area');
        document.getElementById('spice-area').appendChild(spiceXferArea);
        document.getElementById('spice-area').addEventListener('dragover', window.SpiceHtml5.handle_file_dragover, false);
        document.getElementById('spice-area').addEventListener('drop', window.SpiceHtml5.handle_file_drop, false);
      } else {
        console.log("File API is not supported");
      }
    }
  }
};
</script>

<style scoped>
/* 这里可以添加组件特定的样式，spice.css会全局加载 */
.spice-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>