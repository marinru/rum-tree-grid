<template>
  <div class="tree-grid__container">
    <table class="tree-grid__grid">
        <thead v-if="cols">
          <tr>
            <th class="tree-grid__cell" v-for="col in cols" :key="col">
                <div class="tree-grid__cell-wrapper">
                    <slot :name="`${col}-heading`"></slot>
                </div>
            </th>
          </tr>
        </thead>
        <slot></slot>
    </table>
  </div>
</template>

<script>
import TreeGridBody from "./TreeGridBody";

export default {
  components: { TreeGridBody },
  props: { cols: Array }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.tree-grid__container {
    width: 100%;
    overflow-x: auto;
}

.tree-grid__grid {
    min-width: 100%;
    border-spacing: 0;
    border-collapse: collapse;
}

.tree-grid__cell {
    padding: 0;
    border-left: 1px dashed #e2e2e2;
}

.tree-grid__cell:first-child {
    border-left: none;
}

.tree-grid__cell-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 34px;
    overflow: hidden;
}

</style>
