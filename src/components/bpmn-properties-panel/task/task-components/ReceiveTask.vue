<template>
  <div>
    <a-form-model size="small" :label-col="labelCol" :wrapper-col="wrapperCol" @submit.native.prevent>
      <a-form-model-item label="消息实例">
        <div style="display: flex; align-items: center; justify-content: space-between; flex-wrap: nowrap">
          <a-select v-model="bindMessageId" @change="updateTaskMessage">
            <a-select-option v-for="id in Object.keys(messageMap)" :value="id" :key="id">{{ messageMap[id] }}</a-select-option>
          </a-select>
          <a-button size="small" type="primary" icon="plus" style="margin-left: 8px" @click="openMessageModel" />
        </div>
      </a-form-model-item>
    </a-form-model>
    <a-modal :visible="messageModelVisible" :mask-closable="false" title="创建新消息" width="400px" destroy-on-close>
      <a-form-model :model="newMessageForm" :label-col="labelCol" :wrapper-col="wrapperCol" @submit.native.prevent>
        <a-form-model-item label="消息ID">
          <a-input v-model="newMessageForm.id" allow-clear />
        </a-form-model-item>
        <a-form-model-item label="消息名称">
          <a-input v-model="newMessageForm.name" allow-clear />
        </a-form-model-item>
      </a-form-model>
      <template slot="footer">
        <a-button type="primary" @click="createNewMessage">确 认</a-button>
      </template>
    </a-modal>
  </div>
</template>

<script>
export default {
  name: "ReceiveTask",
  props: {
    id: String,
    type: String
  },
  data() {
    return {
      labelCol: { span: 6 },
      wrapperCol: { span: 18 },
      bindMessageId: "",
      newMessageForm: {},
      messageMap: {},
      messageModelVisible: false
    };
  },
  watch: {
    id: {
      immediate: true,
      handler() {
        this.$nextTick(() => this.getBindMessage());
      }
    }
  },
  created() {
    this.bpmnMessageRefsMap = Object.create(null);
    this.bpmnRootElements = window.bpmnInstances.modeler.getDefinitions().rootElements;
    this.bpmnRootElements
      .filter(el => el.$type === "bpmn:Message")
      .forEach(m => {
        this.bpmnMessageRefsMap[m.id] = m;
        this.$set(this.messageMap, m.id, m.name);
      });
    this.$set(this.messageMap, "-1", "无"); // 添加一个空对象，保证可以取消原消息绑定
  },
  methods: {
    getBindMessage() {
      this.bpmnElement = window.bpmnInstances.bpmnElement;
      this.bindMessageId = this.bpmnElement.businessObject?.messageRef?.id || "-1";
    },
    openMessageModel() {
      this.messageModelVisible = true;
      this.newMessageForm = {};
    },
    createNewMessage() {
      if (this.messageMap[this.newMessageForm.id]) {
        this.$message.error("该消息已存在，请修改id后重新保存");
        return;
      }
      const newMessage = window.bpmnInstances.moddle.create("bpmn:Message", this.newMessageForm);
      this.bpmnRootElements.push(newMessage);
      this.$set(this.messageMap, this.newMessageForm.id, this.newMessageForm.name);
      this.bpmnMessageRefsMap[this.newMessageForm.id] = newMessage;
      this.messageModelVisible = false;
    },
    updateTaskMessage(messageId) {
      if (messageId === "-1") {
        window.bpmnInstances.modeling.updateProperties(this.bpmnElement, {
          messageRef: null
        });
      } else {
        window.bpmnInstances.modeling.updateProperties(this.bpmnElement, {
          messageRef: this.bpmnMessageRefsMap[messageId]
        });
      }
    }
  },
  beforeDestroy() {
    this.bpmnElement = null;
  }
};
</script>
