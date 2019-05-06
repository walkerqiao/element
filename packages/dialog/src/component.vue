<template>
  <transition name="dialog-fade" @after-leave="afterLeave">
    <div class="el-dialog__wrapper" v-show="visible" @click.self="handleWrapperClick">
      <div class="el-dialog" :class="[{ 'is-fullscreen': fullscreen, 'el-dialog--center': true }, customClass]" ref="dialog" :style="style">
        <div class="el-dialog__header eas-title">
          <slot name="title">
            <span class="el-dialog__title c-text-title">{{ title }}</span>
          </slot>
          <button type="button" class="el-dialog__headerbtn eas-close" aria-label="Close" v-if="showClose" @click="handleClose">
            <i class="el-icon el-icon-close c-text-title"></i>
          </button>
        </div>
        <div class="el-dialog__body eas-body" v-if="rendered">
          <slot></slot>
        </div>
        <div class="el-dialog__footer" v-if="$slots.footer">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
  </transition>
</template>
<style scoped>
.eas-box {
  border-radius: 0px;
  border: 0px;
}
.eas-body {
  padding: 0px;
}
.eas-title {
  background: #62a8ea;
  padding-top: 13px;
  padding-bottom: 11px;
}
.eas-close {
  top: 17px;
}
.eas-content {
  padding: 40px;
}
.eas-button {
  padding: 9px 24px;
  border-radius: 25px;
}
.eas-message {
  font-size: 16px;
}
.eas-message-info {
  color: #333333;
}
.eas-message-warning {
  color: #d6494b;
}
.c-success {
  background-color: #62a8ea;
}
.c-error {
  background-color: #d6494b;
}
.c-warning {
  background-color: #d6494b;
}
.c-info {
  background-color: #333333;
}
.c-text-title {
  color: white;
  font-size: 16px;
  font-weight: bold;
}
.center {
  text-align: center;
}
</style>
<script>
import Popup from 'element-ui/src/utils/popup';
import Migrating from 'element-ui/src/mixins/migrating';
import emitter from 'element-ui/src/mixins/emitter';

export default {
  name: 'ElDialog',

  mixins: [Popup, emitter, Migrating],

  props: {
    title: {
      type: String,
      default: ''
    },

    modal: {
      type: Boolean,
      default: true
    },

    modalAppendToBody: {
      type: Boolean,
      default: true
    },

    appendToBody: {
      type: Boolean,
      default: false
    },

    lockScroll: {
      type: Boolean,
      default: true
    },

    closeOnClickModal: {
      type: Boolean,
      default: true
    },

    closeOnPressEscape: {
      type: Boolean,
      default: true
    },

    showClose: {
      type: Boolean,
      default: true
    },

    width: String,

    fullscreen: Boolean,

    customClass: {
      type: String,
      default: ''
    },

    top: {
      type: String,
      default: '15vh'
    },
    beforeClose: Function,
    center: {
      type: Boolean,
      default: false
    }
  },

  data() {
    return {
      closed: false
    };
  },

  watch: {
    visible(val) {
      if (val) {
        this.closed = false;
        this.$emit('open');
        this.$el.addEventListener('scroll', this.updatePopper);
        this.$nextTick(() => {
          this.$refs.dialog.scrollTop = 0;
        });
        if (this.appendToBody) {
          document.body.appendChild(this.$el);
        }
      } else {
        this.$el.removeEventListener('scroll', this.updatePopper);
        if (!this.closed) this.$emit('close');
      }
    }
  },

  computed: {
    style() {
      let style = {};
      if (this.width) {
        style.width = this.width;
      }
      if (!this.fullscreen) {
        style.marginTop = this.top;
      }
      return style;
    }
  },

  methods: {
    getMigratingConfig() {
      return {
        props: {
          'size': 'size is removed.'
        }
      };
    },
    handleWrapperClick() {
      if (!this.closeOnClickModal) return;
      this.handleClose();
    },
    handleClose() {
      if (typeof this.beforeClose === 'function') {
        this.beforeClose(this.hide);
      } else {
        this.hide();
      }
    },
    hide(cancel) {
      if (cancel !== false) {
        this.$emit('update:visible', false);
        this.$emit('close');
        this.closed = true;
      }
    },
    updatePopper() {
      this.broadcast('ElSelectDropdown', 'updatePopper');
      this.broadcast('ElDropdownMenu', 'updatePopper');
    },
    afterLeave() {
      this.$emit('closed');
    }
  },

  mounted() {
    if (this.visible) {
      this.rendered = true;
      this.open();
      if (this.appendToBody) {
        document.body.appendChild(this.$el);
      }
    }
  },

  destroyed() {
    // if appendToBody is true, remove DOM node after destroy
    if (this.appendToBody && this.$el && this.$el.parentNode) {
      this.$el.parentNode.removeChild(this.$el);
    }
  }
};
</script>
