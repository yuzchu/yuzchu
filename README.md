<!DOCTYPE html>
<html>
<head>
  <meta charset='utf-8'>
  <meta name='viewport' content='width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover'>
  <title>AI Assistant</title>
  <link rel='stylesheet' href='https://unpkg.com/vant@4/lib/index.css'/>
  <script src='https://unpkg.com/vue@3'></script>
  <script src='https://unpkg.com/vant@4/lib/vant.min.js'></script>
  <style>
    :root { 
      --primary-blue: #0066FF; 
      --bottom-bar-height: 60px;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    html { 
      height: 100%; 
      overflow: hidden;
    }
    
    body { 
      height: 100%;
      background: #f7f8fa; 
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      overflow: hidden;
    }
    
    #app {
      height: 100%;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }
    
    .van-nav-bar { 
      background-color: var(--primary-blue) !important; 
      z-index: 999 !important;
      flex-shrink: 0;
    }
    .van-nav-bar__title, .van-nav-bar .van-icon { color: #fff !important; }

    /* ====== 聊天区域 ====== */
    .chat-container { 
      flex: 1;
      overflow-y: auto;
      overflow-x: hidden;
      -webkit-overflow-scrolling: touch;
      padding: 10px 12px 20px 12px;
      background: #f7f8fa;
      transition: padding-bottom 0.2s ease-out;
    }
    
    .msg { 
      margin-bottom: 14px; 
      width: 100%;
    }
    
    .msg-bubble { 
      word-break: break-word; 
      white-space: pre-wrap; 
      width: 100%;
      padding: 12px 14px; 
      border-radius: 12px; 
      font-size: 15px; 
      line-height: 1.6;
      position: relative;
    }
    
    .msg.user .msg-bubble {
      background: #e6f0ff;
      color: #1a5fb4;
      border: 1px solid #c2d9ff;
    }
    .msg.ai .msg-bubble { 
      background: white; 
      border: 1px solid #e8e8e8; 
    }
    .msg.system .msg-bubble {
      background: #f5f5f5;
      font-size: 13px;
      color: #666;
    }

    /* 钉住消息的样式 */
    .msg.pinned .msg-bubble {
      border-left: 3px solid #ff9800;
    }
    .msg.pinned .msg-bubble::before {
      content: '📌';
      position: absolute;
      top: -8px;
      left: -8px;
      font-size: 14px;
    }

    /* ====== 消息操作按钮 ====== */
    .msg-actions {
      position: absolute;
      top: 4px;
      right: 4px;
      display: flex;
      gap: 2px;
      opacity: 0;
      transition: opacity 0.2s;
      background: rgba(255,255,255,0.9);
      border-radius: 12px;
      padding: 2px;
    }
    .msg-bubble:active .msg-actions,
    .msg-bubble:hover .msg-actions {
      opacity: 1;
    }
    .msg-action-btn {
      width: 24px;
      height: 24px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
      transition: background 0.2s;
    }
    .msg-action-btn:active {
      background: rgba(0,0,0,0.1);
    }
    .msg-action-btn .van-icon {
      font-size: 14px;
      color: #999;
    }
    .msg-action-btn.active .van-icon {
      color: #ff9800;
    }
    .msg-action-btn.delete:active .van-icon {
      color: #ee0a24;
    }
    .msg-action-btn.edit:active .van-icon {
      color: var(--primary-blue);
    }
    .msg-action-btn.add:active .van-icon {
      color: #07c160;
    }

    /* ====== 底部输入栏 ====== */
    .bottom-bar {
      flex-shrink: 0;
      background: #fff; 
      padding: 10px 12px;
      border-top: 1px solid #eee; 
      z-index: 998;
      transition: transform 0.2s ease-out;
      position: relative; 
      bottom: 0;
      left: 0;
      right: 0;
      width: 100%;
      box-sizing: border-box;
      padding-bottom: calc(10px + env(safe-area-inset-bottom, 0));
    }
    
    .keyboard-open .bottom-bar {
      position: fixed;
      transform: translateY(0) !important;
    }

    /* ====== 历史记录开关行 ====== */
    .history-toggle-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 6px 4px 10px 4px;
      font-size: 13px;
      color: #666;
    }
    .history-toggle-row .toggle-label {
      display: flex;
      align-items: center;
      gap: 6px;
    }
    .history-toggle-row .toggle-label .van-icon {
      color: var(--primary-blue);
    }
    .history-count {
      font-size: 12px;
      color: #999;
      background: #f0f0f0;
      padding: 2px 8px;
      border-radius: 10px;
    }
    
    /* 历史操作按钮 */
    .history-action-btn {
      width: 28px;
      height: 28px;
      border-radius: 50%;
      background: #f5f5f5;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
      transition: background 0.2s;
    }
    .history-action-btn:active {
      background: #e0e0e0;
    }
    .history-action-btn .van-icon {
      font-size: 16px;
      color: #666;
    }
    .history-action-btn.copy-btn .van-icon {
      color: var(--primary-blue);
    }
    .history-action-btn.clear-btn .van-icon {
      color: #ee0a24;
    }
    
    .input-row {
      display: flex; 
      align-items: center; 
      gap: 10px;
    }
    .input-wrapper {
      flex: 1;
      background: #f5f5f5;
      border-radius: 20px; 
      padding: 8px 14px;
    }
    .input-wrapper textarea {
      width: 100%;
      border: none;
      background: transparent;
      font-size: 16px;
      line-height: 1.4;
      resize: none;
      outline: none;
      min-height: 24px;
      max-height: 100px;
      font-family: inherit;
      -webkit-appearance: none;
    }
    
    .send-btn {
      width: 56px;
      height: 40px;
      background: var(--primary-blue);
      color: #fff;
      border: none;
      border-radius: 20px;
      font-size: 14px;
      font-weight: 500;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
      -webkit-tap-highlight-color: transparent;
      transition: all 0.2s ease;
    }
    .send-btn:active { opacity: 0.8; }
    .send-btn:disabled { background: #ccc; cursor: not-allowed; }
    
    /* 停止按钮样式 */
    .send-btn.is-loading {
      background: #ee0a24;
      animation: pulse 1.5s infinite;
    }
    .send-btn.is-loading:active {
      background: #d40a21;
      animation: none;
    }
    
    @keyframes pulse {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.7; }
    }
    
    /* 停止图标 - 方形 */
    .stop-icon {
      width: 14px;
      height: 14px;
      background: #fff;
      border-radius: 2px;
    }

    /* ====== 悬浮按钮 ====== */
    .floating-tools {
    position: fixed;
    right: 3px;
    bottom: 140px;  /* 稍微调高一点 */
    display: flex; 
    flex-direction: column; 
    gap: 8px;
    z-index: 1000;
    transition: bottom 0.2s ease-out;
}
    .floating-tools.keyboard-open {
      display: none;
    }
    .tool-icon-btn {
      width: 24px; 
      height: 24px; 
      background: transparent;
      display: flex; 
      align-items: center; 
      justify-content: center; 
      font-size: 18px;
      cursor: pointer;
      opacity: 0.5;
    }
    .tool-icon-btn:active { opacity: 1; transform: scale(0.9); }
    .btn-gray { color: #888; }
    .btn-blue { color: var(--primary-blue); }
    .btn-orange { color: #ff976a; }
    .btn-red { color: #ee0a24; }
    .btn-green { color: #07c160; }

    /* ====== 工具折叠样式 ====== */
    details.tool-output { 
      border: 1px solid #eee; 
      border-radius: 8px; 
      padding: 8px; 
      background: #fafafa; 
      margin-top: 8px;
    }
    details.tool-output summary { 
      color: var(--primary-blue); 
      cursor: pointer; 
      font-weight: 500; 
      font-size: 13px; 
    }
    details.tool-output pre { 
      margin: 8px 0 0; 
      white-space: pre-wrap; 
      word-break: break-all; 
      font-size: 12px; 
      color: #666; 
      background: #fff; 
      padding: 8px; 
      border-radius: 4px;
    }
    /* 指令标签 - 简洁小巧 */
    .cmd-chip {
      display: inline-flex;
      align-items: center;
      gap: 4px;
      padding: 3px 10px;
      margin: 2px 0;
      background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
      color: #fff;
      border-radius: 12px;
      font-size: 11px;
      cursor: pointer;
      vertical-align: middle;
      box-shadow: 0 1px 2px rgba(99, 102, 241, 0.25);
    }
    .cmd-chip:active {
      opacity: 0.85;
    }
    .cmd-detail {
      margin: 4px 0;
      border-radius: 6px;
      overflow: hidden;
      border: 1px solid #e5e7eb;
      background: #fafafa;
    }
    .cmd-detail summary {
      display: flex;
      align-items: center;
      padding: 6px 10px;
      cursor: pointer;
      font-size: 11px;
      color: #9ca3af;
      list-style: none;
    }
    .cmd-detail summary::-webkit-details-marker { display: none; }
    .cmd-detail pre {
      margin: 0;
      padding: 8px 10px;
      background: #fff;
      white-space: pre-wrap;
      word-break: break-all;
      color: #64748b;
      font-size: 10px;
      line-height: 1.4;
      border-top: 1px solid #e5e7eb;
      max-height: 150px;
      overflow-y: auto;
    }
    /* 工具请求折叠样式 - 紫色系 */
    .tool-request {
      margin: 4px 0;
      border-radius: 6px;
      overflow: hidden;
      border: 1px solid #e0d4fc;
    }
    .tool-request summary {
      display: flex;
      align-items: center;
      padding: 8px 10px;
      cursor: pointer;
      font-size: 12px;
      list-style: none;
      background: #f3efff;
      color: #6b46c1;
    }
    .tool-request summary::-webkit-details-marker { display: none; }
    .tool-request pre {
      margin: 0;
      padding: 8px 10px;
      background: #fff;
      white-space: pre-wrap;
      word-break: break-all;
      color: #374151;
      font-size: 10px;
      line-height: 1.4;
      border-top: 1px solid #e0d4fc;
      max-height: 150px;
      overflow-y: auto;
    }
    
    .tool-result {
      margin: 4px 0;
      border-radius: 6px;
      overflow: hidden;
      border: 1px solid #e5e7eb;
    }
    .tool-result summary {
      display: flex;
      align-items: center;
      padding: 8px 10px;
      cursor: pointer;
      font-size: 12px;
      list-style: none;
    }
    .tool-result summary::-webkit-details-marker { display: none; }
    .tool-result.success summary {
      background: #ecfdf5;
      color: #065f46;
    }
    .tool-result.error summary {
      background: #fef2f2;
      color: #991b1b;
    }
    .tool-result pre {
      margin: 0;
      padding: 8px 10px;
      background: #fff;
      white-space: pre-wrap;
      word-break: break-all;
      color: #374151;
      font-size: 10px;
      line-height: 1.4;
      border-top: 1px solid #e5e7eb;
      max-height: 150px;
      overflow-y: auto;
    }
    .exec-status {
      display: inline-flex;
      align-items: center;
      gap: 5px;
      padding: 5px 10px;
      border-radius: 12px;
      font-size: 12px;
    }
    .exec-status.running {
      background: #fef3c7;
      color: #92400e;
    }
   

    .status-card { 
      padding: 10px 12px; 
      background: #f9f9f9; 
      border-radius: 8px; 
      font-size: 13px; 
      color: #666; 
      border: 1px solid #eee; 
    }
    .status-card.success { background: #e6fffa; border-color: #b2f5ea; color: #2c7a7b; }
    .status-card.error { background: #fff5f5; border-color: #feb2b2; color: #c53030; }

    /* ====== 侧边栏样式 ====== */
    .sidebar-container { 
      height: 100%; 
      display: flex; 
      flex-direction: column; 
      background: #fff; 
      padding: 16px;
    }
    .sidebar-header {
      display: flex; 
      justify-content: space-between; 
      align-items: center; 
      margin-bottom: 16px;
      padding-bottom: 12px; 
      border-bottom: 1px solid #f0f0f0;
    }
    .sidebar-title { margin: 0; color: var(--primary-blue); font-size: 18px; font-weight: 600; }
    .sidebar-actions { display: flex; gap: 10px; margin-bottom: 16px; }

    .action-dot-btn {
      display: flex; 
      align-items: center; 
      justify-content: center;
      width: 36px; 
      height: 36px; 
      min-width: 36px;
      border-radius: 50%; 
      background: #f5f5f5;
      color: #666; 
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
    }
    .action-dot-btn:active { background: #e8e8e8; transform: scale(0.95); }

    .session-item {
      display: flex; 
      align-items: center; 
      padding: 12px 10px;
      background: #fff; 
      border-radius: 10px; 
      margin-bottom: 8px;
      border: 1px solid #f0f0f0; 
      gap: 10px;
    }
    .session-item.active { 
      border-color: var(--primary-blue); 
      background: linear-gradient(135deg, #f0f7ff 0%, #e8f4ff 100%); 
    }
    .session-item:active { background: #fafafa; }
    .session-content { flex: 1; min-width: 0; cursor: pointer; }
    .session-title-row { display: flex; align-items: center; gap: 6px; }
    .session-title { 
      font-size: 14px; 
      color: #333; 
      white-space: nowrap; 
      overflow: hidden; 
      text-overflow: ellipsis;
      flex: 1;
    }
    .session-meta { font-size: 11px; color: #999; margin-top: 4px; }
    .pill {
      font-size: 10px; 
      padding: 2px 6px; 
      border-radius: 999px;
      background: #fff3e0; 
      color: #ff9800; 
      border: 1px solid #ffe0b2;
      white-space: nowrap;
      flex-shrink: 0;
    }

    .group-header {
      display: flex; 
      align-items: center; 
      padding: 10px 0;
      border-bottom: 1px solid #f0f0f0; 
      margin-bottom: 8px;
    }
    .group-title { 
      flex: 1; 
      font-size: 15px; 
      font-weight: 600; 
      color: #333; 
      cursor: pointer; 
    }
    .group-toggle { 
      width: 28px; 
      height: 28px; 
      display: flex; 
      align-items: center; 
      justify-content: center;
      margin-right: 6px; 
      color: #999; 
      transition: transform 0.2s; 
      cursor: pointer;
    }
    .group-toggle.expanded { transform: rotate(90deg); }

    .group-add-btn {
      display: flex; 
      align-items: center; 
      justify-content: center; 
      gap: 6px;
      padding: 12px; 
      margin-bottom: 8px;
      border: 1px dashed #ccc; 
      border-radius: 10px;
      color: #666; 
      font-size: 13px; 
      cursor: pointer;
      background: #fafafa;
      -webkit-tap-highlight-color: transparent;
    }
    .group-add-btn:active { background: #f0f0f0; }
    .empty-tip { 
      color: #aaa; 
      font-size: 13px; 
      padding: 16px 0; 
      text-align: center; 
    }

    body.sidebar-open .floating-tools {
      display: none !important;
    }

    /* ====== 自定义弹出层样式 ====== */
    .custom-overlay {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(0, 0, 0, 0.5);
      z-index: 99999;
      display: flex;
      align-items: flex-end;
      justify-content: center;
      -webkit-tap-highlight-color: transparent;
    }
    
    .custom-overlay.center {
      align-items: center;
    }

    .custom-action-sheet {
      width: 100%;
      background: #fff;
      border-radius: 16px 16px 0 0;
      overflow: hidden;
      max-height: 70vh;
      overflow-y: auto;
      -webkit-overflow-scrolling: touch;
      padding-bottom: env(safe-area-inset-bottom, 0);
      animation: slideUp 0.25s ease-out;
    }

    @keyframes slideUp {
      from { transform: translateY(100%); }
      to { transform: translateY(0); }
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: scale(0.9); }
      to { opacity: 1; transform: scale(1); }
    }

    .action-sheet-title {
      padding: 16px;
      text-align: center;
      font-size: 14px;
      color: #999;
      border-bottom: 1px solid #eee;
    }

    .action-item {
      padding: 16px;
      text-align: center;
      font-size: 16px;
      border-bottom: 1px solid #f5f5f5;
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
      transition: background 0.15s;
    }

    .action-item:active {
      background: #f5f5f5;
    }

    .action-cancel {
      padding: 16px;
      text-align: center;
      font-size: 16px;
      color: #666;
      cursor: pointer;
      margin-top: 8px;
      background: #f7f8fa;
      -webkit-tap-highlight-color: transparent;
    }
    
    .action-cancel:active {
      background: #eee;
    }

    .custom-dialog {
      width: 85%;
      max-width: 320px;
      background: #fff;
      border-radius: 16px;
      padding: 24px 20px 20px;
      animation: fadeIn 0.2s ease-out;
    }

    .dialog-title {
      font-size: 17px;
      font-weight: 600;
      text-align: center;
      margin-bottom: 16px;
      color: #333;
    }

    .dialog-message {
      font-size: 14px;
      color: #666;
      text-align: center;
      margin-bottom: 24px;
      line-height: 1.5;
    }

    .dialog-input {
      width: 100%;
      padding: 12px 14px;
      border: 1px solid #ddd;
      border-radius: 8px;
      font-size: 16px;
      margin-bottom: 20px;
      outline: none;
      -webkit-appearance: none;
      transition: border-color 0.2s;
    }

    .dialog-input:focus {
      border-color: var(--primary-blue);
    }
    
    .dialog-textarea {
      width: 100%;
      padding: 12px 14px;
      border: 1px solid #ddd;
      border-radius: 8px;
      font-size: 14px;
      margin-bottom: 20px;
      outline: none;
      -webkit-appearance: none;
      transition: border-color 0.2s;
      resize: none;
      min-height: 120px;
      max-height: 200px;
      font-family: inherit;
      line-height: 1.5;
    }

    .dialog-textarea:focus {
      border-color: var(--primary-blue);
    }

    .dialog-buttons {
      display: flex;
      gap: 12px;
    }

    .dialog-btn {
      flex: 1;
      padding: 12px;
      border: none;
      border-radius: 22px;
      font-size: 15px;
      font-weight: 500;
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
      transition: opacity 0.15s;
    }

    .dialog-btn.cancel {
      background: #f5f5f5;
      color: #666;
    }
    
    .dialog-btn.cancel:active {
      background: #eee;
    }

    .dialog-btn.confirm {
      background: var(--primary-blue);
      color: #fff;
    }
    
    .dialog-btn.confirm:active {
      opacity: 0.8;
    }
    
    /* 角色选择器 */
    .role-selector {
      display: flex;
      gap: 8px;
      margin-bottom: 12px;
    }
    .role-option {
      flex: 1;
      padding: 8px 12px;
      border: 1px solid #ddd;
      border-radius: 8px;
      text-align: center;
      font-size: 13px;
      cursor: pointer;
      transition: all 0.2s;
    }
    .role-option.active {
      border-color: var(--primary-blue);
      background: #e6f0ff;
      color: var(--primary-blue);
    }
    .role-option:active {
      opacity: 0.8;
    }
    
    .btn-green {
        color: #07c160;
    }
    
  </style>
</head>

<body>
<div id='app'>
  <!-- 顶部导航栏 -->
  <van-nav-bar :title='currentModelName' @click-left="openSidebar">
    <template #left><van-icon name='wap-nav' size='22' /></template>
    <template #right><van-icon name='setting-o' size='22' @click='openSettings' /></template>
  </van-nav-bar>

  <!-- 聊天内容区域 -->
  <div class='chat-container' ref="chatContainer" :style="chatContainerStyle">
    <template v-for='(m, i) in messages' :key='i'>
      <div :class='["msg", m.role, { pinned: m.pinned }]'>
        <div class='msg-bubble'>
          <!-- 操作按钮组 -->
          <div class="msg-actions" v-if="!loading">
            <div class="msg-action-btn edit" @click.stop="editMessage(i)" title="编辑">
              <van-icon name="edit" />
            </div>
            <div class="msg-action-btn" :class="{ active: m.pinned }" @click.stop="togglePin(i)" title="钉住">
              <van-icon :name="m.pinned ? 'star' : 'star-o'" />
            </div>
            <div class="msg-action-btn add" @click.stop="addMessageBelow(i)" title="在下方添加">
              <van-icon name="plus" />
            </div>
            <div class="msg-action-btn delete" @click.stop="deleteMessage(i)" title="删除">
              <van-icon name="cross" />
            </div>
          </div>
          <!-- 消息内容：使用安全渲染 -->
          <div v-html="getDisplayContent(m)"></div>
        </div>
      </div>
    </template>
    <template v-if='loading'>
      <div class='msg ai'>
        <div class='msg-bubble'><van-loading size='18px' /> AI正在思考...</div>
      </div>
    </template>
    <div ref="scrollAnchor" style="height: 1px;"></div>
  </div>

  <!-- 悬浮工具按钮 -->
<div class="floating-tools" :class="{ 'keyboard-open': keyboardOpen }">
    <div class="tool-icon-btn btn-green" @click="openWorkspaceManager"><van-icon name="notes-o" /></div>


    <div class="tool-icon-btn btn-blue" @click="openToolsManager"><van-icon name="apps-o" /></div>
    <div class="tool-icon-btn btn-orange" @click="summarizeHistory"><van-icon name="records-o" /></div>
    <div class="tool-icon-btn btn-gray" @click="scrollToBottom"><van-icon name="arrow-down" /></div>
</div>

  <!-- 底部输入栏 -->
  <div class='bottom-bar' ref="bottomBar" :style="bottomBarStyle">
    <!-- 历史记录开关 -->
    <div class="history-toggle-row">
      <div class="toggle-label">
        <van-icon name="clock-o" />
        <span>携带历史</span>
      </div>
      <div style="display: flex; align-items: center; gap: 6px;">
        <div class="history-action-btn copy-btn" @click="copyAllHistory" title="复制历史">
          <van-icon name="orders-o" />
        </div>
        <div class="history-action-btn clear-btn" @click="clearUnpinnedMessages" title="清理未钉住">
          <van-icon name="brush-o" />
        </div>
        <span class="history-count" v-if="includeHistory">{{ messages.length }} 条</span>
        <van-switch v-model="includeHistory" size="20px" @change="onHistoryToggleChange" />
      </div>
    </div>
    
    <div class="input-row">
      <div class="input-wrapper">
        <textarea
          ref="inputRef"
          v-model="userInput"
          placeholder="请输入您的问题..."
          rows="1"
          @input="autoResize"
          @focus="onInputFocus"
          @blur="onInputBlur"
        ></textarea>
      </div>
      <button 
        class="send-btn" 
        :class="{ 'is-loading': loading }"
        :disabled="!userInput.trim() && !loading"
        @click="handleSendClick"
      >
        <template v-if="loading">
          <div class="stop-icon"></div>
        </template>
        <template v-else>发送</template>
      </button>
    </div>
  </div>

  <!-- ====== 侧边栏 Popup ====== -->
  <van-popup 
    v-model:show="showSidebar" 
    position="left" 
    :style="{ width: '85%', height: '100%' }"
    @open="onSidebarOpen"
    @close="onSidebarClose"
  >
    <div class="sidebar-container">
      <div class="sidebar-header">
        <h3 class="sidebar-title">对话管理</h3>
        <van-icon name="cross" size="22" @click="showSidebar = false" style="padding: 8px; cursor: pointer;" />
      </div>

      <div class="sidebar-actions">
        <van-button type="primary" plain size="small" icon="plus" round @click="handleAddGroup" style="flex: 1;">新建分组</van-button>
        <van-button type="default" size="small" icon="exchange" round @click="openQuickSwitch" style="flex: 1;">切换模型</van-button>
      </div>

      <div style="flex: 1; overflow-y: auto; -webkit-overflow-scrolling: touch;">
        <van-loading v-if="sidebarLoading" size="20px" style="margin: 30px 0; text-align: center;">加载中...</van-loading>

        <template v-for="group in orderedGroups" :key="group.id">
          <div style="margin-bottom: 16px;">
            <div class="group-header">
              <div class="group-toggle" :class="{ expanded: activeGroups.includes(group.id) }" @click="toggleGroup(group.id)">
                <van-icon name="arrow" />
              </div>
              <div class="group-title" @click="toggleGroup(group.id)">{{ group.name }}</div>
              <div class="action-dot-btn" @click.stop="openGroupMenu(group)">
                <van-icon name="ellipsis" size="18" />
              </div>
            </div>

            <template v-if="activeGroups.includes(group.id)">
              <div class="group-add-btn" @click="newChatInGroup(group.id)">
                <van-icon name="plus" size="16" />
                <span>在此分组新建会话</span>
              </div>

              <div v-if="getGroupSessions(group.id).length === 0" class="empty-tip">暂无会话</div>

              <template v-for="chat in getGroupSessions(group.id)" :key="chat.id">
                <div class="session-item" :class="{ active: chat.id === currentSessionId }">
                  <div class="session-content" @click="selectChat(chat)">
                    <div class="session-title-row">
                      <span class="pill" v-if="chat.isTop">置顶</span>
                      <span class="session-title">{{ chat.title || '未命名会话' }}</span>
                    </div>
                    <div class="session-meta" v-if="chat.updatedAt">{{ chat.updatedAt }}</div>
                  </div>
                  <div class="action-dot-btn" @click.stop="openSessionMenu(chat)">
                    <van-icon name="ellipsis" size="18" />
                  </div>
                </div>
              </template>
            </template>
          </div>
        </template>
      </div>

      <div style="padding-top: 12px; border-top: 1px solid #f0f0f0; font-size: 13px; color: #666;">
        当前：<span style="color: var(--primary-blue); font-weight: 600;">{{ currentSessionTitle }}</span>
      </div>
    </div>
  </van-popup>

  <!-- ====== 自定义 ActionSheet ====== -->
  <div class="custom-overlay" v-if="actionSheetShow" @click="closeActionSheet">
    <div class="custom-action-sheet" @click.stop>
      <div class="action-sheet-title" v-if="actionSheetTitle">{{ actionSheetTitle }}</div>
      <div 
        class="action-item" 
        v-for="(action, index) in actionSheetItems" 
        :key="index"
        :style="{ color: action.color || '#333' }"
        @click="onActionSelect(action)"
      >
        {{ action.name }}
      </div>
      <div class="action-cancel" @click="closeActionSheet">取消</div>
    </div>
  </div>

  <!-- ====== 自定义输入对话框 ====== -->
  <div class="custom-overlay center" v-if="inputDialogShow" @click="cancelInputDialog">
    <div class="custom-dialog" @click.stop>
      <div class="dialog-title">{{ inputDialogTitle }}</div>
      <input 
        ref="dialogInputRef"
        class="dialog-input" 
        v-model="inputDialogValue" 
        :placeholder="inputDialogPlaceholder"
        @keyup.enter="confirmInputDialog"
      />
      <div class="dialog-buttons">
        <button class="dialog-btn cancel" @click="cancelInputDialog">取消</button>
        <button class="dialog-btn confirm" @click="confirmInputDialog">确定</button>
      </div>
    </div>
  </div>

  <!-- ====== 自定义确认对话框 ====== -->
  <div class="custom-overlay center" v-if="confirmDialogShow" @click="cancelConfirmDialog">
    <div class="custom-dialog" @click.stop>
      <div class="dialog-title">{{ confirmDialogTitle }}</div>
      <div class="dialog-message">{{ confirmDialogMessage }}</div>
      <div class="dialog-buttons">
        <button class="dialog-btn cancel" @click="cancelConfirmDialog">取消</button>
        <button class="dialog-btn confirm" @click="doConfirmDialog">确定</button>
      </div>
    </div>
  </div>

  <!-- ====== 编辑/添加消息对话框 ====== -->
  <div class="custom-overlay center" v-if="editDialogShow" @click="cancelEditDialog">
    <div class="custom-dialog" @click.stop style="max-width: 360px;">
      <div class="dialog-title">{{ editDialogTitle }}</div>
      <!-- 角色选择 -->
      <div class="role-selector">
        <div 
          class="role-option" 
          :class="{ active: editDialogRole === 'user' }"
          @click="editDialogRole = 'user'"
        >用户</div>
        <div 
          class="role-option" 
          :class="{ active: editDialogRole === 'ai' }"
          @click="editDialogRole = 'ai'"
        >AI</div>
        <div 
          class="role-option" 
          :class="{ active: editDialogRole === 'system' }"
          @click="editDialogRole = 'system'"
        >系统</div>
      </div>
      <textarea 
        ref="editTextareaRef"
        class="dialog-textarea" 
        v-model="editDialogContent" 
        placeholder="请输入消息内容..."
      ></textarea>
      <div class="dialog-buttons">
        <button class="dialog-btn cancel" @click="cancelEditDialog">取消</button>
        <button class="dialog-btn confirm" @click="confirmEditDialog">确定</button>
      </div>
    </div>
  </div>
</div>

<script>
const { createApp, ref, onMounted, onUnmounted, computed, nextTick, watch } = Vue;

function generateUUID() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}

createApp({
  setup() {
    const showToast = vant.showToast;

    const API_BASE = 'http://localhost:1234';
    const apiFetch = async (path, options = {}) => {
      const url = path.startsWith('http') ? path : (API_BASE + path);
      const res = await fetch(url, {
        headers: { 'Content-Type': 'application/json', ...(options.headers || {}) },
        ...options
      });
      const text = await res.text();
      try { return JSON.parse(text); } catch { return { status: 'error', message: text, httpStatus: res.status }; }
    };

    // ====== Refs ======
    const inputRef = ref(null);
    const bottomBar = ref(null);
    const chatContainer = ref(null);
    const dialogInputRef = ref(null);
    const scrollAnchor = ref(null);
    const editTextareaRef = ref(null);

    // ====== 中断控制器 ======
    let abortController = null;

    // ====== HTML转义函数 ======
    const escapeHtml = (text) => {
      if (!text) return '';
      const map = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#039;'
      };
      return String(text).replace(/[&<>"']/g, m => map[m]);
    };

    // ====== 从HTML提取纯文本（用于复制） ======
    const extractTextFromHtml = (html) => {
      if (!html) return '';
      const temp = document.createElement('div');
      temp.innerHTML = html;
      
      // 处理 details 元素，提取其中的内容
      const details = temp.querySelectorAll('details');
      details.forEach(d => {
        const summary = d.querySelector('summary');
        const pre = d.querySelector('pre');
        const summaryText = summary ? summary.textContent : '';
        const preText = pre ? pre.textContent : '';
        // 替换 details 为纯文本格式
        const textNode = document.createTextNode(`\n[${summaryText}]\n${preText}\n`);
        d.parentNode.replaceChild(textNode, d);
      });
      
      // 处理 pre 元素
      const pres = temp.querySelectorAll('pre');
      pres.forEach(p => {
        const textNode = document.createTextNode(`\n${p.textContent}\n`);
        p.parentNode.replaceChild(textNode, p);
      });
      
      return temp.textContent || temp.innerText || '';
    };

    // ====== 获取消息的纯文本内容（用于复制） ======
    const getMessagePlainText = (msg) => {
      if (msg.isHtml) {
        return extractTextFromHtml(msg.content);
      }
      return msg.content || '';
    };

    // ====== 安全的消息内容显示 ======
    const getDisplayContent = (msg) => {
      if (msg.isHtml) {
        return msg.content;
      }
      
      const rawContent = msg.content || '';
      
      if (msg.role === 'user') {
        return escapeHtml(rawContent);
      }
      
      if (msg.role === 'ai') {
        return formatAIResponseSafe(rawContent);
      }
      
      return escapeHtml(rawContent);
    };

    // ====== 安全的AI响应格式化 ======
    const formatAIResponseSafe = (rawText) => {
      // 匹配新的工具调用格式
      const toolCallRegex = /<<<AITOOL_CALL>>>\s*([\s\S]*?)\s*<<<AITOOL_END>>>/g;
      let result = '';
      let lastIndex = 0;
      let match;
      
      const text = rawText || '';
      
      while ((match = toolCallRegex.exec(text)) !== null) {
        // 先处理匹配位置之前的文本
        const beforeText = text.slice(lastIndex, match.index);
        result += escapeHtml(beforeText);
        
        // 解析工具调用内容
        const jsonStr = match[1].trim();
        let toolName = "工具调用";
        let displayJson = jsonStr;
        
        try {
          const parsed = JSON.parse(jsonStr);
          toolName = parsed.action || "工具调用";
          // 美化 JSON 显示
          displayJson = JSON.stringify(parsed, null, 2);
        } catch {
          // 解析失败，使用原始内容
        }
        
        // 生成折叠的工具调用显示
        result += `<details class="tool-request"><summary>📡 工具调用 [${escapeHtml(toolName)}]</summary><pre>${escapeHtml(displayJson)}</pre></details>`;
        
        lastIndex = match.index + match[0].length;
      }
      
      // 处理剩余文本
      result += escapeHtml(text.slice(lastIndex));
      
      return result;
    };
    // ====== 键盘状态 ======
    const keyboardOpen = ref(false);
    const keyboardHeight = ref(0);
    let stableHeight = 0;
    let stableCheckTimer = null;
    let lastVVHeight = 0;
    let stableCount = 0;
    
    const ensureStableHeight = () => {
      if (!window.visualViewport) {
        stableHeight = window.innerHeight;
        return stableHeight;
      }
      const currentH = window.visualViewport.height;
      if (stableHeight === 0 || currentH > stableHeight + 50) {
        stableHeight = currentH;
      }
      return stableHeight;
    };
    
    const scheduleStableCheck = () => {
      if (stableCheckTimer) clearTimeout(stableCheckTimer);
      stableCheckTimer = setTimeout(() => {
        if (window.visualViewport) {
          const h = window.visualViewport.height;
          if (Math.abs(h - lastVVHeight) < 10) {
            stableCount++;
            if (stableCount >= 3 && h > stableHeight) {
              stableHeight = h;
            }
          } else {
            stableCount = 0;
          }
          lastVVHeight = h;
        }
      }, 500);
    };
    
    const bottomBarStyle = computed(() => {
      if (keyboardOpen.value) {
        return {
          position: 'fixed',
          left: '0',
          right: '0',
          bottom: '0',
          transform: 'none',
          zIndex: '9999',
          backgroundColor: '#fff',
          borderTop: '1px solid #eee',
          padding: '10px 12px',
          paddingBottom: 'calc(10px + env(safe-area-inset-bottom, 0))'
        };
      }
      return {};
    });
    
    const chatContainerStyle = computed(() => {
      if (keyboardOpen.value && keyboardHeight.value > 0) {
        const barH = bottomBar.value ? bottomBar.value.offsetHeight : 60;
        return { paddingBottom: (keyboardHeight.value + barH + 20) + 'px' };
      }
      return { paddingBottom: '20px' };
    });
    
    const updateKeyboardState = (doScroll = false) => {
      if (!window.visualViewport) return;
      const currentH = window.visualViewport.height;
      ensureStableHeight();
      const kbH = Math.max(0, stableHeight - currentH);
      
      if (kbH > 100) {
        keyboardOpen.value = true;
        keyboardHeight.value = kbH;
        if (doScroll) {
          nextTick(() => {
            setTimeout(() => {
              if (scrollAnchor.value) {
                scrollAnchor.value.scrollIntoView({ behavior: 'smooth', block: 'end' });
              }
            }, 150);
          });
        }
      } else {
        keyboardOpen.value = false;
        keyboardHeight.value = 0;
        if (currentH > stableHeight) {
          stableHeight = currentH;
        }
      }
    };
    
    const initKeyboardHandler = () => {
      if (window.visualViewport) {
        setTimeout(() => {
          stableHeight = window.visualViewport.height;
          lastVVHeight = stableHeight;
        }, 800);
        setTimeout(scheduleStableCheck, 1500);
        setTimeout(scheduleStableCheck, 3000);
        window.visualViewport.addEventListener('resize', () => {
          updateKeyboardState(false);
          scheduleStableCheck();
        });
      }
      window.addEventListener('resize', () => updateKeyboardState(false));
    };
    
    const onInputFocus = () => {
      setTimeout(() => updateKeyboardState(true), 100);
      setTimeout(() => updateKeyboardState(true), 300);
      setTimeout(() => updateKeyboardState(true), 500);
    };
    
    const onInputBlur = () => {
      setTimeout(() => updateKeyboardState(false), 150);
    };

    // ====== UI 状态 ======
    const showSidebar = ref(false);
    const sidebarLoading = ref(false);
    const activeGroups = ref([]);

    // ====== 历史记录开关 ======
    const includeHistory = ref(localStorage.getItem('chat_include_history') === 'true');
    
    const onHistoryToggleChange = (val) => {
      localStorage.setItem('chat_include_history', val ? 'true' : 'false');
      showToast(val ? '已开启历史记录（合并发送）' : '已关闭历史记录');
    };

    // ====== 会话/分组数据 ======
    const groups = ref([]);
    const sessions = ref([]);
    const currentSessionId = ref(localStorage.getItem('chat_session_id') || '');

    // ====== 自定义 ActionSheet 状态 ======
    const actionSheetShow = ref(false);
    const actionSheetTitle = ref('');
    const actionSheetItems = ref([]);
    let currentActionChat = null;
    let currentActionGroup = null;
    let actionSheetType = '';
    let selectedProvider = null;

    // ====== 自定义输入对话框状态 ======
    const inputDialogShow = ref(false);
    const inputDialogTitle = ref('');
    const inputDialogValue = ref('');
    const inputDialogPlaceholder = ref('请输入...');
    let inputDialogResolve = null;

    // ====== 自定义确认对话框状态 ======
    const confirmDialogShow = ref(false);
    const confirmDialogTitle = ref('');
    const confirmDialogMessage = ref('');
    let confirmDialogResolve = null;

    // ====== 编辑/添加消息对话框状态 ======
    const editDialogShow = ref(false);
    const editDialogTitle = ref('');
    const editDialogContent = ref('');
    const editDialogRole = ref('user');
    let editDialogMode = 'edit';
    let editDialogIndex = -1;
    let editDialogResolve = null;
    
    // 当前模型名称
    const currentModelName = ref('AFile');
    
    // 加载当前AI配置
    const loadCurrentModel = async () => {
        try {
            const res = await apiFetch('/api/ai-config', { method: 'GET' });
            if (res.status === 'success' && res.data) {
                const model = res.data.currentModel;
                if (model) {
                    currentModelName.value = model;
                }
            }
        } catch (e) {
            console.error('加载模型配置失败:', e);
        }
    };
    
    // ====== 消息 ======
    const messages = ref([]);
    const userInput = ref('');
    const loading = ref(false);

    // ====== Computed ======
    const orderedGroups = computed(() => {
      const arr = [...groups.value];
      arr.sort((a, b) => (a.order ?? 0) - (b.order ?? 0));
      return arr;
    });

    const currentSessionTitle = computed(() => {
      const s = sessions.value.find(x => x.id === currentSessionId.value);
      return s ? (s.title || '未命名会话') : '未选择';
    });

    // ====== 自定义 ActionSheet 方法 ======
    const showActionSheet = (title, items, type) => {
      actionSheetTitle.value = title;
      actionSheetItems.value = items;
      actionSheetType = type;
      actionSheetShow.value = true;
    };

    const closeActionSheet = () => {
      actionSheetShow.value = false;
      actionSheetTitle.value = '';
      actionSheetItems.value = [];
    };

    const onActionSelect = async (action) => {
      closeActionSheet();
      await nextTick();
      await new Promise(r => setTimeout(r, 100));

      if (actionSheetType === 'session') {
        await handleSessionAction(action);
      } else if (actionSheetType === 'group') {
        await handleGroupAction(action);
      } else if (actionSheetType === 'moveGroup') {
        await handleMoveGroupAction(action);
      } else if (actionSheetType === 'provider') {
        await handleProviderAction(action);
      } else if (actionSheetType === 'model') {
        await handleModelAction(action);
      }
    };

    // ====== 自定义输入对话框方法 ======
    const showInputDialogAsync = (title, defaultValue = '', placeholder = '请输入...') => {
      return new Promise((resolve) => {
        inputDialogTitle.value = title;
        inputDialogValue.value = defaultValue;
        inputDialogPlaceholder.value = placeholder;
        inputDialogResolve = resolve;
        inputDialogShow.value = true;
        nextTick(() => {
          setTimeout(() => {
            if (dialogInputRef.value) dialogInputRef.value.focus();
          }, 100);
        });
      });
    };

    const cancelInputDialog = () => {
      inputDialogShow.value = false;
      if (inputDialogResolve) {
        inputDialogResolve(null);
        inputDialogResolve = null;
      }
      inputDialogValue.value = '';
    };

    const confirmInputDialog = () => {
      const value = inputDialogValue.value.trim();
      inputDialogShow.value = false;
      if (inputDialogResolve) {
        inputDialogResolve(value || null);
        inputDialogResolve = null;
      }
      inputDialogValue.value = '';
    };

    // ====== 自定义确认对话框方法 ======
    const showConfirmDialogAsync = (title, message) => {
      return new Promise((resolve) => {
        confirmDialogTitle.value = title;
        confirmDialogMessage.value = message;
        confirmDialogResolve = resolve;
        confirmDialogShow.value = true;
      });
    };

    const cancelConfirmDialog = () => {
      confirmDialogShow.value = false;
      if (confirmDialogResolve) {
        confirmDialogResolve(false);
        confirmDialogResolve = null;
      }
    };

    const doConfirmDialog = () => {
      confirmDialogShow.value = false;
      if (confirmDialogResolve) {
        confirmDialogResolve(true);
        confirmDialogResolve = null;
      }
    };
    
    // ====== 编辑/添加消息对话框方法 ======
    const showEditDialogAsync = (title, content = '', role = 'user', mode = 'edit', index = -1) => {
      return new Promise((resolve) => {
        editDialogTitle.value = title;
        editDialogContent.value = content;
        editDialogRole.value = role;
        editDialogMode = mode;
        editDialogIndex = index;
        editDialogResolve = resolve;
        editDialogShow.value = true;
        nextTick(() => {
          setTimeout(() => {
            if (editTextareaRef.value) editTextareaRef.value.focus();
          }, 100);
        });
      });
    };

    const cancelEditDialog = () => {
      editDialogShow.value = false;
      if (editDialogResolve) {
        editDialogResolve(null);
        editDialogResolve = null;
      }
      editDialogContent.value = '';
    };

    const confirmEditDialog = () => {
      const content = editDialogContent.value.trim();
      if (!content) {
        showToast('内容不能为空');
        return;
      }
      editDialogShow.value = false;
      if (editDialogResolve) {
        editDialogResolve({
          content: content,
          role: editDialogRole.value
        });
        editDialogResolve = null;
      }
      editDialogContent.value = '';
    };
    
    // ====== 删除单条消息 ======
    const deleteMessage = async (index) => {
      const msg = messages.value[index];
      if (msg.pinned) {
        showToast('已钉住的消息不能删除');
        return;
      }
      const confirmed = await showConfirmDialogAsync('删除消息', '确定要删除这条消息吗？');
      if (confirmed) {
        messages.value.splice(index, 1);
        await syncSaveMessages();
        showToast('已删除');
      }
    };

    // ====== 编辑消息 ======
    const editMessage = async (index) => {
      const msg = messages.value[index];
      // 获取原始内容
      const originalContent = msg.isHtml ? extractTextFromHtml(msg.content) : (msg.content || '');
      
      const result = await showEditDialogAsync(
        '编辑消息', 
        originalContent, 
        msg.role, 
        'edit', 
        index
      );
      
      if (result) {
        messages.value[index] = {
          ...msg,
          content: result.content,
          role: result.role,
          isHtml: false
        };
        await syncSaveMessages();
        showToast('已保存');
      }
    };

    // ====== 钉住/取消钉住消息 ======
    const togglePin = async (index) => {
      const msg = messages.value[index];
      msg.pinned = !msg.pinned;
      await syncSaveMessages();
      showToast(msg.pinned ? '已钉住' : '已取消钉住');
    };

    // ====== 在下方添加消息 ======
    const addMessageBelow = async (index) => {
      const result = await showEditDialogAsync(
        '添加消息', 
        '', 
        'user', 
        'add', 
        index
      );
      
      if (result) {
        const newMsg = {
          role: result.role,
          content: result.content,
          pinned: false
        };
        messages.value.splice(index + 1, 0, newMsg);
        await syncSaveMessages();
        showToast('已添加');
        scrollToBottom();
      }
    };

    // ====== 复制所有历史到剪切板（包含完整内容） ======
    const copyAllHistory = async () => {
      if (messages.value.length === 0) {
        showToast('没有历史记录');
        return;
      }
      
      let historyStr = '';
      messages.value.forEach((m, index) => {
        const roleLabel = m.role === 'user' ? '[用户]' : (m.role === 'ai' ? '[AI]' : '[系统]');
        const content = getMessagePlainText(m);
        historyStr += `${roleLabel}:\n${content}\n`;
        if (index < messages.value.length - 1) {
          historyStr += '\n---\n\n';
        }
      });
      
      try {
        await navigator.clipboard.writeText(historyStr.trim());
        showToast('已复制到剪切板');
      } catch (err) {
        // 回退方案
        const textarea = document.createElement('textarea');
        textarea.value = historyStr.trim();
        textarea.style.position = 'fixed';
        textarea.style.left = '-9999px';
        document.body.appendChild(textarea);
        textarea.select();
        try {
          document.execCommand('copy');
          showToast('已复制到剪切板');
        } catch (e) {
          showToast('复制失败');
        }
        document.body.removeChild(textarea);
      }
    };

    // ====== 清理未钉住的消息 ======
    const clearUnpinnedMessages = async () => {
      const unpinnedCount = messages.value.filter(m => !m.pinned).length;
      if (unpinnedCount === 0) {
        showToast('没有未钉住的消息');
        return;
      }
      
      const confirmed = await showConfirmDialogAsync(
        '清理未钉住消息', 
        `将清理 ${unpinnedCount} 条未钉住的消息，已钉住的消息会保留。确定继续吗？`
      );
      
      if (confirmed) {
        messages.value = messages.value.filter(m => m.pinned);
        await syncSaveMessages();
        showToast(`已清理 ${unpinnedCount} 条消息`);
      }
    };

    // ====== 清空所有消息 ======
    const clearAllMessages = async () => {
      if (messages.value.length === 0) {
        showToast('没有消息可清空');
        return;
      }
      const pinnedCount = messages.value.filter(m => m.pinned).length;
      let message = '确定要清空当前会话的所有消息吗？此操作不可恢复。';
      if (pinnedCount > 0) {
        message = `当前有 ${pinnedCount} 条已钉住的消息也会被清空。确定继续吗？`;
      }
      
      const confirmed = await showConfirmDialogAsync('清空消息', message);
      if (confirmed) {
        messages.value = [{ role: 'ai', content: '您好！我是您的AI助手，有什么可以帮您？' }];
        await syncSaveMessages();
        showToast('已清空');
      }
    };
    
    // ====== 方法 ======
    const getGroupSessions = (groupId) => {
      const list = sessions.value.filter(s => (s.groupId || 'default') === groupId);
      return list.sort((a, b) => {
        const at = a.isTop ? 1 : 0;
        const bt = b.isTop ? 1 : 0;
        if (at !== bt) return bt - at;
        return (b.updatedAt || '').localeCompare(a.updatedAt || '');
      });
    };

    const toggleGroup = (groupId) => {
      const idx = activeGroups.value.indexOf(groupId);
      if (idx > -1) {
        activeGroups.value.splice(idx, 1);
      } else {
        activeGroups.value.push(groupId);
      }
    };

    // ====== 侧边栏 ======
    const openSidebar = () => { showSidebar.value = true; };
    const onSidebarOpen = () => { document.body.classList.add('sidebar-open'); };
    const onSidebarClose = () => { document.body.classList.remove('sidebar-open'); };

    // ====== 输入框 ======
    const autoResize = () => {
      const el = inputRef.value;
      if (el) {
        el.style.height = 'auto';
        el.style.height = Math.min(el.scrollHeight, 100) + 'px';
      }
    };
    
    // ====== 滚动 ======
    const scrollToBottom = () => {
      nextTick(() => {
        if (chatContainer.value) {
          chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
        }
      });
    };

    // ====== 本地存储 ======
    const localKey = (sid) => `chat_messages_${sid}`;
    const loadMessagesLocal = (sid) => {
      const saved = localStorage.getItem(localKey(sid));
      if (saved) { try { return JSON.parse(saved); } catch {} }
      return [{ role: 'ai', content: '您好！我是您的AI助手，有什么可以帮您？' }];
    };
    const saveMessagesLocal = (sid, msgs) => {
      try { localStorage.setItem(localKey(sid), JSON.stringify(msgs)); } catch {}
    };

    const loadMessagesBackend = async (sid) => {
      const res = await apiFetch(`/api/sessions/${sid}/messages`, { method: 'GET' });
      if (res.status === 'success' && res.messages?.length) return res.messages;
      return null;
    };

    const saveMessagesBackend = async (sid, msgs) => {
      const res = await apiFetch(`/api/sessions/${sid}/messages`, {
        method: 'POST',
        body: JSON.stringify({ messages: msgs })
      });
      return res.status === 'success';
    };

    const syncSaveMessages = async () => {
      const sid = currentSessionId.value;
      if (!sid) return;
      saveMessagesLocal(sid, messages.value);
      await saveMessagesBackend(sid, messages.value).catch(() => {});
    };

    // ====== 会话刷新 ======
    const refreshSessions = async () => {
      sidebarLoading.value = true;
      try {
        const res = await apiFetch('/api/sessions', { method: 'GET' });
        if (res.status === 'success') {
          groups.value = res.data?.groups || [];
          sessions.value = res.data?.sessions || [];
          if (!activeGroups.value.length) {
            activeGroups.value = groups.value.map(g => g.id);
          }
          if (currentSessionId.value && !sessions.value.find(s => s.id === currentSessionId.value)) {
            currentSessionId.value = '';
          }
          if (!currentSessionId.value) {
            if (sessions.value.length) {
              await selectChat(sessions.value[0], { silent: true });
            } else {
              await newChatInGroup('default', { silent: true });
            }
          } else {
            const backendMsgs = await loadMessagesBackend(currentSessionId.value).catch(() => null);
            messages.value = backendMsgs?.length ? backendMsgs : loadMessagesLocal(currentSessionId.value);
          }
        } else {
          if (!currentSessionId.value) {
            currentSessionId.value = 'local_' + generateUUID();
            localStorage.setItem('chat_session_id', currentSessionId.value);
          }
          messages.value = loadMessagesLocal(currentSessionId.value);
        }
      } catch (e) {
        console.error('refreshSessions error:', e);
        if (!currentSessionId.value) {
          currentSessionId.value = 'local_' + generateUUID();
          localStorage.setItem('chat_session_id', currentSessionId.value);
        }
        messages.value = loadMessagesLocal(currentSessionId.value);
      } finally {
        sidebarLoading.value = false;
      }
    };

    // ====== 会话操作 ======
    const newChatInGroup = async (groupId, opts = {}) => {
      const { silent = false } = opts;
      showSidebar.value = false;
      const res = await apiFetch('/api/sessions', {
        method: 'POST',
        body: JSON.stringify({ title: '', groupId: groupId || 'default' })
      });
      if (res.status !== 'success') {
        const newId = 'local_' + generateUUID();
        currentSessionId.value = newId;
        localStorage.setItem('chat_session_id', newId);
        messages.value = [{ role: 'ai', content: '您好！我是您的AI助手，有什么可以帮您？' }];
        if (!silent) showToast('新会话已创建(本地)');
        return;
      }
      await refreshSessions();
      if (res.session) await selectChat(res.session, { silent: true });
      if (!silent) showToast('新会话已创建');
    };

    const selectChat = async (chat, opts = {}) => {
      const { silent = false } = opts;
      currentSessionId.value = chat.id;
      localStorage.setItem('chat_session_id', chat.id);

      const backendMsgs = await loadMessagesBackend(chat.id).catch(() => null);
      messages.value = backendMsgs?.length ? backendMsgs : loadMessagesLocal(chat.id);
      await saveMessagesBackend(chat.id, messages.value).catch(() => {});

      showSidebar.value = false;
      if (!silent) showToast(`已切换：${chat.title || '未命名会话'}`);
      scrollToBottom();
    };

    const openSessionMenu = (chat) => {
      currentActionChat = chat;
      showActionSheet('', [
        { name: chat.isTop ? '取消置顶' : '置顶', key: 'top' },
        { name: '重命名', key: 'rename' },
        { name: '移动分组', key: 'move' },
        { name: '删除', key: 'delete', color: '#ee0a24' }
      ], 'session');
    };

    const handleSessionAction = async (action) => {
      const chat = currentActionChat;
      if (!chat) return;

      if (action.key === 'top') {
        const res = await apiFetch(`/api/sessions/${chat.id}`, {
          method: 'PUT', body: JSON.stringify({ isTop: !chat.isTop })
        });
        if (res.status === 'success') { 
          await refreshSessions(); 
          showToast(chat.isTop ? '已取消置顶' : '已置顶'); 
        } else {
          showToast(res.message || '操作失败');
        }
      }
      
      if (action.key === 'rename') {
        const newName = await showInputDialogAsync('重命名会话', chat.title || '', '请输入会话名称');
        if (newName) {
          const res = await apiFetch(`/api/sessions/${chat.id}`, {
            method: 'PUT', body: JSON.stringify({ title: newName })
          });
          if (res.status === 'success') { 
            await refreshSessions(); 
            showToast('已修改'); 
          } else {
            showToast(res.message || '修改失败');
          }
        }
      }
      
      if (action.key === 'move') {
        showActionSheet('选择目标分组', 
          groups.value.map(g => ({ name: g.name, groupId: g.id })), 
          'moveGroup'
        );
      }
      
      if (action.key === 'delete') {
        const confirmed = await showConfirmDialogAsync('确认删除', '删除后无法恢复对话内容');
        if (confirmed) {
          const res = await apiFetch(`/api/sessions/${chat.id}`, { method: 'DELETE' });
          if (res.status === 'success') { 
            await refreshSessions(); 
            showToast('已删除'); 
          } else {
            showToast(res.message || '删除失败');
          }
        }
      }
    };

    const handleMoveGroupAction = async (action) => {
      const chat = currentActionChat;
      if (!chat) return;
      
      const res = await apiFetch(`/api/sessions/${chat.id}`, {
        method: 'PUT', body: JSON.stringify({ groupId: action.groupId })
      });
      if (res.status === 'success') { 
        await refreshSessions(); 
        showToast('已移动'); 
      } else {
        showToast(res.message || '移动失败');
      }
    };

    // ====== 分组操作 ======
    const handleAddGroup = async () => {
      const name = await showInputDialogAsync('新建分组', '', '请输入分组名称');
      if (name) {
        const res = await apiFetch('/api/groups', { method: 'POST', body: JSON.stringify({ name }) });
        if (res.status === 'success') { 
          await refreshSessions(); 
          showToast('分组已创建'); 
        } else {
          showToast(res.message || '创建失败');
        }
      }
    };

    const openGroupMenu = (group) => {
      currentActionGroup = group;
      const actions = [{ name: '重命名', key: 'rename' }];
      if (group.id !== 'default' && group.id !== 'ungrouped') {
        actions.push({ name: '删除分组', key: 'delete', color: '#ee0a24' });
      }
      showActionSheet('', actions, 'group');
    };

    const handleGroupAction = async (action) => {
      const group = currentActionGroup;
      if (!group) return;

      if (action.key === 'rename') {
        const newName = await showInputDialogAsync('重命名分组', group.name || '', '请输入分组名称');
        if (newName) {
          const res = await apiFetch(`/api/groups/${group.id}`, {
            method: 'PUT', body: JSON.stringify({ name: newName })
          });
          if (res.status === 'success') { 
            await refreshSessions(); 
            showToast('已修改'); 
          } else {
            showToast(res.message || '修改失败');
          }
        }
      }
      
      if (action.key === 'delete') {
        const confirmed = await showConfirmDialogAsync('删除分组', '删除分组后，该分组内会话将移动到"未分组"。确定继续吗？');
        if (confirmed) {
          const res = await apiFetch(`/api/groups/${group.id}`, { method: 'DELETE' });
          if (res.status === 'success') { 
            await refreshSessions(); 
            showToast('分组已删除'); 
          } else {
            showToast(res.message || '删除失败');
          }
        }
      }
    };

    // ====== 快速切换模型 ======
    const openQuickSwitch = async () => {
      const res = await apiFetch('/api/ai-config', { method: 'GET' });
      if (res.status !== 'success') { 
        showToast(res.message || '读取AI配置失败'); 
        return; 
      }
      const providers = (res.data?.providers || []).filter(p => p.enabled !== false);
      if (!providers.length) { 
        showToast('没有可用接口，请先到设置里添加'); 
        return; 
      }
      showActionSheet('选择接口', 
        providers.map(p => ({ name: p.name, provider: p })), 
        'provider'
      );
    };

    const handleProviderAction = (action) => {
      selectedProvider = action.provider;
      const models = (action.provider.models || []).map(m => ({ name: m, model: m }));
      if (!models.length) { 
        showToast('该接口未配置模型'); 
        return; 
      }
      setTimeout(() => {
        showActionSheet('选择模型', models, 'model');
      }, 150);
    };

    const handleModelAction = async (action) => {
        if (!selectedProvider) return;
        const res = await apiFetch('/api/ai-config/switch', {
            method: 'POST', 
            body: JSON.stringify({ providerId: selectedProvider.id, model: action.model })
        });
        if (res.status === 'success') {
            currentModelName.value = action.model; // 更新标题
            showToast(`已切换：${selectedProvider.name} / ${action.model}`);
        } else {
            showToast(res.message || '切换失败');
        }
    };

    // ====== 总结历史 ======
    async function summarizeHistory() {
      const confirmed = await showConfirmDialogAsync('总结历史', '这将总结早期的对话历史，保留最后一段完整对话。确定要继续吗？');
      if (!confirmed) return;
      
      loading.value = true;
      abortController = new AbortController();
      
      try {
        const response = await fetch(API_BASE + '/api/summarize-history', {
          method: 'POST', 
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ session_id: currentSessionId.value }),
          signal: abortController.signal
        });
        const result = await response.json();
        showToast(result.status === 'success' ? '总结成功' : '总结失败');
      } catch (error) {
        if (error.name === 'AbortError') {
          showToast('已停止');
        } else {
          showToast('请求失败: ' + error.message);
        }
      } finally { 
        loading.value = false;
        abortController = null;
      }
    }

    // ====== 构建历史记录字符串 ======
    const buildHistoryString = () => {
      if (messages.value.length === 0) return '';
      
      // 使用更清晰的格式
      let historyStr = '【以下是对话历史，请基于这些上下文回答】\n\n';
      
      messages.value.forEach((m, index) => {
        const content = m.isHtml ? extractTextFromHtml(m.content) : (m.content || '');
        if (m.role === 'user') {
          historyStr += `Human: ${content}\n\n`;
        } else if (m.role === 'ai') {
          historyStr += `Assistant: ${content}\n\n`;
        }
      });
      
      historyStr += '---\n\n';
      return historyStr;
    };

    // ====== 停止生成 ======
    const stopGeneration = () => {
      if (abortController) {
        abortController.abort();
        abortController = null;
      }
      loading.value = false;
      showToast('已停止生成');
    };

    // ====== 发送按钮点击处理 ======
    const handleSendClick = () => {
      if (loading.value) {
        stopGeneration();
      } else {
        onSendEnhanced();
      }
    };
    const executeToolFromAI = async (aiResponse, statusMsg, signal) => {
      try {
        // ✅ 使用新的标记格式匹配工具调用
        // 匹配 <<<AITOOL_CALL>>> ... <<<AITOOL_END>>>
        const toolCallRegex = /<<<AITOOL_CALL>>>\s*([\s\S]*?)\s*<<<AITOOL_END>>>/g;
        const toolCalls = [];
        let match;
        
        while ((match = toolCallRegex.exec(aiResponse)) !== null) {
          const jsonStr = match[1].trim();
          try {
            // 直接解析 JSON，所有转义由 JSON.parse 自动处理
            const toolCall = JSON.parse(jsonStr);
            if (toolCall && toolCall.action) {
              toolCalls.push(toolCall);
            }
          } catch (parseError) {
            console.error('工具调用 JSON 解析失败:', parseError);
            console.error('原始内容:', jsonStr);
            // 记录解析失败的调用，但继续处理其他调用
            toolCalls.push({
              action: '_parse_error',
              error: parseError.message,
              raw: jsonStr.substring(0, 200)
            });
          }
        }
        
        if (!toolCalls.length) { 
          return []; 
        }
    
        statusMsg.content = `<span class="exec-status running">⏳ 正在执行 ${toolCalls.length} 个工具...</span>`;
        statusMsg.isHtml = true;
        scrollToBottom();
    
        // 工具 action 到 API 端点的映射
        const apiMap = {
          // 文件操作
          'file_read':        { url: '/api/file-tools/read',        method: 'POST', paramMap: { filepath: 'filepath', lines: 'lines' } },
          'file_write':       { url: '/api/file-tools/write',       method: 'POST', paramMap: { filepath: 'filepath', content: 'content', mode: 'mode' } },
          'file_replace':     { url: '/api/file-tools/replace',     method: 'POST', paramMap: { filepath: 'filepath', old: 'old', new: 'new', regex: 'regex', dry_run: 'dry_run' } },
          'file_patch':       { url: '/api/file/patch',             method: 'POST', paramMap: { filepath: 'filepath', anchor: 'anchor', code: 'code', mode: 'mode' } },
          'file_list':        { url: '/api/file-tools/list',        method: 'POST', paramMap: { directory: 'directory' } },
          'file_info':        { url: '/api/file-tools/info',        method: 'POST', paramMap: { filepath: 'filepath' } },
          'file_find':        { url: '/api/file-tools/find-files',  method: 'POST', paramMap: { pattern: 'pattern', content: 'content', max_results: 'max_results', directory: 'directory' } },
          'file_find_in':     { url: '/api/file-tools/find-in-file',method: 'POST', paramMap: { filepath: 'filepath', pattern: 'pattern' } },
          'file_delete':      { url: '/api/file-tools/delete',      method: 'POST', paramMap: { filepath: 'filepath' } },
          'file_copy':        { url: '/api/file-tools/copy',        method: 'POST', paramMap: { source: 'source', destination: 'destination' } },
          'file_move':        { url: '/api/file-tools/move',        method: 'POST', paramMap: { source: 'source', destination: 'destination' } },
          'file_mkdir':       { url: '/api/file-tools/mkdir',       method: 'POST', paramMap: { directory: 'directory' } },
          'file_analyze':     { url: '/api/file-tools/analyze',     method: 'POST', paramMap: { filepath: 'filepath' } },
          'file_analyze_dir': { url: '/api/file-tools/analyze-dir', method: 'POST', paramMap: { directory: 'directory' } },
          'file_undo':        { url: '/api/file-tools/undo',        method: 'POST', paramMap: {} },
          'file_history':     { url: '/api/file-tools/history',     method: 'GET',  paramMap: { limit: 'limit' } },
          'file_replace_all': { url: '/api/file-tools/replace-all', method: 'POST', paramMap: { pattern: 'pattern', old: 'old', new: 'new', dry_run: 'dry_run' } },
          // 脚本运行
          'script_run':       { url: '/api/script/run',             method: 'POST', paramMap: { script: 'script' } },
          'script_run_file':  { url: '/api/script/run-file',        method: 'POST', paramMap: { path: 'path', args: 'args', type: 'type' } },
        };
    
        // 执行单个工具调用
        const executeOne = async (tool) => {
          const action = tool.action;
          
          // 检查是否是解析错误
          if (action === '_parse_error') {
            return { 
              action: 'JSON解析错误', 
              success: false, 
              error: tool.error,
              raw: tool.raw
            };
          }
          
          const api = apiMap[action];
          if (!api) {
            return { action, success: false, error: `未知的工具: ${action}` };
          }
          
          // 构建请求参数（根据 paramMap 映射参数名）
          const params = {};
          for (const [toolParam, apiParam] of Object.entries(api.paramMap)) {
            if (tool[toolParam] !== undefined) {
              params[apiParam] = tool[toolParam];
            }
          }
          
          try {
            let response;
            
            if (api.method === 'GET') {
              // GET 请求，参数放在 URL 中
              const queryParams = new URLSearchParams();
              for (const [key, value] of Object.entries(params)) {
                if (value !== undefined && value !== null) {
                  queryParams.append(key, value);
                }
              }
              const url = API_BASE + api.url + (queryParams.toString() ? '?' + queryParams.toString() : '');
              response = await fetch(url, { method: 'GET', signal });
            } else {
              // POST 请求
              response = await fetch(API_BASE + api.url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(params),
                signal
              });
            }
            
            const textRes = await response.text();
            let data;
            try {
              data = JSON.parse(textRes);
            } catch {
              data = { rawText: textRes };
            }
            
            // 判断是否成功
            const isSuccess = data.success !== false && data.status !== 'error';
            
            return { action, success: isSuccess, data };
          } catch (err) {
            if (err.name === 'AbortError') throw err;
            return { action, success: false, error: err.message };
          }
        };
    
        // 并行执行所有工具调用
        const results = await Promise.all(toolCalls.map(call => executeOne(call)));
        
        // 生成结果 HTML
        const resultHtml = results.map(r => {
          const displayName = r.action.length > 30 ? r.action.substring(0, 30) + '...' : r.action;
          
          if (r.success) {
            const safeData = escapeHtml(JSON.stringify(r.data, null, 2));
            return `<details class="tool-result success"><summary>✅ ${escapeHtml(displayName)}</summary><pre>${safeData}</pre></details>`;
          } else {
            const errorInfo = r.error || (r.data ? JSON.stringify(r.data, null, 2) : '未知错误');
            let content = escapeHtml(errorInfo);
            if (r.raw) {
              content += '\n\n原始内容:\n' + escapeHtml(r.raw);
            }
            return `<details class="tool-result error"><summary>❌ ${escapeHtml(displayName)}</summary><pre>${content}</pre></details>`;
          }
        }).join('');
        
        statusMsg.content = resultHtml;
        statusMsg.isHtml = true;
        return results;
        
      } catch (error) {
        if (error.name === 'AbortError') throw error;
        statusMsg.content = `<span class="exec-status running">❌ 执行错误: ${escapeHtml(error.message || '未知错误')}</span>`;
        statusMsg.isHtml = true;
        return [];
      }
    };

    const onSendEnhanced = async () => {
        if (!userInput.value.trim() || loading.value) return;
        const text = userInput.value.trim();
        
        const historyStr = includeHistory.value ? buildHistoryString() : '';
        
        messages.value.push({ role: 'user', content: text, pinned: false });
        userInput.value = '';
        loading.value = true;
        
        abortController = new AbortController();
        const signal = abortController.signal;
        
        if (inputRef.value) {
            inputRef.value.style.height = 'auto';
            inputRef.value.blur();
        }
        
        scrollToBottom();
        await syncSaveMessages();
    
        const decoder = new TextDecoder('utf-8');
        // ✅ 新的工具调用检测正则
        const toolPattern = /<<<AITOOL_CALL>>>[\s\S]*?<<<AITOOL_END>>>/;
                
        // 设置最大循环次数防止无限循环
        const MAX_TOOL_ROUNDS = 50;
        let currentRound = 0;
        let currentPrompt = historyStr ? (historyStr + '[当前用户输入]\n' + text) : text;
        let isFirstRound = true;
    
        try {
            while (currentRound < MAX_TOOL_ROUNDS) {
                currentRound++;
                
                const payload = {
                    prompt: currentPrompt,
                    includeHistory: isFirstRound && includeHistory.value
                };
    
                const response = await fetch(`${API_BASE}/sessions/${currentSessionId.value}/chat/stream`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload),
                    signal: signal
                });
    
                if (!response.ok) {
                    throw new Error(`请求失败: ${response.status}`);
                }
    
                const reader = response.body.getReader();
                
                // 创建 AI 消息占位
                const aiMsg = { role: 'ai', content: '', pinned: false };
                messages.value.push(aiMsg);
    
                // 流式读取
                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    
                    const chunk = decoder.decode(value, { stream: true });
                    aiMsg.content += chunk;
                    
                    messages.value = [...messages.value];
                    scrollToBottom();
                }
    
                await syncSaveMessages();
    
                // ✅ 关键改动：每次 AI 响应后都检查是否有工具调用
                if (!toolPattern.test(aiMsg.content)) {
                    // 没有工具调用，结束循环
                    break;
                }
    
                // 有工具调用，执行工具
                const statusMsg = { 
                    role: 'system', 
                    content: `<span class="exec-status running">⏳ 第 ${currentRound} 轮工具执行中...</span>`,
                    isHtml: true,
                    pinned: false
                };
                messages.value.push(statusMsg);
                scrollToBottom();
                
                const toolResults = await executeToolFromAI(aiMsg.content, statusMsg, signal);
                
                if (!toolResults || toolResults.length === 0) {
                    // 工具执行失败或无结果，移除状态消息并结束
                    const statusIndex = messages.value.indexOf(statusMsg);
                    if (statusIndex > -1) {
                        messages.value.splice(statusIndex, 1);
                    }
                    break;
                }
    
                // ✅ 准备下一轮的 prompt，包含工具执行结果
                currentPrompt = `Tool execution complete (Round ${currentRound}). Results:\n${JSON.stringify(toolResults, null, 2)}\n\nPlease continue based on these results. If you need to call more tools, you can do so.`;
                isFirstRound = false;
                
                await syncSaveMessages();
            }
    
            if (currentRound >= MAX_TOOL_ROUNDS) {
                messages.value.push({ 
                    role: 'system', 
                    content: `<div class="status-card">⚠️ 达到最大工具调用轮数 (${MAX_TOOL_ROUNDS})，已停止</div>`,
                    isHtml: true,
                    pinned: false
                });
            }
    
        } catch (error) {
            if (error.name === 'AbortError') {
                messages.value.push({ 
                    role: 'system', 
                    content: '<div class="status-card">⏹️ 生成已停止</div>',
                    isHtml: true,
                    pinned: false
                });
            } else {
                messages.value.push({ 
                    role: 'system', 
                    content: `<div class="status-card error">❌ 请求错误: ${escapeHtml(error.message)}</div>`,
                    isHtml: true,
                    pinned: false
                });
            }
            await syncSaveMessages();
        } finally { 
            loading.value = false;
            abortController = null;
            scrollToBottom();
        }
    };
    const openWorkspaceManager = () => { 
        window.location.href = '/static/workspace.html'; 
    };
    // ====== 工具方法 ======
    const openToolsManager = () => { window.location.href = '/static/tools_manager.html'; };
    const settingsNavigating = ref(false);
    const openSettings = () => {
      if (settingsNavigating.value) return;
      settingsNavigating.value = true;
      window.location.assign('/static/settings.html');
    };

    // ====== 初始化 ======
    onMounted(async () => {
        initKeyboardHandler();
        updateKeyboardState(false); 
        await loadCurrentModel(); // 加载模型名称
        await refreshSessions();
        scrollToBottom();
        
        // ✅ 新增：智能预加载
        // 延迟 1.5秒执行，确保不影响首页首屏渲染速度
        setTimeout(() => {
          preloadOtherPages();
        }, 1500);
    });
    
    onUnmounted(() => {
      // 清理 AbortController
      if (abortController) {
        abortController.abort();
        abortController = null;
      }
      if (window.visualViewport) {
        window.visualViewport.removeEventListener('resize', () => updateKeyboardState(false));
      }
      window.removeEventListener('resize', () => updateKeyboardState(false));
    });
    
    

    return {
      // Refs
      inputRef, bottomBar, chatContainer, dialogInputRef, scrollAnchor, editTextareaRef,
      
      // 键盘
      keyboardOpen, keyboardHeight, bottomBarStyle, chatContainerStyle,
      
      // 状态
      showSidebar, sidebarLoading, activeGroups,
      orderedGroups, getGroupSessions, currentSessionTitle, currentSessionId,
      toggleGroup, openSidebar, onSidebarOpen, onSidebarClose,
      
      // 历史记录开关
      includeHistory, onHistoryToggleChange,
      
      // 消息显示
      getDisplayContent,
      
      // 自定义 ActionSheet
      actionSheetShow, actionSheetTitle, actionSheetItems,
      closeActionSheet, onActionSelect,
      
      // 自定义输入对话框
      inputDialogShow, inputDialogTitle, inputDialogValue, inputDialogPlaceholder,
      cancelInputDialog, confirmInputDialog,
      
      // 自定义确认对话框
      confirmDialogShow, confirmDialogTitle, confirmDialogMessage,
      cancelConfirmDialog, doConfirmDialog,
      
      // 编辑/添加消息对话框
      editDialogShow, editDialogTitle, editDialogContent, editDialogRole,
      cancelEditDialog, confirmEditDialog,
      
      // 消息
      messages, userInput, loading,
      handleSendClick, stopGeneration,
      autoResize, onInputFocus, onInputBlur,
      summarizeHistory,
      
      // 消息操作
      deleteMessage, editMessage, togglePin, addMessageBelow,
      clearAllMessages, copyAllHistory, clearUnpinnedMessages,
      
      // 操作
      newChatInGroup, selectChat, handleAddGroup, 
      openSessionMenu, openGroupMenu,
      openToolsManager, openSettings, openQuickSwitch,
      scrollToBottom,
      // 模型名称（新增）
      currentModelName,
      
      openWorkspaceManager,
    };
    
  }
}).use(vant).mount('#app');
</script>
</body>
</html>