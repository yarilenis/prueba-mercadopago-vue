<template>
  <div class="card shadow-sm bg-light overflow-hidden mb-3 border-left-0">
    <div
      class="p-3"
      :class="recomended ? 'border-primary' : 'border-dark'"
      style="border-left: 7px solid"
    >
      <div class="card-body py-3" @click="visible = !visible">
        <div class="d-flex justify-content-between">
          <h6 class="mb-0">
            <span v-if="title">
              <b>{{ title }}</b>
            </span>
            <slot v-else name="title"></slot>
          </h6>
          <div class="text-nowrap">
            <small v-if="recomended" class="text-primary d-none d-sm-inline">
              Recomendado
            </small>
            <span class="ml-2">
              <i v-if="visible" class="fas fa-chevron-up"></i>
              <i v-else class="fas fa-chevron-down"></i>
            </span>
          </div>
        </div>
      </div>
      <b-collapse v-model="visible">
        <div class="card-body pt-1">
          <slot name="description"></slot>
          <slot></slot>
        </div>
      </b-collapse>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    recomended: {
      type: Boolean,
      default: false,
    },
    title: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      visible: false,
    };
  },
  created() {
    if (this.recomended) {
      this.visible = true;
    }
  },
};
</script>
